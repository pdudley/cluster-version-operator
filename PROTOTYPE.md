# OCPBUGS-14392 Prototype: Option 2+3 Hybrid

**Branch:** `proto/ocpbugs-14392-opt2-3` (throwaway)  
**Worktree:** `.worktrees/opt23`

## Question

Does `InfoS` + `SyncWorkerStatus.String()` give the best of both worlds — structured logging with a curated, readable status summary?

## Options Recap

| Option | Approach | Pros | Cons |
|--------|----------|------|------|
| **1** | `Infof` with `%#v` | Simple | Unreadable dumps; no structure |
| **2** | `String()` with curated format | Readable, stable shape | Loses per-field structure for log parsers |
| **3** | `InfoS` with exploded fields | Fully structured | Verbose; many fields; hard to curate |
| **2+3** | `InfoS` + `String()` as one value | Structured envelope + curated payload | `status` is opaque string to parsers |

## Hybrid Design

```go
klog.V(level).InfoS("Synchronizing status",
    "errs", validationErrs,
    "status", status.String(),
)
```

- **Option 3** provides the structured `InfoS` envelope: message key, separate `errs` field, separate `status` field.
- **Option 2** owns the human-readable dump via `SyncWorkerStatus.String()` — one curated string, not 15 exploded keys.
- Callers pass `status.String()` explicitly so klog never falls back to `%#v`-style formatting on a pointer.

## Expected Log Shape

```
I0804 16:40:00.123456       1 status.go:165] Synchronizing status errs=[] status="generation=5 done=42/100 completed=2 reconciling=true initial=false version=\"4.16.0\" image=\"quay.io/openshift-release-dev/ocp-release@sha256:abc...\" failure=\"<none>\" loadPayloadStep=\"\" loadPayloadFailure=\"<none>\""
```

With validation errors at V(2):

```
I0804 16:40:00.123456       1 status.go:165] Synchronizing status errs=[spec.desired.version: Invalid value: "": version must be set] status="generation=0 done=0/0 completed=0 reconciling=false initial=true version=\"\" image=\"\" failure=\"<none>\" loadPayloadStep=\"\" loadPayloadFailure=\"<none>\""
```

Previous sync status (sync_worker.go):

```
I0804 16:40:00.123456       1 sync_worker.go:716] Previous sync status status="generation=4 done=100/100 completed=1 reconciling=false initial=false version=\"4.15.0\" image=\"quay.io/...\" failure=\"<none>\" loadPayloadStep=\"done\" loadPayloadFailure=\"<none>\""
```

## Files Changed

- `pkg/cvo/sync_worker.go` — `SyncWorkerStatus.String()`, `InfoS` for previous status
- `pkg/cvo/status.go` — `InfoS` for synchronizing status

## Verdict (fill in after review)

- [ ] Readable enough for on-call triage?
- [ ] `errs` still structured enough when non-empty?
- [ ] Prefer this over pure Option 3 field explosion?
