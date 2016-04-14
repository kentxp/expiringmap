# ExpiringMap
[![Build Status](https://travis-ci.org/jhalterman/expiringmap.svg)](https://travis-ci.org/jhalterman/expiringmap)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/net.jodah/expiringmap/badge.svg)](https://maven-badges.herokuapp.com/maven-central/net.jodah/expiringmap/)
[![License](http://img.shields.io/:license-apache-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)

A high performance, low-overhead, zero dependency, thread-safe [ConcurrentMap](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html) implementation that expires entries. Features include:

* [Expiration policies](#expiration-policies)
* [Variable expiration](#variable-expiration)
* [Expiration listeners](#expiration-listeners)
* [Lazy entry loading](#lazy-entry-loading)
* [Expiration Introspection](#expiration-introspection)

Supports Java 6+ though the documentation uses lambdas for simplicity.

## Usage

ExpiringMap allows you to create a map that expires entries after a certain time period:

```java
Map<String, Connection> map = ExpiringMap.builder()
  .expiration(30, TimeUnit.SECONDS)
  .build();
  
// Expires after 30 seconds
map.put("connection", connection);
```

#### Expiration Policies

Expiration can occur based on an entry's creation time or last access time:

```java
Map<String, Connection> map = ExpiringMap.builder()
  .expirationPolicy(ExpirationPolicy.ACCESSED)
  .build(); 
```

We can also specify an expiration policy for individual entries:

```java
map.put("connection", connection, ExpirationPolicy.CREATED);
```

And we can change policies on the fly:

```java
map.setExpirationPolicy("connection", ExpirationPolicy.ACCESSED);
```

#### Variable Expiration
        
Entries can have individually variable expiration times and policies:

```java
ExpiringMap<String, Connection> map = ExpiringMap.builder()
  .variableExpiration()
  .build();

map.put("connection", connection, ExpirationPolicy.ACCESSED, 5, TimeUnit.MINUTES);
```

Expiration times and policies can also be changed on the fly:

```java
map.setExpiration(connection, 5, TimeUnit.MINUTES);
map.setExpirationPolicy(connection, ExpirationPolicy.ACCESSED);
```

#### Expiration Listeners

Expiration listeners can be notified when an entry expires:

```java
Map<String, Connection> map = ExpiringMap.builder()
  .expirationListener((key, connection) -> connection.close())
  .build();
```

Expiration listeners are called synchronously as entries expire, and write operations to the map are blocked until the listeners complete. Asynchronous expiration listeners can also be configured and are called in a separate thread without blocking map operations:

```java
Map<String, Connection> map = ExpiringMap.builder()
  .asyncExpirationListener((key, connection) -> connection.close())
  .build();
```

Expiration listeners can also be added and removed on the fly:

```java
ExpirationListener<String, Connection> connectionCloser = (key, connection) -> connection.close();
map.addExpirationListener(connectionCloser);
map.removeExpirationListener(connectionCloser);
```

#### Lazy Entry Loading

Entries can be lazily loaded via an `EntryLoader` when `ExpiringMap.get` is called:

```java
Map<String, Connection> connections = ExpiringMap.builder()
  .expiration(10, TimeUnit.MINUTES)
  .entryLoader(address -> new Connection(address))
  .build();
  
// Loads a new connection into the map via the EntryLoader
connections.get("jodah.net");
```

Lazily loaded entries can also be made to expire at varying times:

```java
Map<String, Connection> connections = ExpiringMap.builder()
  .expiringEntry(address -> new ExpiringValue(new Connection(address), 5, TimeUnit.MINUTES))
  .build();
```

#### Expiration Introspection

ExpiringMap allows you to learn when an entry is expected to expire:

```java
long expiration = map.getExpectedExpiration("jodah.net");
```

We can also reset the internal expiration timer for an entry:

```java
map.resetExpiration("jodah.net");
```

And we can learn the configured expiration for individual entries:

```java
map.getExpiration("jodah.net");
```

## Additional Notes

#### On Variable Expiration

When variable expiration is disabled (default), `put` and `remove` operations have a constant O(1) cost. When variable expiration is enabled `put` and `remove` operations have a cost of O(log n).

#### On Google App Engine Integration

Google App Engine users must specify a `ThreadFactory` prior to constructing an `ExpiringMap` instance in order to avoid runtime permission errors:

```java

ExpiringMap.setThreadFactory(com.google.appengine.api.ThreadManager.currentRequestThreadFactory());
ExpiringMap.create();
```

See the [GAE docs on threads](https://cloud.google.com/appengine/docs/java/runtime#threads) for more info.

## Docs

JavaDocs are available [here](https://jhalterman.github.com/expiringmap/javadoc).

## License

Copyright 2009-2016 Jonathan Halterman - Released under the [Apache 2.0 license](http://www.apache.org/licenses/LICENSE-2.0.html).
