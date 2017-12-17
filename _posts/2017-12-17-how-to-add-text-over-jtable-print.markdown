---
title: How to add text over JTable print?
date: 2017-12-17 14:45:00 Z
categories:
- java
tags:
- java
- JTable
- pdf
---

During work on one of my last projects I faced to problem, how to add text over JTable print? I couldn't find any solution which would suit to my case. After several hours of trying another approaches I found easy way how to add text over JTable print. Let's see how to do this!

I suppose that you have code with JTable, which is filled with your data.
Let's say that your app has "Print" button.

We need to implement code, which will be executed when click action will perform.
{% highlight java %}
private void jButton_printButtonActionPerformed(java.awt.event.ActionEvent evt) {                                                
    MessageFormat header = new MessageFormat("");
    MessageFormat footer = new MessageFormat("Page {0,number,integer}");
    try {
        PrinterJob printerJob = PrinterJob.getPrinterJob();
        printerJob.printerJob(new JTablePrintableWrapper(yourJTable.getPrintable(JTable.PrintMode.FIT_WIDTH, header, footer)), printerJob.defaultPage());
        printerJob.print();
    } catch (Exception e) {
       System.err.format("Error during print JTable data");
       e.printStackTrace();
    }
}
{% endhighlight %}

The tricky part is to implement JTablePrintableWrapper.

{% highlight java %}
public class JTablePrintableWrapper implements Printable {
    private final Printable jTablePrintable;

    public JTablePrintableWrapper(Printable jTablePrintable) {
        this.jTablePrintable= jTablePrintable;
    }

    @Override
    public int print(Graphics g, PageFormat pf, int pageIndex) throws PrinterException {

       if (pageIndex == 0) {
           Graphics2D g2d = (Graphics2D) g;
           //this is a place where you can print what you want using Graphics2D object
           // after this you should remember to correct translate Graphics2D
           // to the place where you finished you custom text
           // it's necessary to print data below your text
           g.translate(0, Y_COORDINATE_WHERE_YOU_FINISHED_PRINTING);
        }
        // this is place where you print your data from JTable
        return jTablePrintable.print(g, pf, pageIndex);
    }
}
{% endhighlight %}

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!