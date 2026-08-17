# Upstream Sync Workflow Conflict — 2026-08-17

Type: incident  
Status: resolved

## Summary

The scheduled `Sync upstream into deploy` workflow failed while merging `maillab/cloud-mail:main` into the fork's `deploy` branch. Both sides had changed `.github/workflows/deploy-cloudflare.yml`, producing a content conflict.

Production remained available because the sync stopped before pushing `deploy` or dispatching a Cloudflare deployment.

## Root cause

The sync workflow restored fork-owned paths only after `git merge` succeeded. When one of those paths conflicted, the workflow aborted the merge before reaching its restore logic.

The fork's deployment workflow had also drifted from current runtime requirements: it selected Node.js 20 even though the installed Wrangler version required Node.js 22 or newer.

## Resolution

- Use the workflow-triggering default-branch commit as the source of truth for fork-owned paths.
- Restore those paths before evaluating whether unresolved conflicts remain.
- Continue automatically only when every remaining conflict was in a fork-owned path; otherwise abort for manual review.
- Propagate fork-owned path changes to `deploy` even when upstream has no new commits.
- Pin the deploy runtime to Node.js 24 and pnpm 11.

The fix was committed as `1e00591`. Sync run `31996245559` merged the upstream application changes into `deploy`, and deploy run `31996260673` published Worker version `d9568add-64c2-4327-9132-e3ffa792636e`.

## Verification

- The exact conflict was reproduced in a temporary worktree and resolved without unmerged files.
- All 29 changed application files were retained in the simulated merge.
- The deployed workflow retained the `deploy` branch trigger and selected Node.js 24 / pnpm 11.
- Both GitHub Actions runs completed successfully.
- `https://mail.podbays.com/` returned HTTP 200.
- `/api/setting/websiteConfig` reported `r2Domain=mail.podbays.com/api/oss` and `@podbays.com` as the active mail domain after database initialization.

## Runbook note

If a future sync still fails, inspect `git diff --name-only --diff-filter=U` in the failed run. Conflicts outside `.github/workflows`, `AGENTS.md`, and `doc/github-action.md` intentionally require manual reconciliation on `deploy`.
