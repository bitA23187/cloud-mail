# Cloud Mail Repo Notes

## Purpose

This repo is being used to migrate from the existing `freemail` deployment to `cloud-mail` on Cloudflare.
Do not break or replace the current `freemail` production service during testing.

## Current deployment state

- Existing service still active:
  - Worker: `mailfree`
  - URL: `https://mail.podbays.com/`
- Parallel `cloud-mail` test deployment is active:
  - Worker: `cloud-mail`
  - Workers URL: `https://cloud-mail.seunosleep.workers.dev`
  - Custom domain: `https://cloudmail.podbays.com/`
  - Test mail domain: `lab.podbays.com`
  - Admin email: `admin@lab.podbays.com`
- GitHub fork automation:
  - Fork repo: `https://github.com/bitA23187/cloud-mail`
  - Local remotes: `origin=bitA23187/cloud-mail`, `upstream=maillab/cloud-mail`
  - Deploy branch: `deploy`
  - Sync workflow: `Sync upstream into deploy`
  - Deploy workflow: `Deploy cloud-mail to Cloudflare Workers`
  - Sync cadence: weekly on Monday at 11:00 Asia/Shanghai (`0 3 * * 1` UTC)

## Cloudflare resources already created

- Account ID: `b532fa3dc7f4e9cc0f4528f6a2d4dd47`
- D1:
  - name: `cloud-mail-db`
  - id: `48e537e4-7f7f-4fbe-8494-00fc7f82ee53`
- KV:
  - name: `cloud-mail-kv`
  - id: `aed941c97a8d4167ae8567a2bf61b5e1`
- R2:
  - name: `cloud-mail-r2`

## Important local files

- Worker deploy config: `mail-worker/wrangler.podbays.toml`
- Frontend env: `mail-vue/.env.podbays`
- Deployment notes: `doc/podbays-parallel-rollout.md`
- Upstream deploy workflow: `.github/workflows/deploy-cloudflare.yml`
- Upstream docs for GitHub Action deploy: `doc/github-action.md`

## Constraints

- Keep `freemail` untouched until `cloud-mail` is fully verified.
- Do not move or overwrite `mail.podbays.com` yet.
- Do not delete existing Cloudflare resources for `mailfree` / `freemail`.
- Avoid committing secrets into the repo.

## What has been verified

- `cloud-mail` Worker is deployed.
- `cloudmail.podbays.com` returns the app.
- D1 schema is initialized.
- First admin account exists and login was verified.
- GitHub fork was created and wired as the local `origin`.
- GitHub Actions variables and secrets needed for the test deployment were configured on the fork.
- The first `Sync upstream into deploy` workflow run succeeded.
- The first auto-dispatched `Deploy cloud-mail to Cloudflare Workers` run on `deploy` succeeded.

## Pending tasks

1. Keep deployment isolated to the `cloud-mail` test environment until acceptance is complete.
2. Monitor future weekly sync runs and handle merge conflicts manually if they occur.
3. Continue manual setup outside GitHub:
   - Email Routing for `lab.podbays.com`
   - Resend sender domain verification
4. Run acceptance testing for inbound mail, outbound mail, login, and admin flows on `cloudmail.podbays.com`.
5. Only after acceptance testing, plan cutover from `freemail` to `cloud-mail`.

## Notes for the next session

- Start by checking the latest Actions runs on the fork before changing workflow logic.
- Read this file plus `doc/podbays-parallel-rollout.md` for the current Podbays test setup.
- Keep `mail-vue/.env.podbays` and `mail-worker/wrangler.podbays.toml` local-only; they are intentionally ignored by git.
- If GitHub-side changes are needed, continue using the existing deploy workflow rather than inventing a second deployment path.
