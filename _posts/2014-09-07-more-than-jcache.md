---
layout: post
title: More than just JCache
published: true
tags: java cache hazelcast jcache jsr107 async
comments: true
---

[JSR107](https://github.com/jsr107/jsr107spec), *Java Temporary Caching API*, also known as *JCache* is finally out after [an effort of 13 years](http://sdtimes.com/13-years-jcache-specification-finally-complete/). It's like the only everlasting JSR of the JCP history. But long story short, it's completed now and companies in NoSQL/in-memory-data-grid/big-data world are competing each other to release their own implementations first.

As *JCache* being only a spec, it has a very basic API. But on the other hand it provides a standard way to access the proprietary features that an implementor can add to extend standard API.

<!--excerpt-->

```java
Cache cache = ...;
SomeProprietaryCacheImpl cacheImpl = cache.unwrap(SomeProprietaryCacheImpl.class);
cacheImpl.doSomethingSpecial(withSomeArgs);

```

### There's more than *just JCache*

Hazelcast, leading open source in memory data grid, will include *JCache* support by upcoming version 3.3.1. Apart from standard *JCache* api, Hazelcast will provide a feature rich extension api, including async versions of each method in spec.

**Introducing [com.hazelcast.cache.ICache](https://github.com/hazelcast/hazelcast/blob/master/hazelcast/src/main/java/com/hazelcast/cache/ICache.java) !**

(Don't get this wrong, not like an Apple product, it's just like `IMap`, `IQueue`...)

```java
/**
 * Hazelcast cache extension interface
 */
public interface ICache<K, V> extends javax.cache.Cache<K, V> {

    /**
     * Asynchronously get an entry from cache
     */
    Future<V> getAsync(K key);

    /**
     * Asynchronously get an entry from cache
     * with a provided expiry policy
     */
    Future<V> getAsync(K key, ExpiryPolicy expiryPolicy);

    /**
     * Asynchronously associates the specified value
     * with the specified key in the cache.
     */
    Future<Void> putAsync(K key, V value);

    /**
     * Asynchronously associates the specified value
     * with the specified key in the cache
     * using a custom expiry policy
     */
    Future<Void> putAsync(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * Asynchronously associates the specified value
     * with the specified key in the cache if not already exist
     * using a custom expiry policy
     */
    Future<Boolean> putIfAbsentAsync(K key, V value,
                                    ExpiryPolicy expiryPolicy);

    /**
     * Asynchronously associates the specified value
     * with the specified key in this cache,
     * returning an existing value if one existed.
     */
    Future<V> getAndPutAsync(K key, V value);

    /**
     * Asynchronously associates the specified value
     * with the specified key in this cache,
     * returning an existing value if one existed
     * using a custom expiry policy
     */
    Future<V> getAndPutAsync(K key, V value,
                            ExpiryPolicy expiryPolicy);

    /**
     * Asynchronously removes the mapping for a key
     * from this cache if it is present.
     */
    Future<Boolean> removeAsync(K key);

    /**
     * Asynchronously removes the mapping for a key
     * only if currently mapped to the given value.
     */
    Future<Boolean> removeAsync(K key, V oldValue);

    /**
     * Asynchronously removes the entry for a key
     * returning the removed value if one existed.
     */
    Future<V> getAndRemoveAsync(K key);

    /**
     * Asynchronously replaces the entry for a key
     * only if currently mapped to a given value.
     */
    Future<Boolean> replaceAsync(K key, V oldValue, V newValue);

    /**
     * Asynchronously replaces the entry for a key
     * only if currently mapped to a
     * given value using custom expiry policy
     */
    Future<Boolean> replaceAsync(K key, V oldValue, V newValue,
                                ExpiryPolicy expiryPolicy);

    /**
     * Asynchronously replaces the entry for a key
     * only if currently mapped to some
     */
    Future<V> getAndReplaceAsync(K key, V value);

    /**
     * Asynchronously replaces the entry for a key
     * only if currently mapped to some
     * using custom expiry policy
     */
    Future<V> getAndReplaceAsync(K key, V value,
                                ExpiryPolicy expiryPolicy);

    /**
     * Get with custom expiry policy
     */
    V get(K key, ExpiryPolicy expiryPolicy);

    /**
     * getAll with custom expiry policy
     */
    Map<K, V> getAll(Set<? extends K> keys,
                    ExpiryPolicy expiryPolicy);

    /**
     * put with custom expiry policy
     */
    void put(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * getAndPut with custom expiry policy
     */
    V getAndPut(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * putAll with custom expiry policy
     */
    void putAll(java.util.Map<? extends K, ? extends V> map,
                ExpiryPolicy expiryPolicy);

    /**
     * putIfAbsent with custom expiry policy
     */
    boolean putIfAbsent(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * replace with custom expiry policy
     */
    boolean replace(K key, V oldValue, V newValue,
                    ExpiryPolicy expiryPolicy);

    /**
     * replace with custom expiry policy
     */
    boolean replace(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * getAndReplace with custom expiry policy
     */
    V getAndReplace(K key, V value, ExpiryPolicy expiryPolicy);

    /**
     * total cache size
     */
    int size();
}
```

Go distributed, go async!
