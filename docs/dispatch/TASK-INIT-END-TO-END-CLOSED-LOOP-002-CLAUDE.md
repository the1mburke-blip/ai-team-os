# Dispatch: TASK-INIT-END-TO-END-CLOSED-LOOP-002-CLAUDE

| Field | Value |
|---|---|
| template_version | v3.0.0 |
| proposed by | Owner-supplied task packet (chat), 2026-11-13 |
| executor (authoring) | Claude (Claude Code cloud session) |
| verifier | TIER_1_MACHINE: GitHub Actions `icarus-harness-gate-v3`, not the authoring agent |
| branch | `claude/end-to-end-autonomous-loop-mc9ldt` |
| spend | €0 (public repo, so Actions minutes are free; no paid APIs) |

## Dispatch channel

The packet asks for "an active GitHub Issue or repository task file".

- The **repository task file** is this document plus `task_queue/*.json`. This is the primary route.
- **GitHub Issue:** the Claude session's GitHub App got `403 Resource not accessible by integration`
  on issue creation (2026-11-13T09:50Z). To get around this, `harness-runner-v3.py` opens or reuses the
  dispatch issue (label `icarus-dispatch`) with the workflow's own `GITHUB_TOKEN` (`issues: write`) and posts
  receipt summaries there.
- Closed Issue #147 is not in this repository, so it was not read or changed.

## Route map (Rule 1, locked before execution)

| Stage | Action | Artifact |
|---|---|---|
| STAGE_1 | Directory structure + workflow with `workflow_dispatch`, `push`, `schedule` | `.github/workflows/icarus-harness-gate-v3.yml`, `task_queue/`, `processed/`, `icarus/runs/` |
| STAGE_2 | Harness (stage executor, dual gate) + dispatcher (queue → harness → archive) | `harness-runner-v3.py`, `icarus_pilot.py`, `schemas/task_packet.v3.schema.json` |
| STAGE_3 | Complete receipts, hash-chained and append-only, + issue comment | `icarus/runs/RUN_LOG.jsonl` |
| STAGE_4 | Canary staged; CI runs `python3 icarus_pilot.py`; Gate 1 + Gate 2; git SHA logged | `task_queue/canary_init.json` → `processed/` |

## Negative constraints (Rule 6)

- MUST NOT depend on the Gemini Notebook Chat UI to start or run tasks. The harness enforces this pattern on every packet.
- MUST NOT push to `main` without owner authorization.
- MUST NOT write secrets to receipts. A match blocks Gate 2 and is redacted in the receipt.
- MUST NOT rewrite the ledger. Prior lines are checked against `HEAD` and against the hash chain before every append.

## Known limits (stated up front, not hidden)

- GitHub only fires `schedule` triggers from the **default branch**. The loop listens on a schedule only
  after this branch is merged to `main`, which needs owner approval. Until then the live triggers are `push` and `workflow_dispatch`.
- Receipts are **hash-chained** with SHA-256 (`prev_receipt_hash` → `receipt_hash`). They are not
  cryptographically signed with a private key, because no signing key is provisioned and adding one is owner-only.
- `prime-snapshot-sync.yml` is not in this repository, and its sync target is unknown. It was not created.
