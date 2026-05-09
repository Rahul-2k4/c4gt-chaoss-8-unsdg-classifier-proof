# CHAOSS #8 Failing Test - Bad Payload Proof

**Date**: 2026-05-04

## Test Case

Attempted to classify a DPG project by sending structured metadata fields directly:
```json
{
  "title": "African Storybook",
  "abstract": "Provides open access to picture storybooks...",
  "url": "https://..."
}
```

## Expected Result

HTTP 200 with SDG classification

## Actual Result

```
HTTP 500 Internal Server Error
```

## Root Cause

Aurora API expects a simple `{"text":"..."}` payload, not structured project metadata.

## Lesson / Fix Required

**Payload normalization** - convert project metadata into simple text before calling Aurora.

## Proof Files

- Raw response: `aurora-single-row-result.json` (shows both success and failure cases)
- Documented in: `PROGRESS_LOG.md`, `PROPOSAL_DRAFT.md`

## Status

This is **intentional proof** of a known limitation - proof that error handling work is needed.