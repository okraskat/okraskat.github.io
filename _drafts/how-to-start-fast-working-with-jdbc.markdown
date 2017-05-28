---
title: How to start fast working with JDBC?
date: 2017-05-28 18:19:00 +02:00
tags:
- java
- jdbc
- h2
---

Many people, at the beginning of their adventure with software development, are scared how many things and tools they have to learn. This is example how to start fast working with JDBC (for more information please go [here](http://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)).

First step - download H2 database driver from [here](https://mvnrepository.com/artifact/com.h2database/h2).

Second step - add this jar file to Your project.

Let's write some code!

Create class JdbcExample.

Then define database configuration:
{% highlight java %}
    private static final String DRIVER_CLASS_NAME = "org.h2.Driver";
    private static final String DATABASE_URL = "jdbc:h2:~/test";
    private static final String DATABASE_USER = "";
    private static final String DATABASE_USER_PASSWORD = "";
{% endhighlight %}

Create our main method to fire example after implementation
{% highlight java %}
    public static void main(String[] args) {
        DeleteDbFiles.execute("~", "test", true);
        Connection connection = initializeDatabaseConnection();
        initializeDatabaseSchema(connection);
        insertExampleValues(connection);
        selectValues(connection);
        closeDatabaseConnection(connection);
    }
{% endhighlight %}

As You can see first step is to delete old database file - located in user home directory, named test.mkv.
In next line, we initialize database connection.
Next method is responsible for creating table - initializing schema.
Then we can insert values into our database.
In next step we would like to read/select inserted data.
After all we need to close database connection.