---
title: Caching per request with Spring
date: 2017-11-05 15:53:00 Z
---

Recently, I had to cache some values during processing HTTP request in Spring application. Spring does not offer out of the box solution for this, so I had to write my own request cache.

In first step, we need to define an annotation, which will help us to mark methods which calls should be cached:

{% highlight java %}
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestCache {
}
{% endhighlight %}

Next step is implement a cache manager:
{% highlight java %}
@Component
@RequestScope(proxyMode = ScopedProxyMode.TARGET_CLASS)
class RequestCacheManager {

    private final Map<InvocationTarget, Object> cache = new ConcurrentHashMap<>();

    Optional<Object> get(InvocationTarget invocationContext) {
        return Optional.ofNullable(cache.get(invocationContext));
    }

    void put(InvocationTarget methodInvocation, Object result) {
        cache.put(methodInvocation, result);
    }
}
{% endhighlight %}

As we see, cache manager is standard Spring component which is annotated with @RequestScope which means, that Spring IoC will create new instance of our manager for every HTTP request. We will use ConcurrentHashMap to cache our values (this map is thread safe - if you know that you will work with single thread, you should use HashMap - it's not thread safe but faster).
Our cache manager has two methods - one to get value and second to put value in cache. The key of our map is InvocationTarget object which implementation is:

{% highlight java %}
class InvocationTarget {

    private static final String TO_STRING_TEMPLATE = "%s.%s(%s)";

    private final Class targetClass;
    private final String targetMethod;
    private final Object[] args;

    InvocationTarget(Class targetClass, String targetMethod, Object[] args) {
        this.targetClass = targetClass;
        this.targetMethod = targetMethod;
        this.args = args;
    }

    @Override
    public boolean equals(Object o) {
        return EqualsBuilder.reflectionEquals(this, o);
    }

    @Override
    public int hashCode() {
        return HashCodeBuilder.reflectionHashCode(this);
    }

    @Override
    public String toString() {
        return String.format(TO_STRING_TEMPLATE, targetClass.getName(), targetMethod, Arrays.toString(args));
    }
}
{% endhighlight %}

As we can see, it contains class, method and arguments of method call. It allows us to determine, if the method from same class was called with same arguments. As we use objects of this class as ConcurrentHashMap key, we need to remember to override hashCode and equals methods from Java Object class.

When we have these elements, we can implement aspect which will allow us to process value caching.
{% highlight java %}
@Aspect
@Component
class RequestScopeAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(RequestScopeAspect.class);

    private final RequestCacheManager requestCacheManager;

    @Autowired
    RequestScopeAspect(RequestCacheManager requestCacheManager) {
        this.requestCacheManager = requestCacheManager;
    }

    @Around("@annotation(io.okraskat.requestcache.RequestCache)")
    Object processRequestCache(ProceedingJoinPoint pjp) throws Throwable {
        InvocationTarget invocationTarget = new InvocationTarget(
                pjp.getSignature().getDeclaringType(),
                pjp.getSignature().getName(),
                pjp.getArgs()
        );
        Optional<Object> cachedResult = requestCacheManager.get(invocationTarget);
        if (cachedResult.isPresent()) {
            Object result = cachedResult.get();
            LOGGER.info("Using cached value {}, for invocation: {}", result, invocationTarget);
            return result;
        } else {
            Object methodResult = pjp.proceed();
            LOGGER.info("Caching result: {}, for invocation: {}", methodResult, invocationTarget);
            requestCacheManager.put(invocationTarget, methodResult);
            return methodResult;
        }
    }
}
{% endhighlight %}

This aspect processes around method annotated with @RequestCache. It constructs InvocationTarget object and checks, if cache contains value for this key.
If cached result is present, then this cached value is returned. Otherwise target method is called and result is stored in our cache.

It's time to test our implementation. Let's create class which calls will be cached:
{% highlight java %}
@Component
class RandomGenerator {
    private static final Logger LOGGER = LoggerFactory.getLogger(RandomGenerator.class);
    private static final Random RANDOM = new Random();

    @RequestCache
    int getRandomNumber() {
        LOGGER.info("Generating random number ...");
        return RANDOM.nextInt(100) + 1;
    }
}
{% endhighlight %}
It should generate new random number every time when getRandomNumber method is called.

Let's create controller which will use this class:
{% highlight java %}
@RestController("/random")
class RandomController {

    private final RandomGenerator randomGenerator;

    @Autowired
    RandomController(RandomGenerator randomGenerator) {
        this.randomGenerator = randomGenerator;
    }

    @GetMapping
    int getRandomNumberSum() {
        int first = randomGenerator.getRandomNumber();
        int second = randomGenerator.getRandomNumber();
        return first + second;
    }
}
{% endhighlight %}

This controller calls RandomGenerator twice, so if our cache is working second call should return same value as with the first call.

Let's start our application with command: mvn spring-boot:run
and then type in our browser:
http://localhost:8080/random

Now it's time to check logs:
{% highlight java %}
2017-11-05 16:05:30.446  INFO 2948 --- [io-8080-exec-10] i.okraskat.requestcache.RandomGenerator  : Generating random number ...
2017-11-05 16:05:30.447  INFO 2948 --- [io-8080-exec-10] i.o.requestcache.RequestScopeAspect      : Caching result: 35, for invocation: io.okraskat.requestcache.RandomGenerator.getRandomNumber([])
2017-11-05 16:05:30.466  INFO 2948 --- [io-8080-exec-10] i.o.requestcache.RequestScopeAspect      : Using cached value 35, for invocation: io.okraskat.requestcache.RandomGenerator.getRandomNumber([])
{% endhighlight %}

As we can see first call of RandomGenerator was successfully cached. Every thing works as expected!

Spring does not offer every kind of cache out of the box, but it's powerful framework and we can simply implement our custom cache using Spring features.

You can find the source code in my Github repository [how-to](https://github.com/okraskat/how-to) under a requestcache directory.

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!