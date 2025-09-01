---
title: 'Concurrency vs Parallelism: The Difference That Changes Designs'
published: 2025-09-01
draft: false
description: 'A practical, first‑person explanation of concurrency vs parallelism with modern Java/.NET examples and trade‑offs.'
tags: ['concurrency', 'parallelism', 'architecture', 'java', 'csharp', 'dotnet', 'jvm', 'systems']
---

I used to mix concurrency and parallelism together. Then production traffic taught me the difference. Concurrency is about structure and progress; parallelism is about hardware and simultaneous execution. Mixing them up led me to overcomplicate simple IO‑bound code and underutilize CPUs in compute‑heavy jobs.

## TL;DR

- **Concurrency**: make progress on multiple tasks at once (design and coordination).
- **Parallelism**: run tasks at the same time on multiple cores (hardware utilization).
- **IO‑bound** work wants concurrency; **CPU‑bound** work wants parallelism.
- You can have one without the other, or both.

## How I explain it

- **Concurrency** is like a good barista: one person takes orders, starts drinks, and keeps everything moving. One worker, many in‑flight tasks.
- **Parallelism** is like four baristas working at once: multiple drinks truly being made simultaneously.

## Where I reach for each

- **Concurrency**: web servers handling many requests, pipelines juggling DB/cache calls, streaming consumers.
- **Parallelism**: CPU‑heavy transforms, image/video processing, analytics, crypto, ML inference.

## Java example

```java
// Concurrency for IO-bound work with CompletableFuture
CompletableFuture<User> userF = httpGetUser(userId);
CompletableFuture<List<Order>> ordersF = httpGetOrders(userId);
CompletableFuture<UserSummary> summaryF = userF.thenCombine(ordersF,
    (user, orders) -> new UserSummary(user, orders.size()));

UserSummary summary = summaryF.get();

// Parallelism for CPU-bound work with a fixed thread pool
ExecutorService pool = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
List<Future<Result>> results = items.stream()
    .map(item -> pool.submit(() -> heavyCompute(item)))
    .toList();
for (Future<Result> r : results) r.get();
pool.shutdown();
```

## C#/.NET example

```csharp
// Concurrency for IO-bound work
var userTask = http.GetFromJsonAsync<User>($"/users/{userId}", ct);
var ordersTask = http.GetFromJsonAsync<List<Order>>($"/users/{userId}/orders", ct);
await Task.WhenAll(userTask, ordersTask);
var summary = new UserSummary(userTask.Result!, ordersTask.Result!.Count);

// Parallelism for CPU-bound work
Parallel.ForEach(items, new ParallelOptions { MaxDegreeOfParallelism = Environment.ProcessorCount }, item =>
{
    var r = HeavyCompute(item);
    WriteResult(r);
});
```

## Trade‑offs I watch

- **Thread pools**: async reduces pressure for IO‑bound paths; CPU pools should match cores.
- **Backpressure**: bounded queues/timeouts for concurrency; work chunking for parallel loops.
- **Context**: tracing and metrics tell you if you’re stalling on IO or pegging CPU.

## How I choose

I ask: is the slow part waiting or computing?

- Waiting → design for concurrency first (async IO, batching, caches).
- Computing → design for parallelism (vectorization, chunking, bounded pools).
- Both → pipeline: async at the edges, parallel in the heavy middle.

That mental split keeps systems simpler and makes performance work measurable.
