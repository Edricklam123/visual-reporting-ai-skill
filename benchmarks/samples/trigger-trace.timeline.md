# Sample: live smoke-test trace — as timeline diagram

Same facts as `trigger-trace.prose.md`, drawn as a sequence timeline.

---

```
me (dev PC) ──send──► crawler-pc-jobs ──receive──► dispatcher ──spawn──► collector
   09:48:4x           in_flight=1 @10s             09:48:53 s3_known     09:49:22 listed
                      deleted    @20s   ◄── exit 0 ─────────────────────────────┘
```
