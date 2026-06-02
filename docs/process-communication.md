# Process Communication

## Single-node

```
Client ──POST /process/{name}──► ProcSvc
                                     │
                               ProcessQueue          JobObserver
                               (HostedService)      (OTel spans +
                                     │ Task.Run      metrics)
                                     ▼                   ▲
                               ProcessJob ───────────────┘
                                     │
                                     │ Parallel.ForEachAsync  (MaxDOP = -1)
                              ┌──────┼──────┐
                           file₁  file₂  file₃   …
                              │      │      │
                         SemaphoreSlim.WaitAsync          ChannelJobEventSink
                         (MaxDegreeOfParallelism)    ◄──── Channel<JobEvent>
                              │                              (in-memory)
                        OsProcessRunner                          │
                              │                                  │
                        ProcessStarter (OS proc)                 │
                              │                                  │
                         native .so/.dll                         │
                                                                 │
Client ──DELETE /process/{id}──► job.Cancel()                    │
        (sets CancellationTokenSource on ProcessJob)             │
                                                                 │
Client ◄──SSE── GET /process/{id}/events ◄───────────────────────┘
        same instance reads Channel<T> directly
```

## Distributed (Redis + PostgreSQL)

```
Client ──POST /process/{name}──► ProcSvc (any instance)
                                     │
                         DistributedProcessQueue
                                     │ RPUSH
                                     ▼
                              ┌─────────────┐
                              │  Redis List │  (work queue)
                              └─────────────┘
                                     │ LPOP poll every 100 ms
                                     ▼
                         DistributedProcessQueue        JobObserver
                           (any instance)              (OTel spans +
                                     │ Task.Run         metrics)
                                     ▼                      ▲
                               ProcessJob ──────────────────┘
                                     │
                                     │ Parallel.ForEachAsync  (MaxDOP = -1)
                              ┌──────┼──────┐
                           file₁  file₂  file₃   …
                              │      │      │
                         SemaphoreSlim.WaitAsync          RedisStreamJobEventSink
                         (per-instance throttle)     ◄──── Redis Streams
                              │                              (cross-instance)
                        OsProcessRunner                          │
                              │                                  │
                        ProcessStarter (OS proc)                 │
                              │                                  │
                         native .so/.dll                         │
                                                                 │
Client ──DELETE /process/{id}──► DistributedProcessQueue         │
        sets Redis cancel key                                    │
             │                                                   │
             ▼  poll every CancellationPollIntervalMs            │
        processing instance detects key → job.Cancel()           │
                                                                 │
Client ◄──SSE── GET /process/{id}/events ◄───────────────────────┘
        any instance reads Redis Streams
```

## Key differences

|                      | Single-node              | Distributed                        |
|----------------------|--------------------------|------------------------------------|
| Work queue           | `Channel<T>` (in-memory) | Redis List (`RPUSH` / `LPOP`)      |
| Job events           | `ChannelJobEventSink`    | `RedisStreamJobEventSink`          |
| Event transport      | in-process memory        | Redis Streams                      |
| Cancellation         | `CancellationTokenSource`| Redis key, polled                  |
| SSE server           | must be same instance    | any instance                       |
| `SemaphoreSlim` scope| global (one process)     | per-instance                       |
| User store           | `UsersStorage` (JSON)    | `DatabaseUsersStorage` (PostgreSQL)|

The deepest difference: in single-node the event channel is an in-process `Channel<T>` so the SSE
endpoint must hit the same process that runs the job. In distributed mode Redis Streams decouple
producer from consumer, so any instance can serve SSE for any job running anywhere in the cluster.
