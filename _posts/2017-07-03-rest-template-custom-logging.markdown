---
title: How to customize HTTP communication logging with RestTemplate?
date: 2017-07-03 20:11:00 +02:00
categories:
- java
tags:
- java
- resttemplate
- spring
---

Nowadays HTTP communication is very important in enterprise projects.
For people who use Spring related technologies an obvious decision is to use RestTemplate for executing HTTP requests. Today, I'm going to describe how to customize HTTP communication logging while using RestTemplate.

First create simple Spring Boot application:
{% highlight java %}
@SpringBootApplication
public class Application {
    public static void main(String\[\] args) {
        SpringApplication.run(Application.class, args);
    }
}
{% endhighlight %}

Second step is to configure our application. In out configuration class we define RestTemplate bean. We set interceptor list for our RestTemplate. Interceptor intercepts client-side HTTP requests. CustomClientHttpRequestInterceptor is our class implementing ClientHttpRequestInterceptor interface. It's allow us to process HTTP requests and responses.
{% highlight java %}
@Configuration
public class ApplicationConfig {
    @Bean
    public RestTemplate restTemplate(CustomClientHttpRequestInterceptor customClientHttpRequestInterceptor) {
        RestTemplate restTemplate = new RestTemplate();
        List<ClientHttpRequestInterceptor> interceptors = Collections.singletonList(customClientHttpRequestInterceptor);
        restTemplate.setInterceptors(interceptors);
        return restTemplate;
    }
}
{% endhighlight %}

Let's see how our CustomClientHttpRequestInterceptor looks like. We need to override intercept metohd which parameters are: a HttpRequest instance, an bytes array which contains request body and ClientHttpRequestExecution instance.

We register Spring bean with @Component annotation. In first line we define LOGGER to to write interesting data into standard output or file (depends on configuration).
In out method implementation we log request URI on debug level. Then we trace request data. Next step is to execute HttpRequest with given body. Then we can trace response data and log response code on debug level. After all we need to return response instance. traceRequest and traceResponse methods are quite obvoius. If you have any question about them, please leave a comment.

{% highlight java %}
@Component
public class CustomClientHttpRequestInterceptor implements ClientHttpRequestInterceptor {
private static final Logger LOGGER = LoggerFactory.getLogger(CustomClientHttpRequestInterceptor.class);

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        LOGGER.debug("Request to: {}", request.getURI());
        traceRequest(request, body);
        ClientHttpResponse response = execution.execute(request, body);
        LOGGER.debug("Response code: {}", response.getRawStatusCode());
        traceResponse(response);
        return response;
    }
    
    private void traceRequest(HttpRequest request, byte[] body) throws IOException {
        LOGGER.trace("=============================request begin================================================");
        LOGGER.trace("URI         : {}", request.getURI());
        LOGGER.trace("Method      : {}", request.getMethod());
        LOGGER.trace("Headers     : {}", request.getHeaders());
        LOGGER.trace("Request body: {}", new String(body, "UTF-8"));
        LOGGER.trace("=============================request end================================================");
    }
    
    private void traceResponse(ClientHttpResponse response) throws IOException {
        StringBuilder responseBodyBuilder = new StringBuilder();
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(response.getBody(), StandardCharsets.UTF_8));
        String line = bufferedReader.readLine();
        while (line != null) {
            responseBodyBuilder.append(line).append(System.lineSeparator());
            line = bufferedReader.readLine();
        }
        LOGGER.trace("============================response begin==========================================");
        LOGGER.trace("Status code : {}", response.getStatusCode());
        LOGGER.trace("Status text : {}", response.getStatusText());
        LOGGER.trace("Headers     :");
        response.getHeaders().forEach((headerName, headerValue) -> LOGGER.trace("Header name : {}, header value: {}", headerName, headerValue));
        LOGGER.trace("Response body: {}", responseBodyBuilder.toString());
        LOGGER.trace("=======================response end=================================================");
    }

}
{% endhighlight %}

Ok, but we need have to some rest HTTP service to exchange data. For this example we will use http://jsonplaceholder.typicode.com (http://jsonplaceholder.typicode.com) which is free rest api for testing and education purposes.

Let's define our SimpleHttpClient

{% highlight java %}
@Component
public class SimpleHttpClient {
private final RestTemplate restTemplate;
private final String serviceUrl;

    @Autowired
    public SimpleHttpClient(RestTemplate restTemplate, @Value("${service.url}") String serviceUrl) {
        this.restTemplate = restTemplate;
        this.serviceUrl = serviceUrl;
    }
    
    @PostConstruct
    public void postForEntity() {
        ServiceExchangeData request = buildRequestData();
        String url = serviceUrl + "/posts";
        restTemplate.postForEntity(url, request, ServiceExchangeData.class);
    }
    
    private ServiceExchangeData buildRequestData() {
        ServiceExchangeData request = new ServiceExchangeData();
        request.setTitle("Title");
        request.setBody("Body");
        request.setUserId(222L);
        return request;
    }

}
{% endhighlight %}

As earlier, we register Spring bean, then we can use Spring Dependency Injection to inject dependencies into our bean. The first dependency is RestTemplate defined in Application config. Second one is serviceUrl which is provided in application.properties file.

Let's see this file:

{% highlight %}
spring.profiles.active=develop
service.url=http://jsonplaceholder.typicode.com
{% endhighlight %}

We create postForEntity method which is annotated with @PostConstruct annotation. It means that this method will be fired just after bean creation. In our metohd we provide some data to exchange, create requestUrl and exchange data with RestTemplate.

Ok, but how to see HTTP communication data which is transfered in this example?

We need to include in out pom logging starter, with this dependency we can configure our logging.
