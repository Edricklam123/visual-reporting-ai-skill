# Sample: "how does the crawl trigger work end to end?" — WITH the skill

Model: Haiku 4.5, skill text injected, identical input facts.

---

# Crawl Trigger: End to End

**Portal to PC:** The portal's `enqueue_job()` drops one SQS message onto `crawler-pc-jobs` (standard queue, not FIFO). Message shape: `{task_id, type, payload}`.

**PC dispatcher polls and executes:**

```
                        ┌─────────────────────────────┐
                        │ Startup: Preflight check    │
                        │ Run collector (no args)     │
                        └──────────┬──────────────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
            Exit 2            Exit 1             Other
          (healthy)         (broken venv)       (error)
                 │                 │                 │
           ┌─────▼─┐          ┌────▼─────┐      ┌───▼──┐
           │ Ready │          │ Dispatcher│     │Crash │
           │to poll│          │ exits     │     └──────┘
           └─────┬─┘          └───────────┘
                 │
                 │ Long-poll queue (WaitTimeSeconds=20)
                 │
          ┌──────▼────────────────────┐
          │ Message received          │
          │ {task_id, type, payload}  │
          └──────┬────────────────────┘
                 │
         ┌───────▼────────┐
         │ Unknown type?  │
         └───┬──────────┬─┘
        Yes  │          │ No
             │          │
    ┌────────▼───┐   ┌──▼─────────────────────┐
    │ Log+Delete │   │ Kill Chrome profile    │
    │ (no poison)│   │ C:\crawler_profile     │
    └────────────┘   │                        │
                     │ Spawn:                 │
                     │ collector_articles/    │
                     │ run.py "<company>"     │
                     │ Timeout: 1800s         │
                     └───┬──────────────────┬─┘
                         │                  │
                     Exit 0            Nonzero/timeout
                         │                  │
             ┌───────────▼┐             ┌──┴──────────────┐
             │DeleteMsg   │             │ Leave message   │
             │(consumed)  │             │ in queue        │
             └────────────┘             │ Visibility: 45m │
                                        └──┬──────────────┘
                                           │
                                      ┌────▼────────┐
                                      │ Receive #3? │
                                      └────┬─────┬──┘
                                     Yes   │     │ No → loops
                                      ┌────▼───┐
                                      │ Move to│
                                      │ DLQ    │
                                      └────────┘
```

**Failure modes:**

| Failure | Exit code/signal | Dispatcher action | Result |
|---------|------------------|-------------------|--------|
| Chrome won't launch | Nonzero | Don't delete; visibility resets | Retry #2 and #3, then DLQ |
| Timeout (slow crawl) | Timeout (1800s exceeded) | Don't delete; visibility resets | Same: retry → DLQ |
| Collector broken | Exit 1 at startup (preflight) | Dispatcher exits entirely | Queue ignored; manual restart needed |
| Typo in type field | Any | Log it, delete message | Won't poison queue; move on |
| Success | Exit 0 | DeleteMessage | Consumed, done |

**Key constraint:** The PC is outbound-only (no inbound ports, no SendMessage IAM). It can only:
- Receive and delete from `crawler-pc-jobs`
- PutObject to its own S3 prefix (`raw/{source}/*`)

So the only way for the portal to trigger the PC is via SQS; the PC cannot dial back.
