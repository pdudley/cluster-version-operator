# OCPBUGS-14392 Prototype — Option 3

Throwaway worktree for evaluating whether `klog.InfoS` structured logging makes `Failure` readable without a `SyncWorkerStatus.String()` method.

**Branch:** `proto/ocpbugs-14392-opt3` (if pushed)

## Question

Does listing `failure` as an explicit InfoS key field produce readable error text in CVO logs, without adding `String()` on `SyncWorkerStatus`?

## What changed

- Replaced `%#v` struct dumps with `klog.InfoS` and explicit key/value pairs in `pkg/cvo/status.go` (`syncStatus`) and selected sites in `pkg/cvo/sync_worker.go`
- **No** `SyncWorkerStatus.String()` — that is Option 2 / hybrid

## Diff against main

From this worktree root:

```bash
git diff main -- pkg/cvo/status.go pkg/cvo/sync_worker.go
```

## Do not merge

This is an experiment to compare structured logging against Option 2 (`String()`). Discard after evaluation unless promoted to a real fix.
