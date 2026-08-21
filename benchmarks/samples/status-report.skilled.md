# Sample: end-of-session status report — WITH the skill

Model: Haiku 4.5, skill text injected, identical input facts.

---

**Verdict:** All tests passing (180/180). Schema mismatch fixed with backward-compatible read.

## Issue & Fix

| What | Detail |
|------|--------|
| Failures | 2 tests in `parsers/tests/test_feed_derive.py` |
| Error | `KeyError: 'article_slug'` at `derive.py:84` |
| Root cause | Manifest schema v2 renamed `article_slug` → `slug`; code still read old key |
| Fix | Read `slug` with fallback to `article_slug` for backward compat |
| File changed | `parsers/feed/derive.py` |

## Test Results

Before fix:
```
178 passed, 2 failed
```

After fix:
```
180 passed, 0 failed
```

## Deployment Impact

None. These are test-only failures in the S3 manifest read path; live code unaffected.

## Side Note

S3 listing in the test suite runs ~40s per test. If test runtime becomes a bottleneck, a local file cache could cut that, but not worth doing now unless you're hitting the tests repeatedly.
