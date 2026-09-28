# merdoss-sync — MERDOSS distributed coordination workspace

Single source of truth shared by every MERDOSS device/server (Atelier, v8).

## Layout

| Path | Purpose |
|---|---|
| `work/index.json` | Distributed work index — devices claim tasks here via CAS (`expectedSha`), one task per (repo, headSha) |
| `devices/<deviceRef>/heartbeat.json` | Per-device heartbeat: what it is working on right now |
| `artifacts/<workItemId>/` | Outputs produced for a claimed work item |
| `import/` | Raw state snapshots imported from individual machines (compressed) |
| `state/` | Canonical merged baselines |

## Restoring the merged baseline

`state/merged-baseline-2026-09-28.json` is a valid BackupService payload (schema 1.1, hash-sealed).
It contains the union of the cloud instance (2026-09-28) and the local device, deduplicated:
500 missions, 1380 evidence, 24 projects, 7 virtual employees, 2 capital accounts (treasury intact).

Restore on any device:

```
POST /api/backup/restore  { "file": "<path to downloaded baseline json>" }
```

## Rules

- Every device writes only to its own path (`devices/<deviceRef>/`, `artifacts/<ref>/`).
- The work index is updated with compare-and-swap; on 409 re-read and retry.
- Never commit secrets: tokens live in the vault (`GITHUB_TOKEN`, provider keys), not here.
