---
title: "Integrating Caffeine async cache loader with reactive data source"
author: yiqi
permalink: /blogs/2023-09-11-caffeine-async-reactive
date: 2023-09-11
collection: blogs
tag:
- Programming
- Java
---

***
This post assumes you know the basic concepts and usage of [Project Reactor](https://projectreactor.io)  

***

I recently had a task to add Caffeine cache in a WebFlux service. Since everything is async in the service, it was natural to think of Caffeine's async cache loader. It took me some time to get it work.  

Here's what I got to started with, as well as requirements:
* The data source of the cache is a reactive Feign client. Thus the api gives a ```Mono``` or ```Flux```.  
* Caffeine async cache loader works in the world of ```CompletableFuture```.  
* The business logic level is all reactive.  

So, you might already see the problem: How to do the transform of Mono/Flux -> CompletableFuture -> Mono/Flux while integrating with the async cache loader?

## [BAD] Blocking?  

At first, I thought "How hard it could be? I've used Caffeine cache loader quite a few times." And I came up with the following code.  

```java
@Bean
public AsyncLoadingCache<UUID, List<DataObject>> asyncCache() {
    return Caffeine.newBuilder()
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .maximumSize(500)
        .buildAsync(key -> reactiveFeignClient.getDataById(key)
                .blockOptional()
                .orElse(List.of()));
}
```

Well, though it's straightforward and easy to implement, it's a really bad code because of the use of ```blockOptional```. It's quite obvious that blocking is never the right thing to do in reactive world.  

## [GOOD] True async implementation

```java
@Bean
public AsyncLoadingCache<UUID, List<DataObject>> asyncCache() {
    return Caffeine.newBuilder()
        .expireAfterWrite(5, TimeUnit.MINUTES)
        .maximumSize(500)
        .buildAsync((key, executor) -> reactiveFeignClient.getDataById(key)
                .subscribeOn(Schedulers.fromExecutor(executor))
                .toFuture());
}
```

Instead of blocking the reactor, subscribe it on the cache loader's own thread by ```subscribeOn(Schedulers.fromExecutor(executor))```.  

On the business logic level, do the following to transform the cache back to Mono.  

```java
Mono.fromFuture(cache.get(key));
```

## [EVEN BETTER] Abstracted cache layer

Let's make the cache layer generic so that it can get plugged in anywhere.  

```java
public class AsyncCacheService<K, V> {
    private final AsyncLoadingCache<K, V> cache;

    /**
     * Constructor. Build the cache loader.
     * 
     * @param duration Caffeine expireAfterWrite value.
     * @param fn       Cache data loading function.
     */
    public AsyncCacheService(Duration expireAfterWrite, Function<K, Mono<V>> fn) {
        cache = Caffeine.newBuilder()
                .expireAfterWrite(expireAfterWrite)
                .buildAsync((key, executor) -> fn.apply(key)
                        .subscribeOn(Schedulers.fromExecutor(executor))
                        .toFuture());
    }

    public Mono<V> get(K k) {
        return Mono.fromFuture(cache.get(k));
    }
}
```