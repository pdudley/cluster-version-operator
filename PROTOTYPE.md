# OCPBUGS-14392 Prototype — Option 2

Throwaway worktree for evaluating whether `SyncWorkerStatus.String()` plus `%s` logging makes `Failure` readable in CVO logs.

**Branch:** `proto/ocpbugs-14392-opt2` (if pushed)

## What changed

- Added `SyncWorkerStatus.String()` and `LoadPayloadStatus.String()` in `pkg/cvo/sync_worker.go`
- Switched key status log lines from `%#v` to `%s` / `%v` so Stringer output is used

## Diff against main

From this worktree root:

```bash
git diff main -- pkg/cvo/sync_worker.go pkg/cvo/status.go
```

Or compare the whole prototype:

```bash
git diff main
```

## Do not merge

This is an experiment to answer: does curated String() output make sync failures easier to read than Go struct dumps? Discard after evaluation unless promoted to a real fix.
