# Sample: "how does the crawl trigger work end to end?" — WITHOUT the skill

Model: Haiku 4.5, no reporting instructions, identical input facts.

---

Here's the end-to-end flow:

**Portal kicks it off:**
When someone triggers a crawl from the portal, `enqueue_job()` drops a message onto the SQS queue `crawler-pc-jobs` with `{task_id, type, payload}`. Then it's done — fire and forget.

**Collection PC wakes up:**
The dispatcher is sitting there doing a long-poll on that queue (20-second wait). When a message arrives, it gets consumed immediately.

**Preflight check (on startup only):**
First time the dispatcher starts, it runs the collector with no arguments — just making sure the venv is good. If it exits with code 2, great, we're ready. Code 1? That means the venv is broken, so the dispatcher bails without touching the queue.

**Chrome cleanup:**
Before running any job, the dispatcher kills any leftover Chrome process that might be holding the profile at `C:\crawler_profile`. That keeps things clean for a fresh run.

**Run the collector:**
Spawns `collector_articles/run.py "<company>"` with a hard 1800-second (30 min) timeout. The job runs, uploads whatever it parsed to S3 under its own prefix (the PC can only write to that — tight IAM scope).

**Outcome handling:**
- Exit 0? Message gets deleted from the queue. We're done.
- Nonzero exit or timeout? Message stays in the queue. SQS visibility timeout is 45 minutes, so it reappears later and the dispatcher tries it again.
- On the 3rd receive? SQS automatically moves it to the DLQ `crawler-pc-jobs-dlq` so it doesn't loop forever.
- Unknown message type? Logged and deleted immediately — so a typo in the type field doesn't poison the whole queue.

**Security model:**
The PC is deliberately outbound-only. No inbound ports. Its IAM user can only receive/delete on this one queue and put objects to its S3 prefix. So even if the PC gets compromised, it can't exfil data or access other infrastructure — it's scoped to exactly what it needs.

The beauty of it is that the portal never needs to know if the PC is up or idle — it just enqueues and moves on. The PC's fault tolerance is built into SQS's retry mechanism.
