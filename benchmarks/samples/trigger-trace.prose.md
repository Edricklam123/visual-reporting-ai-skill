# Sample: live smoke-test trace — as prose

Same facts as `trigger-trace.timeline.md`, written as a paragraph.

---

I sent the test message from the dev PC at 09:48:4x. Within ten seconds the
queue showed one message in flight, meaning the dispatcher had already
received it. The dispatcher's S3 log confirmed the receive at 09:48:53 and
recorded `s3_known`. It then spawned the collector, which finished listing at
09:49:22 and exited with code 0. At the twenty-second check the queue showed
the message deleted, so the full round trip — send, receive, spawn, collect,
delete — completed successfully.
