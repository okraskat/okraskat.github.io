---
title: Selenium has a nice support for "accept cookies" popups
date: 2017-09-11 21:56:00 +02:00
---

Few days ago, I wrote some simple project using [Selenium](http://www.seleniumhq.org/). More than a year passed from time, when I worked with this library. I was really nice surprised with new features provided by Selenium.

Nowadays, many web sites show us popups about cookie policy which we have to accept to read page content. Selenium has nice support for dealing with these popups.

Let's see a simple example of application which checks [Oracle](https://www.oracle.com/technetwork/topics/security/alerts-086861.html) critical patch updates.
Firstly, we need to define our method which will search for patches from selected period:
{% highlight java %}
Map<LocalDate, String> searchPatchesForPeriod(WebDriver webDriver, LocalDate from, LocalDate to) {
    webDriver.get(url);
    waitForPageLoad(webDriver);
    waitForAcceptCookiesFrame(webDriver);
    acceptCookiesPolicy(webDriver);
    return searchForPatchUpdates(webDriver, from, to);
}
{% endhighlight %}

In first step we open page with url passed by constructor, then we wait for page being loaded. After page loaded we expect accept cookies frame which we have to accept. Then we can search for interesting patches.

Let's see waitForPageLoad implementation:
{% highlight java %}
private void waitForPageLoad(WebDriver webDriver) {
    Wait<WebDriver> wait = new WebDriverWait(webDriver, TIMEOUT_IN_SECONDS);
    wait.until(webDriver1 -> ((JavascriptExecutor) webDriver).executeScript(DOCUMENT_READY_STATE).equals(COMPLETE));
}
{% endhighlight %}
In this method we check document state using JavascriptExecutor. We can tell WebDriver to wait until page is ready. When waiting time is greater than TIMEOUT_IN_SECONDS WebDriver will throw exception.

But for me, the most interesting part is defined in acceptCookiesPolicy method:
{% highlight java %}
private void acceptCookiesPolicy(WebDriver webDriver) {
    webDriver.findElement(By.xpath(ACCEPT_COOKIES_BUTTON_X_PATH)).click();
    webDriver.switchTo().defaultContent();
}
{% endhighlight %}
As we see, after accept cookies button click we can simply switch to default content and Selenium will now operate on the main document.
Is that so simple? :)

Now we can search for interesting patch updates:
{% highlight java %}
private Map<LocalDate, String> searchForPatchUpdates(WebDriver webDriver, LocalDate from, LocalDate to) {
        Map<LocalDate, String> foundUpdates = new HashMap<>();
        List<WebElement> patchUpdates = webDriver.findElements(By.xpath(PATCH_UPDATE_TABLE_ROWS_X_PATH));
        patchUpdates.stream()
            .skip(1)
            .forEach(patchUpdate -> {
                List<WebElement> patchUpdateColumns = patchUpdate.findElements(By.tagName(TD));
                WebElement columnWithDate = patchUpdateColumns.get(1);
                String dateText = columnWithDate.getText().split(COMMA)[1].substring(1);
                Optional<LocalDate> patchDate = parseDate(dateText);

                if (patchDate.isPresent() && isDateBetween(from, to, patchDate.get())) {
                     foundUpdates.put(patchDate.get(), String.format("%s %s", patchUpdateColumns.get(0).getText(), columnWithDate.getText()));
                }
           });
    return foundUpdates;
}
{% endhighlight %}
In this method we load all rows from table containing rows with patch updates. We skip a first row, which contains column titles. Then we can parse date from second column and check if it is between interested time period.

In my opinion, after almost 2 years selenium API became more friendly for developers. It has also great community support with lot of [StackOverflow questions](https://stackoverflow.com/questions/tagged/selenium).

You can find the source code in my Github repository [how-to](https://github.com/okraskat/how-to) under a selenium-cookies directory.

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!