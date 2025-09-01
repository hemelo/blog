---
title: 'Achieving Concurrency and Parallelism in Java'
published: 2025-09-01
draft: false
description: 'A practical, first‑person guide to building concurrent and parallel Java systems with Executors, CompletableFuture, virtual threads, and streams.'
tags: ['java', 'concurrency', 'parallelism', 'jvm', 'architecture', 'virtual-threads']
---

I’ve shipped Java services where the slow part was waiting on I/O, and others where the slow part was CPU. The toolkit is different in each case. Here’s how I approach both.

## TL;DR

- **Concurrency (I/O‑bound)**: structure code so multiple tasks make progress together. Use non‑blocking I/O, `CompletableFuture`, reactive APIs, or virtual threads.
- **Parallelism (CPU‑bound)**: use multiple cores. Use bounded executors, parallel streams for pure functions, and chunk work intentionally.
- **Measure first**: profile to know which one you need, then pick the simplest tool.

## Concurrency in Java (I/O‑bound)

### 1) CompletableFuture for fan‑out/fan‑in

```java
CompletableFuture<User> userF = httpGetUser(userId);
CompletableFuture<List<Order>> ordersF = httpGetOrders(userId);

UserSummary summary = userF.thenCombine(ordersF,
    (user, orders) -> new UserSummary(user, orders.size()))
    .get();
```

### 2) Structured concurrency with virtual threads (Project Loom, Java 21+)

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<User> userTask = scope.fork(() -> httpGetUser(userId));
    Subtask<List<Order>> ordersTask = scope.fork(() -> httpGetOrders(userId));
    scope.join().throwIfFailed();
    User user = userTask.get();
    List<Order> orders = ordersTask.get();
    return new UserSummary(user, orders.size());
}
```

Virtual threads give you the synchronous style with excellent scalability for I/O—without callback pyramids.

### 3) Reactive (WebClient, Reactor)

```java
Mono<User> userM = client.get().uri("/users/{id}", userId).retrieve().bodyToMono(User.class);
Mono<List<Order>> ordersM = client.get().uri("/users/{id}/orders", userId).retrieve().bodyToFlux(Order.class).collectList();

Mono<UserSummary> summaryM = Mono.zip(userM, ordersM, UserSummary::new);
```

Reactive shines for high‑fan‑out APIs and streaming; use it where your team is comfortable with the model.

## Parallelism in Java (CPU‑bound)

### 1) Executors with a bounded pool

```java
int workers = Runtime.getRuntime().availableProcessors();
ExecutorService pool = new ThreadPoolExecutor(
    workers, workers,
    0L, TimeUnit.MILLISECONDS,
    new ArrayBlockingQueue<>(workers * 2),
    new ThreadPoolExecutor.CallerRunsPolicy());

try {
    List<Future<Result>> futures = items.stream()
        .map(item -> pool.submit(() -> heavyCompute(item)))
        .toList();
    for (Future<Result> f : futures) f.get();
} finally {
    pool.shutdown();
}
```

### 2) Parallel streams for pure, CPU‑heavy transforms

```java
List<Result> results = items.parallelStream()
    .map(Compute::transform)
    .toList();
```

Be strict about purity (no shared mutable state) to avoid hidden contention.

### 3) Virtual threads are not for CPU parallelism

Virtual threads multiplex I/O efficiently but don’t speed up CPU work. For CPU‑bound tasks, prefer platform threads in bounded pools.

## Backpressure, timeouts, and cancellation

- **I/O paths**: timeouts on HTTP/DB, circuit breakers, bulkheads. Avoid thread‑pool starvation from blocking calls.
- **CPU paths**: bounded queues, chunk sizes that fit cache, metrics for p95/p99.
- **Cross‑cutting**: propagate deadlines; in Loom, use `ScopedValue` or explicit parameters.

## How I choose on a new service

1) Profile a baseline endpoint.
2) If waiting dominates, start with async (CompletableFuture or virtual threads). Keep code readable.
3) If CPU dominates, add bounded parallelism where it counts. Keep data structures immutable.
4) Re‑measure. Remove anything that didn’t move the needle.

That loop keeps designs simple and performance honest.
