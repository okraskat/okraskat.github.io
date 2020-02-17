---
title: Throttling in Java - using Spring + Guava
date: 2020-02-17 08:48:00 Z
tags:
- java
---

Recently, I had to limit request count for our API in Spring application.
I tried to write my own throttling implementation.

In first step, we need to define an annotation, which will help us to mark methods which calls should be throttled:

{% highlight java %}
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Throttling {
    int timeFrameInSeconds();
    int calls();
}
{% endhighlight %}

This annotation allow us to limit calls count in given time frame. If second is to big time frame, you can change it to millis or nanos.

Next step is implement a throttle manager:

{% highlight java %}
@Service
class ThrottlingManager {
    private final Map<EndpointMethod, Map<String, Cache<Long, Long>>> ENDPOINT_THROTTLE_MAPPING = new ConcurrentHashMap<>();

    void throttleRequest(EndpointMethod endpointMethod, String userId, ThrottlingConfig throttlingConfig) {
        Map<String, Cache<Long, Long>> endpointThrottle = ENDPOINT_THROTTLE_MAPPING.computeIfAbsent(endpointMethod, k -> new HashMap<>());
        Cache<Long, Long> autoExpiringUserCallsCounter = endpointThrottle.computeIfAbsent(userId, k -> buildCacheWhichRemovesEntriesAfterTimeFrame(throttlingConfig));
        Long callsCount = autoExpiringUserCallsCounter.size();
        if (requestLimitReached(throttlingConfig, callsCount)) {
            autoExpiringUserCallsCounter.cleanUp();
            if (requestLimitReached(throttlingConfig, autoExpiringUserCallsCounter.size())) {
                throw new RequestLimitReached(userId, endpointMethod);
            }
        } else {
            long randomKeyToIncreaseCounter = new SecureRandom().nextLong();
            autoExpiringUserCallsCounter.put(randomKeyToIncreaseCounter, randomKeyToIncreaseCounter);
        }
    }

    private boolean requestLimitReached(ThrottlingConfig throttlingConfig, Long callsCount) {
        return callsCount != null && callsCount + 1 > throttlingConfig.getCallsCount();
    }

    private Cache<Long, Long> buildCacheWhichRemovesEntriesAfterTimeFrame(ThrottlingConfig throttlingConfig) {
        return CacheBuilder.newBuilder()
                .expireAfterWrite(throttlingConfig.getTimeFrameInSeconds(), TimeUnit.SECONDS)
                .build();
    }
}
{% endhighlight %}

All magic is placed in ThrottlingManager.
It takes endpoint method, userId and throttling config and it counts calls in given timeframe using Guava cache. Guava gives us simple tool to delete expired entries in our cache counter. Every endpoint call is represented in cache as a random number. We are not interested about request body, parameters etc. We only care about calls count.

Here is an implementation of simple ThrottlingConfig.

{% highlight java%}
class ThrottlingConfig {
    static final ThrottlingConfig DEFAULT = new ThrottlingConfig(600, 300);

    private int timeFrameInSeconds;
    private int callsCount;

    public ThrottlingConfig(int timeFrameInSeconds, int callsCount) {
        this.timeFrameInSeconds = timeFrameInSeconds;
        this.callsCount = callsCount;
    }

    public int getTimeFrameInSeconds() {
        return timeFrameInSeconds;
    }

    public int getCallsCount() {
        return callsCount;
    }
}
{% endhighlight %}

This is how our EnpointMethod looks like.

{% highlight java %}
class EndpointMethod {

    private final Class targetClass;
    private final String targetMethod;

    public EndpointMethod(Class targetClass, String targetMethod) {
        this.targetClass = targetClass;
        this.targetMethod = targetMethod;
    }

    public String getTargetMethod() {
        return targetMethod;
    }

    public Class getTargetClass() {
        return targetClass;
    }

    //equals and hashCode methods
}
{% endhighlight %}

As we can see, it contains class and method name. We use objects of this class as ConcurrentHashMap key, remember to override hashCode and equals methods from Java Object class.

When we have these elements, we can implement aspect which will allow us to process throttling.
{% highlight java %}
@Aspect
@Component
public class ThrottlingAspect {

    private final ThrottlingManager throttlingManager;

    @Autowired
    public ThrottlingAspect(ThrottlingManager throttlingManager) {
        this.throttlingManager = throttlingManager;
    }

    @Pointcut("within(@(@org.springframework.stereotype.Controller *) *)")
    public void controllerPointcut() {
        // pointuct
    }

    @Before("controllerPointcut()")
    public void log(JoinPoint pjp) {
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        Method method = signature.getMethod();
        ThrottlingConfig throttlingConfig = getThrottlingConfig(method);
        EndpointMethod endpointMethod = new EndpointMethod(pjp.getTarget().getClass(), method.getName());
        UserIdProvider.getCurrentUserId()
                .ifPresent(id -> throttlingManager.throttleRequest(endpointMethod, id, throttlingConfig));
    }

    private ThrottlingConfig getThrottlingConfig(Method method) {
        return Arrays.stream(method.getDeclaredAnnotations())
                .filter(d -> d.annotationType() == Throttling.class)
                .findFirst()
                .map(d -> {
                    Throttling t = (Throttling) d;
                    return new ThrottlingConfig(t.timeFrameInSeconds(), t.calls());
                })
                .orElse(ThrottlingConfig.DEFAULT);
    }
}
{% endhighlight %}

This aspect processes around method annotated with @Controller or @RestController. It constructs EndpointMethod object and checks, if method was annotated with @Throttling and build proper config, otherwise it takes default value. You can change this behaviour as you want.

It's time to test our implementation. Let's create simple endpoint:
{% highlight java %}
@RestController("/test")
public class TestController {

    @GetMapping
    @Throttling(timeFrameInSeconds = 60, calls = 2)
    public String exampleEndpoint() {
        return UUID.randomUUID().toString();
    }

}
{% endhighlight %}

This controller has throttled method, so if our throttling is working third call should be rejected with exception.

Let's start our application with command: mvn spring-boot:run
and then type in our browser:
http://localhost:8080/test

Refresh page 2 times.

Now it's time to check logs:
{% highlight java %}
2020-02-17 08:56:33.970  INFO 12387 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 30 ms
Call number: 1
Call number: 2
{% endhighlight %}

As we can see method was called 2 times but third call was rejected. You can see something familiar in your browser:

Mon Feb 17 08:57:02 CET 2020
There was an unexpected error (type=Too Many Requests, status=429).
User: test@domain.com, reached calls limit for method: io.okraskat.throttling.EndpointMethod@71fd5723

Hope this example will help you to deal with throttling or be an inspiration, how can you combine Spring and Guava features.

You can find the source code in my Github repository [how-to](https://github.com/okraskat/how-to) under a throttling directory.

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!