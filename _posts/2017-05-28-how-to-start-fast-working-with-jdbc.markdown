---
title: How to start fast working with JDBC?
date: 2017-05-28 18:19:00 +02:00
tags:
- java
- jdbc
- h2
---

Many people, at the beginning of their adventure with software development, are scared how many things and tools they have to learn. This is example how to start fast working with JDBC (for more information please go [here](http://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)). There is no need to install any database engine - we will use simple [H2 database](http://www.h2database.com/html/main.html).

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

Let's see implementation of the first method:
{% highlight java %}
private static Connection initializeDatabaseConnection() {
    try {
        Class.forName(DRIVER_CLASS_NAME);
        return DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_USER_PASSWORD);
    } catch (ClassNotFoundException | SQLException e) {
        e.printStackTrace(System.err);
        throw new IllegalStateException(e);
    }
}
{% endhighlight %}

First of all we need register JDBC Driver class name. After this done, we can try to get connection for our databse. We need to handle 2 exceptions and terminate our program when one of them occurse.

Second method is responsible for creating database schema. When we already have connection to database, we can execute query to create example Car table.
After executing query we should close used statement. Handled SQLException is print to default system error output.
{% highlight java %}
private static void initializeDatabaseSchema(Connection connection) {
    try {
        String createTableCommand = "CREATE TABLE CAR(id IDENTITY, make varchar(30), model varchar (30))";
        PreparedStatement createPreparedStatement = connection.prepareStatement(createTableCommand);
        createPreparedStatement.executeUpdate();
        createPreparedStatement.close();
    } catch (SQLException e) {
        e.printStackTrace(System.err);
    }
}
{% endhighlight %}

Third method insert data to our Car table. We create prepared statement based on SQL stored in insertValuesCommand. Question marks present query arguments.
As we can see we set arguments in our statement - first argument of setInt or setString method is index of argument in our query - starting from 1.

When we set arguments we can execute query. Prepared statements can be reused- as in example - to execute multiple queries.
{% highlight java %}
private static void insertExampleValues(Connection connection) {
    try {
        String insertValuesCommand = "INSERT INTO CAR(id, make, model) VALUES(?, ?, ?)";
        PreparedStatement preparedStatement = connection.prepareStatement(insertValuesCommand);
        preparedStatement.setInt(1, 1);
        preparedStatement.setString(2, "Toyota");
        preparedStatement.setString(3, "Corolla");
        preparedStatement.executeUpdate();
        preparedStatement.setInt(1, 2);
        preparedStatement.setString(2, "Ford");
        preparedStatement.setString(3, "Focus");
        preparedStatement.executeUpdate();
        preparedStatement.close();
    } catch (SQLException e) {
        e.printStackTrace(System.err);
    }
}
{% endhighlight %}

Now we can fetch inserted values. We create select query - remeber - using '*' is bad practise, You should know which data You want to select/read (You can read more about this [here](https://stackoverflow.com/questions/3639861/why-is-select-considered-harmful)).
When we execute query prepared statement will return result set, which contains fetched data - it gives ous possibility to iterate over these values.

With "next" method we check, if there is some not read yet record. We can get specified data by column name (id, make) or column index in result set (model).
We print fetched data to console.
{% highlight java %}
private static void selectValues(Connection connection) {
    try {
        String selectValuesCommand = "SELECT id, make, model FROM CAR";
        PreparedStatement createPreparedStatement = connection.prepareStatement(selectValuesCommand);
        ResultSet resultSet = createPreparedStatement.executeQuery();
        while (resultSet.next()) {
            int carId = resultSet.getInt("id");
            String make = resultSet.getString("make");
            String model = resultSet.getString(3);
            System.out.println(String.format("Car fetched from database: id = %d, make = %s, model = %s", carId, make, model));
        }
        createPreparedStatement.close();
    } catch (SQLException e) {
        e.printStackTrace(System.err);
    }
}
{% endhighlight %}

At the end of our example we close connection to our database. Database engine has limited count of connection, so You should remember to close it, when You finish your work.
{% highlight java %}
private static void closeDatabaseConnection(Connection connection) {
    try {
        connection.close();
    } catch (SQLException e) {
        e.printStackTrace(System.err);
    }
}
{% endhighlight %}

You can find source code in my Github repository [how-to](https://github.com/okraskat/how-to) under jdbc directory.

Hope you enjoy this post about starting working with JDBC. If You have any questions or problems leave comment or mail to me.

See You soon!