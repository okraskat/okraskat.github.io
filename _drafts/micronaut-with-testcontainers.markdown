---
title: Micronaut with Testcontainers
date: 2021-09-22 18:11:00 Z
---

I wanted to share with you how to configure Micronaut tests with Testcontainers.
This example allows you to run 
{% highlight java %}
@MicronautTest
{% endhighlight %}
which builds whole Micronaut context and create a connection to the dockerized PostgreSQL database.

This 