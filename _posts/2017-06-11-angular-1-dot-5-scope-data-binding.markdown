---
title: Angular 1.5 - scope data binding
date: 2017-06-11 16:47:00 +02:00
tags:
- java script
- angular
---

This week the QA reported an issue. It was about not downloading file, when a download button was clicked. We use Angular 1.5 to develop frontend of our application.
This was Angular related issue which I will describe for you.

In our project there is a directive responsible for downloading attached files.
For simplicity let name this downloadButton.
In Angular you need remember that there are several ways to pass attributes to directive scope.
Based on git log, when this directive was created all attributes where passed as strings, a scope of this directive looked like:

{% highlight javascript %}
scope: {
    downloadUrlAttribute: '@'
}
{% endhighlight %}
where '@' means that you pass attribute as string, because DOM elements are strings.
It creates one way binding from parent controller to the child one.
But during development process one of team mates changed '@' to '=', this change caused reported issue.
Someone could ask: "but why? He changed one way binding to bidirectional. So what's wrong?"

For example, let have simple Angular app, where in scope there is defined a variable 
named downloadUrl which should contains link to pdf to download. As I mentioned before, we have separated directive responsible for downloading files, so we have pass attributes to this directive.

When you pass attribute to directive as a string, you need to remember to interpolate these attribute so it should look like:
{% highlight html %}
{% raw %}
<download-button download-url-attribute="{{downloadUrl}}"></download-button>
{% endraw %}
{% endhighlight %}

When you change binding to bidirectional, you should pass object to the directive, so you should remove string interpolation and following code should look like:
{% highlight html %}
<download-button download-url-attribute="downloadUrl"></download-button>
{% endhighlight %}
The team mate forgot to remove string interpolation, so he broke files downloading.

I prepared simple [plunker](https://plnkr.co/edit/EOiEUfmCKsmEhH4ih31K?p=preview) to show how it works in practice.
You can fork it and change source code to see what's gonning to happen.


If you have any questions or problems leave comment or mail to me.

See you soon!