---
title: Code in Google Sheets
date: 2017-10-07 11:34:00 Z
tags:
- JavaScript
---

Do you know that Google Sheets offers scripting?

Under Tools -> Script editor You can simply write your own JavaScript code to improve your sheet.

The easiest way to fire function is to create custom menu. You just need to type in script editor following code:
{% highlight javascript %}
function onOpen() {
  var ui = SpreadsheetApp.getUi();
  ui.createMenu('First custom menu')
      .addItem('Fire alert function', 'alertFunction')
      .addToUi();
}
{% endhighlight %}
This function will be fired after document's open. It creates custom menu option with one item which will fire 'alertFunction'.
Let's define this function.

{% highlight javascript %}
function alertFunction() {
   SpreadsheetApp.getUi().alert('You just implemented you first function in GSuite! Congratulations!');
}
{% endhighlight %}
Cooperating with UI is not the only thing we can do by scripts.

Google offers us much more functionalities like operating with sheet data:
{% highlight javascript %}
function getFirstCellValue() {
   var firstCellValue = SpreadsheetApp.getActiveSheet().getRange('A1').getValue();
   SpreadsheetApp.getUi().alert(firstCellValue);
}
{% endhighlight %}

Sending emails:
{% highlight javascript %}
function sendEmail() {
   var emailAddress = 'recipient@domain.com';
   var subject = 'Subject';
   var message = 'Welcome message';
   MailApp.sendEmail(emailAddress, subject, message);
}
{% endhighlight %}
Emails will be send from our gmail account.
{% include warning.html content="Remember that you script can only send 100 emails per day, if you need more you need to have Google business account" %}

In your scripts you can use all JavaScript features.
In my opinion - it's very important that we don't need to learn another syntax - just code!

For more information visit [Official documentation](https://developers.google.com/apps-script/).

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!
