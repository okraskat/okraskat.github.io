---
title: String.format - not so obvious
date: 2017-07-16 14:29:00 +02:00
categories:
- java
- jdk
tags:
- java
- jdk
- string
- format
---

To format numbers in Java we should use String.format method, which arguments are:
- Locale instance
- specified string format
- arguments to format (array of objects)
we also can use method with default Locale.

In one of projects that I'm working on, there was a method:
{% highlight java %}
 private static String formatDoubleNumber(Object argument) {
    return String.format("Number: %.4f", argument);
}
{% endhighlight %}

This is simple method to format double number with 4 digits after separator. But, as you can see, argument is an object, which allows you to pass both, integer and double numbers. If argument is a integer instance the IllegalFormatConversionException will be thrown. But why? The first thought is that, Java should also format integer with additional zeros at the end. It's not so obvious.
If you look deeper in String class, it uses java.util.Formatter class to format strings. In this class there is a method to print float numbers (which was our target). Let see implementation of this method:

{% highlight java %}
private void printFloat(Object arg, Locale l) throws IOException {
    if (arg == null)
        print("null");
    else if (arg instanceof Float)
        print(((Float)arg).floatValue(), l);
    else if (arg instanceof Double)
        print(((Double)arg).doubleValue(), l);
    else if (arg instanceof BigDecimal)
        print(((BigDecimal)arg), l);
    else
        failConversion(c, arg);
}
{% endhighlight %}
As we can see, with this method we can only format Float, Double and BigDecimal instances. Developer should remember to convert object to one of the listed class.
How correct and null save implementation of the formatDoubleNumber method should look like?
Here is a solution:

{% highlight java %}
private static String formatDoubleNumber(Object argument) {
    Double doubleValue = null;
    if (argument instanceof Number) {
        doubleValue = ((Number) argument).doubleValue();
    }
    return String.format("Number: %.4f", doubleValue);
}
{% endhighlight %}

Firstly we should check that given argument is instanceof Number, which is abstract class with doubleValue() method. If we sure, that passed argument is Number, then we can cast this object and call doubleValue method.

This implementation allows us to pass instances of all number data types in Java: Byte, Short, Integer, Long, Float, Double and BigDecimal.

You can find the source code in my Github repository [how-to](https://github.com/okraskat/how-to) under a stringformat directory.

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!