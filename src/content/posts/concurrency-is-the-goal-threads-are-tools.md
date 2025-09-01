---
title: 'Concurrency Is the Goal: Threads Are Just One Tool'
published: 2025-09-01
draft: false
description: 'A first‑person take on designing for concurrency and choosing between threads, processes, and async—with modern Java/C# examples.'
tags: ['concurrency', 'multithreading', 'async', 'systems', 'architecture', 'java', 'csharp', 'dotnet', 'jvm']
---

I used to say “we’ll multithread it” when what I meant was “we need concurrency.” That shortcut cost me bugs and late nights in production. Concurrency is a design goal. Threads, processes, and async are just ways to reach it.

## TL;DR

- **Concurrency is the goal**: make progress on multiple tasks at once.
- **Threads, processes, async are tools**: pick based on risks and constraints.
- **On services and apps**: async/non‑blocking I/O scales better; use threads when CPU‑bound or stuck with blocking libraries.
- **Threads shine** when you truly need parallel CPU or you’re stuck with blocking libraries.

## What I mean by concurrency

Concurrency is about structuring work so multiple tasks advance together: serving HTTP while hitting a database and a cache, refreshing a feed while streaming events, coordinating background jobs without stalling request paths. You can achieve that with different tools, each with trade‑offs.

## The tools I reach for (and when)

### Multithreading (shared memory)

- **Why it’s useful**: great when you have multi‑core CPUs or libraries that block and you can’t rewrite them.
- **Costs**: race conditions, deadlocks, priority inversion, per‑thread stacks that add up quickly.
- **My rule**: only reach for threads when parallel compute or immovable blocking I/O justifies the complexity.

### Multiprocessing (isolated memory)

- **Why it’s useful**: isolation and crash containment; clean CPU scaling on servers.
- **Costs**: heavier on memory/IPC; solid fit on Linux servers and containers.
- **My rule**: use processes when one component must never be allowed to corrupt another.

### Asynchronous programming (event loop/state machine)

- **Why it’s useful**: single thread, cooperative tasks, low memory, fewer synchronization bugs.
- **Costs**: you must design state machines clearly; blocking calls become explicit awaits or callbacks.
- **My rule**: default to async for I/O‑bound systems, especially in high‑concurrency web services on the JVM or .NET.

## Lessons from JVM/.NET projects

- **Thread pool exhaustion is sneaky**: a single blocking call in a hot path can starve the pool. Switching to an async DB/HTTP client or offloading to a dedicated pool fixed tail latencies.
- **Context switches add up**: fine‑grained threads looked elegant but hurt throughput. Using async pipelines (C# `async`/`await`) or fewer, well‑bounded executors (Java) simplified flow and improved p99.
- **Deterministic backpressure matters**: bounded queues and timeouts kept the system stable under load; cancellation tokens (C#) and timeouts (Java) made intent explicit.
- **Debuggability improves with intent**: with async, the ugly bugs were in my state transitions, not lock ordering. Structured logging and tracing made stack hops understandable.

## Modern example: concurrent I/O in .NET and Java

```csharp
public async Task<UserSummary> GetUserSummaryAsync(Guid userId, HttpClient http, CancellationToken ct)
{
    Task<User?> userTask = http.GetFromJsonAsync<User>($"/api/users/{userId}", ct);
    Task<List<Order>?> ordersTask = http.GetFromJsonAsync<List<Order>>($"/api/users/{userId}/orders", ct);

    await Task.WhenAll(userTask, ordersTask);

    User user = userTask.Result ?? throw new InvalidOperationException("User not found");
    List<Order> orders = ordersTask.Result ?? new List<Order>();

    return new UserSummary(user, orders.Count);
}
```

Same idea in Java with `CompletableFuture` and `allOf`, or with virtual threads if you prefer a threads‑per‑request model.

## How I choose in practice

- **Pick async** when memory is tight, tasks are I/O‑bound, and timing needs to be predictable.
- **Pick threads** when you need true parallel CPU or must integrate blocking code you can’t change.
- **Pick processes** when you want isolation or crash containment across components.
- **Measure first**: profile the slow path and let numbers drive the choice, not habit.

## Words I use with teams

I try to say “we need concurrency” and then ask “what’s the simplest tool that achieves it with the risks we can live with?” Most of the time, that conversation lands on async. When we do pick threads, we write down why, set budgets for stacks, choose synchronization primitives intentionally, and add tests for the race conditions we’re inviting.

## Closing thought

Concurrency is the outcome you want; threads are just one way to get there. Pick the tool that keeps your system simple, testable, and predictable—especially when your thread pool graphs are screaming.
