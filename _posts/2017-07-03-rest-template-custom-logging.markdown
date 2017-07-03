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