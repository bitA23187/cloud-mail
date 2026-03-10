# Podbays Parallel Rollout

Last verified: 2026-03-10

This deployment is isolated from the existing `freemail` service and does not replace `mail.podbays.com`.

## Current state

- Worker: `cloud-mail`
- Workers URL: `https://cloud-mail.seunosleep.workers.dev`
- Custom domain: `https://cloudmail.podbays.com`
- Test mail domain: `lab.podbays.com`
- Admin email: `admin@lab.podbays.com`

## Bound Cloudflare resources

- D1: `cloud-mail-db`
- D1 ID: `48e537e4-7f7f-4fbe-8494-00fc7f82ee53`
- KV: `cloud-mail-kv`
- KV ID: `aed941c97a8d4167ae8567a2bf61b5e1`
- R2: `cloud-mail-r2`

## Local config used

- Worker config: `mail-worker/wrangler.podbays.toml`
- Frontend env: `mail-vue/.env.podbays`

## Verified

- The Worker is deployed and reachable on both the Workers URL and `cloudmail.podbays.com`.
- The D1 schema is initialized.
- The first admin account exists and can log in.
- The existing `freemail` resources remain separate.

## GitHub automation

- Fork repo: `https://github.com/bitA23187/cloud-mail`
- Deployment branch: `deploy`
- Sync workflow: `Sync upstream into deploy`
- Deploy workflow: `Deploy cloud-mail to Cloudflare Workers`
- Sync source: `maillab/cloud-mail` branch `main`
- Sync cadence: weekly on Monday at 11:00 Asia/Shanghai (`0 3 * * 1` UTC)
- Auto deploy rule: only dispatch Cloudflare deployment when synced changes touch `mail-worker/**` or `mail-vue/**`
- First bootstrap status: succeeded on 2026-03-10; `deploy` branch was created and initial automated deployment completed
- Required GitHub settings:
  - Actions workflow permissions must be set to `Read and write permissions`
  - Repository secrets or variables must include `NAME`, `CUSTOM_DOMAIN`, `DOMAIN`, `ADMIN`, `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_API_TOKEN`, `D1_DATABASE_ID`, `KV_NAMESPACE_ID`, `R2_BUCKET_NAME`, and `JWT_SECRET`

## Remaining manual steps

- Configure Cloudflare Email Routing for `lab.podbays.com` to deliver incoming mail to the `cloud-mail` Worker.
- Verify a sending domain in Resend and add the token in Cloud Mail settings before testing outbound mail.
- After acceptance testing, migrate the production web/mail entrypoints from `freemail` to `cloud-mail`.
