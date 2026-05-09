# 100-Row Benchmark Execution Plan — CHAOSS #8

## Target

Run the UN SDG classifier against 100 repository rows from the DPG registry to establish baseline accuracy.

**Target Accuracy: 85%** (not yet achieved — this is the goal)

## Input CSV Schema

| Column | Type | Description | Required |
|--------|------|-------------|----------|
| `repo_url` | string | GitHub URL (e.g., `https://github.com/chaoss/grimoirelab`) | ✅ |
| `repo_name` | string | Short name for display | ✅ |
| `description` | string | Repo description from GitHub API | ✅ |
| `primary_language` | string | Primary language tag | ✅ |
| `topics` | string | Comma-separated topics | optional |
| `readme_snippet` | string | First 500 chars of README | ✅ |

## Expected Output Schema

| Column | Type | Description |
|--------|------|-------------|
| `repo_name` | string | Input repo_name |
| `predicted_sdgs` | string | Comma-separated SDG IDs (e.g., "4,5,9") |
| `confidence_scores` | JSON | `{"1": 0.12, "4": 0.89, ...}` |
| `model_version` | string | Model identifier (e.g., `aurora-v1`) |
| `processed_at` | ISO8601 | Timestamp |
| `latency_ms` | int | Processing time |
| `status` | enum | `success`, `rate_limited`, `timeout`, `error` |
| `error_message` | string | Error details if failed |

## Resume Logic

The runner must support checkpoint-based resumption:

```python
# Checkpoint file: benchmark-checkpoint.json
{
  "completed_rows": [0, 1, 2, 5, 8],  # row indices done
  "last_successful": {
    "index": 8,
    "repo_name": "grimoirelab",
    "timestamp": "2026-05-08T10:30:00Z"
  },
  "rate_limit_reset_at": "2026-05-08T10:31:00Z"
}
```

**Resume behavior:**
1. Load checkpoint on start
2. Skip already-completed rows
3. If rate-limited, wait until `rate_limit_reset_at`
4. Save checkpoint after each successful row

## Rate-Limit Wrapper

```python
class RateLimitedRunner:
    def __init__(self, min_interval=1.0):
        self.min_interval = min_interval
        self.last_call = {}
    
    def call(self, provider, fn, *args):
        now = time.monotonic()
        elapsed = now - self.last_call.get(provider, 0)
        if elapsed < self.min_interval:
            time.sleep(self.min_interval - elapsed)
        try:
            result = fn(*args)
            self.last_call[provider] = time.monotonic()
            return {"status": "success", "result": result}
        except RateLimitError as e:
            return {"status": "rate_limited", "retry_after": e.retry_after}
        except TimeoutError:
            return {"status": "timeout", "error": "Provider timeout"}
        except Exception as e:
            return {"status": "error", "error": str(e)}
```

## Failure Categories

| Category | Expected % | Handling |
|----------|------------|----------|
| `success` | ~70-80% | Write result to output CSV |
| `rate_limited` | ~10-15% | Wait, retry up to 3 times |
| `timeout` | ~5-10% | Skip row, log timeout |
| `error` | ~5% | Skip row, log error, continue |

## Execution Command

```bash
python run_benchmark.py \
  --input dpg-repos-100.csv \
  --output benchmark-results-100.csv \
  --checkpoint benchmark-checkpoint.json \
  --rate-limit 1.0 \
  --max-retries 3 \
  --timeout 30
```

## Success Criteria

- 85% of rows classified with `success` status
- Each success includes confidence scores for all 17 SDGs
- Checkpoint enables safe resume after interruption
- Rate-limited rows retry automatically without data loss

## What This Does NOT Claim

- ❌ 100% accuracy (target is 85%)
- ❌ All 100 rows succeed (expect ~15-20% failure rate)
- ❌ Real-time performance (intentionally slow for safety)
- ✅ Reproducible benchmark with audit trail