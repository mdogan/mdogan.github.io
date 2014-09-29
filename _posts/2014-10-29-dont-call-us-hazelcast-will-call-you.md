---
layout: post
title: Don't call us, Hazelcast will call you when you arrive!
published: true
tags: java dependency injection hazelcast cdi jsr330 spring guice
comments: true
---

Sending tasks or application logic to the data is a good step to achieve greater scalability, generally you need to inject some local dependencies into remote tasks before they are executed. In Hazelcast, there are a few ways of injecting dependencies into user objects.

<!--excerpt-->

### Spring

Who don't use Spring nowadays... If you are a Spring user, you can also [configure Hazelcast with Spring](http://docs.hazelcast.org/docs/latest/manual/html/springintegration.html). By doing this you also get
ability of injecting Spring dependencies into your remote tasks (`Callable`s, `Runnable`s, `EntryProcessor`s etc). Only thing you need to do is mark those classes with `@SpringAware` annotation and Hazelcast will let Spring to inject dependencies just after a task arrives to the remote node.

```xml
<beans>
    <context:annotation-config />
    <hz:hazelcast id="hazelcast">
        <hz:config>
            ...
        </hz:config>
    </hz:hazelcast>

    <bean id="someBean" class="com.hazelcast.examples.spring.SomeBean" scope="singleton" />
    ...
</beans>
```

```java
@SpringAware
public class SomeTask implements Callable, ApplicationContextAware,
    java.io.Serializable {

    private transient ApplicationContext context;

    private transient SomeBean someBean;

    public Object call() throws Exception {
        assert context != null;
        assert someBean != null;

        // do something meaningful with someBean
        // ...
        return result;
    }

    public void setApplicationContext(ApplicationContext applicationContext)
        throws BeansException {
        context = applicationContext;
    }

    @Autowired
    public void setSomeBean(final SomeBean someBean) {
        this.someBean = someBean;
    }
}
```

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
HazelcastInstance hazelcast = (HazelcastInstance) context.getBean("hazelcast");
hazelcast.getExecutorService().submit(new SomeTask());
```


### ManagedContext

[`ManagedContext`](http://docs.hazelcast.org/docs/latest/javadoc/com/hazelcast/core/ManagedContext.html) is used to initialize an object upon it's created. By default Hazelcast has two out-of-the-box implementation of it, one is used to inject `HazelcastInstance` into `HazelcastInstanceAware` objects and the other one is to inject Spring dependencies. Apart from these one can implement its own `ManagedContext` to inject its dependencies using its own way or using any other DI framework, guice etc.

`ManagedContext` is a single-abstract-method interface, it's very trivial to implement.

```java
/**
 * Container managed context, such as Spring, Guice and etc.
 */
public interface ManagedContext {
    /**
     * Initialize the given object instance.
     * This is intended for repopulating select fields and methods for deserialized instances.
     * It is also possible to proxy the object, e.g. with AOP proxies.
     *
     * @param obj Object to initialize
     * @return the initialized object to use
     */
    Object initialize(Object obj);
}
```

A sample implementation that injects JPA `EntityManager` into data access objects:

```java
public interface DAO {
    void setEntityManager(EntityManager entityManager);
    ...
}

public class JPAManagedContext implements ManagedContext {
    final EntityManagerFactory factory;

    JPAManagedContext(EntityManagerFactory factory) {
        this.factory = factory;
    }

    public Object initialize(Object obj) {
        if (obj instanceof DAO) {
            EntityManager entityManager factory.createEntityManager();
            ((DAO) obj).setEntityManager(entityManager);
        }
        return obj;
    }
}
```

You should enable `ManagedContext` implementation in Hazelcast configuration:

```java
Config config = new Config();
config.setManagedContext(new JPAManagedContext());
```

### Manual way: User context map

Each Hazelcast instance has its own user context map, which can be initialized in configuration and can be accessed directly from `HazelcastInstance` interface. It's just a plain `ConcurrentMap<String, Object>`.

```java
ConcurrentMap<String, Object> context = new ConcurrentHashMap<String, Object>();
context.put("key1", value1);
context.put("key2", value2);

Config config = new Config();
config.setUserContext(context);

HazelcastInstance hazelcastInstance = Hazelcast.newHazelcastInstance(config);
```

You can access that context map inside a distributed task by implementing [`HazelcastInstanceAware`](http://docs.hazelcast.org/docs/latest/javadoc/com/hazelcast/core/HazelcastInstanceAware.html) interface.

```java
public class SomeTask implements Callable, HazelcastInstanceAware,
    java.io.Serializable {

    private transient HazelcastInstance hazelcastInstance;

    public Object call() throws Exception {
        assert hazelcastInstance != null;

        ConcurrentMap<String, Object> context = hazelcastInstance.getUserContext();
        Object value1 = context.get("key1");
        Object value2 = context.get("key2");
        // do something meaningful
        // ...
        return result;
    }

    public void void setHazelcastInstance(HazelcastInstance hazelcastInstance) {
        this.hazelcastInstance = hazelcastInstance;
    }
}
```

Now submit all your remote tasks and let Hazelcast execute them in place...
