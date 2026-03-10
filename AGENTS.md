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

## Pending tasks

1. Set up "auto sync upstream + auto deploy to Cloudflare" using the user's fork.
2. Keep deployment isolated to `cloud-mail` test environment first.
3. Configure GitHub Actions secrets/variables for the fork:
   - `NAME=cloud-mail`
   - `CUSTOM_DOMAIN=cloudmail.podbays.com`
   - `DOMAIN=["lab.podbays.com"]`
   - `ADMIN=admin@lab.podbays.com`
   - `CLOUDFLARE_ACCOUNT_ID`
   - `CLOUDFLARE_API_TOKEN`
   - `D1_DATABASE_ID`
   - `KV_NAMESPACE_ID`
   - `R2_BUCKET_NAME`
   - `JWT_SECRET`
4. Add an upstream sync workflow or bot so the fork can regularly pull from `maillab/cloud-mail`.
5. Ensure sync to the fork's deploy branch triggers `.github/workflows/deploy-cloudflare.yml`.
6. After automation is stable, continue manual setup outside GitHub:
   - Email Routing for `lab.podbays.com`
   - Resend sender domain verification
7. Only after acceptance testing, plan cutover from `freemail` to `cloud-mail`.

## Notes for the next session

- The user wants to continue with the automation setup in a fresh session because the previous context became too long.
- Start by checking current git status and reading this file plus `doc/podbays-parallel-rollout.md`.
- If GitHub-side changes are needed, prefer using the repo's existing deploy workflow rather than inventing a second deployment path.
