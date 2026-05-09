# Rate-Limit Wrapper Prototype

Status: design/prototype note for proposal proof. Not yet implemented in repo.

## Requirement

Issue #8 notes that Aurora and OSDG API usage should stay near one request per second to avoid too-many-requests failures.

## Proposed Shape

Add a shared backend wrapper for external classifier calls:

- one request per provider per second;
- retry/backoff for transient failures;
- timeout with clear error reason;
- benchmark cache keyed by provider + normalized input text;
- structured response envelope so the frontend can show useful errors.

## Pseudocode

```python
import time
from functools import wraps

LAST_CALL_AT = {}

def rate_limited(provider, min_interval_seconds=1.0):
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            now = time.monotonic()
            last = LAST_CALL_AT.get(provider, 0)
            wait_for = min_interval_seconds - (now - last)
            if wait_for > 0:
                time.sleep(wait_for)
            try:
                result = fn(*args, **kwargs)
                LAST_CALL_AT[provider] = time.monotonic()
                return {
                    "ok": True,
                    "provider": provider,
                    "result": result,
                    "error": None,
                }
            except Exception as exc:
                LAST_CALL_AT[provider] = time.monotonic()
                return {
                    "ok": False,
                    "provider": provider,
                    "result": None,
                    "error": str(exc),
                }
        return wrapper
    return decorator
```

## Benchmark Cache Sketch

```python
cache_key = f"{provider}:{hash(normalized_project_text)}"
if cache_key in benchmark_cache:
    return benchmark_cache[cache_key]
result = call_provider(project_text)
benchmark_cache[cache_key] = result
return result
```

## Proposal Implication

The 100-row DPG benchmark should not hammer external services. A queued/cached runner makes the benchmark repeatable, slower but safer, and easier for maintainers to audit.
