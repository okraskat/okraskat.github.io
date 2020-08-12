---
title: Testing time in Java
date: 2020-08-12 09:56:00 Z
---

Recently I read article about testing time in Java (written in Polish):
[link](https://sztukakodu.pl/jak-madrze-testowac-czas-w-javie/)

I think that, it's worth to prefer usage of classes provided by JDK, than creating special interfaces. I would like to present you, my approach for testing time in Java.

Let's have simple entity with 2 LocalDateTime fields.
When we create our entity, we initialize expiration date with one month period.
Entity has also simple update method to set updatedAt field.

{% highlight java %}
class Entity {
    private static Clock clock = Clock.systemDefaultZone();

    private LocalDateTime updatedAt;
    private LocalDateTime expiresAt;
    
    Entity() {
        this.expiresAt = LocalDateTime.now(clock).plusMonths(1);
    }

    void update() {
        this.updatedAt = LocalDateTime.now(clock);
    }
}
{% endhighlight %}

Ok, let's test time!
{% highlight groovy %}
class EntityTest extends Specification {

    def "should set proper dates"() {
        given:
        def fixedDateTime = LocalDateTime.of(2020, Month.AUGUST, 12, 10, 0)
        Entity.clock = Clock.fixed(fixedDateTime.atZone(ZoneId.systemDefault()).toInstant(), ZoneId.systemDefault())

        when:
        def entity = new Entity()

        then:
        entity.expiresAt == LocalDateTime.of(2020, Month.SEPTEMBER, 12, 10, 0)

        and:
        entity.update()

        then:
        entity.updatedAt == fixedDateTime
    }
}
{% endhighlight %}

If you prefer injecting values, your entity will look like this:
{% highlight java %}
class Entity {
    private final Clock clock;

    private LocalDateTime updatedAt;
    private LocalDateTime expiresAt;

    Entity(Clock clock) {
        this.clock = clock;
        this.expiresAt = LocalDateTime.now(clock).plusMonths(1);
    }

    void update() {
        this.updatedAt = LocalDateTime.now(clock);
    }
}
{% endhighlight %}

Now test would look like:
{% highlight groovy %}
class EntityTest extends Specification {

    def "should set proper dates"() {
        given:
        def fixedDateTime = LocalDateTime.of(2020, Month.AUGUST, 12, 10, 0)
        def clock = Clock.fixed(fixedDateTime.atZone(ZoneId.systemDefault()).toInstant(), ZoneId.systemDefault())

        when:
        def entity = new Entity(clock)

        then:
        entity.expiresAt == LocalDateTime.of(2020, Month.SEPTEMBER, 12, 10, 0)

        and:
        entity.update()

        then:
        entity.updatedAt == fixedDateTime
    }
}
{% endhighlight %}

I think, that this is a simple way to test time operations in our classes without any specific classes, using only JDK.

Hope you enjoy this post. If You have any questions or problems leave a comment or send email.

See You soon!