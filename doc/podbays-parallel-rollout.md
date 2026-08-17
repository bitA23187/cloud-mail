# Podbays Deployment

Last updated: 2026-08-17

`cloud-mail` is the production email service for `podbays.com`.

## Current state

- Worker: `cloud-mail`
- Workers URL: `https://cloud-mail.seunosleep.workers.dev`
- Primary domain: `https://mail.podbays.com`
- Alias domain: `https://cloudmail.podbays.com` (kept as legacy alias, can be removed)
- Mail domain: `podbays.com`
- Admin email: `admin@podbays.com`

## Bound Cloudflare resources

- D1: `cloud-mail-db`
- D1 ID: `48e537e4-7f7f-4fbe-8494-00fc7f82ee53`
- KV: `cloud-mail-kv`
- KV ID: `aed941c97a8d4167ae8567a2bf61b5e1`
- R2: `cloud-mail-r2`

## Local config used

- Worker config: `mail-worker/wrangler.podbays.toml`
- Frontend env: `mail-vue/.env.podbays`

## Email Routing

- Catch-all `*@podbays.com` → `cloud-mail` Worker
- MX records: `route1/2/3.mx.cloudflare.net`
- SPF: `v=spf1 include:_spf.mx.cloudflare.net ~all`
- Resend verified for `podbays.com` (and `lab.podbays.com`)

## GitHub automation

- Fork repo: `https://github.com/bitA23187/cloud-mail`
- Deployment branch: `deploy`
- Sync workflow: `Sync upstream into deploy`
- Deploy workflow: `Deploy cloud-mail to Cloudflare Workers`
- Sync source: `maillab/cloud-mail` branch `main`
- Sync cadence: weekly on Monday at 11:00 Asia/Shanghai (`0 3 * * 1` UTC)
- Auto deploy rule: only dispatch Cloudflare deployment when synced changes touch `mail-worker/**` or `mail-vue/**`
- Fork-owned paths (`.github/workflows`, `AGENTS.md`, and `doc/github-action.md`) are sourced from the fork's default-branch commit. Conflicts limited to those paths are resolved automatically; other conflicts still stop the sync for manual review.
- Deployment runtime: Node.js 24 and pnpm 11
- Required GitHub settings:
  - Actions workflow permissions must be set to `Read and write permissions`
  - Repository secrets or variables must include `NAME`, `CUSTOM_DOMAIN`, `DOMAIN`, `ADMIN`, `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_API_TOKEN`, `D1_DATABASE_ID`, `KV_NAMESPACE_ID`, `R2_BUCKET_NAME`, and `JWT_SECRET`

## Cutover log (2026-03-10)

Migrated from parallel test deployment to production:

1. Deleted old `mailfree` Worker and `maill_free_db` D1 database
2. Deployed `cloud-mail` on `mail.podbays.com` + `cloudmail.podbays.com`
3. Migrated D1: user/account emails `@lab.podbays.com` → `@podbays.com`, updated `r2Domain` to `mail.podbays.com/api/oss`, updated `resendTokens` to map `podbays.com`
4. Refreshed KV cache
5. Updated Email Routing catch-all to `cloud-mail`
6. Updated GitHub Actions variables for production

## Automation incident (2026-08-17)

The scheduled upstream sync failed because both the fork and upstream had changed `.github/workflows/deploy-cloudflare.yml`. The workflow attempted to restore fork-owned paths only after a successful merge, so it aborted before the restore logic could run.

Resolution:

1. Changed the sync workflow to restore fork-owned paths during conflict resolution as well as after clean merges.
2. Made the default branch the source of truth for fork-owned paths, preventing `main` and `deploy` workflow drift.
3. Updated the deploy workflow from Node.js 20 to 24 and pinned pnpm 11. This also fixed the prior Wrangler failure requiring Node.js 22 or newer.
4. Re-ran the sync and deployment successfully:
   - Sync run: `31996245559`
   - Deploy run: `31996260673`
   - Worker version: `d9568add-64c2-4327-9132-e3ffa792636e`
5. Verified `https://mail.podbays.com/` returned HTTP 200 and the public settings endpoint still reported `r2Domain=mail.podbays.com/api/oss` with `@podbays.com` as the active mail domain.

## Remaining cleanup

- Old freemail R2 buckets (`mail`, `mail-eml`) still exist with data — delete after emptying
- `cloudmail.podbays.com` route can be removed from `wrangler.podbays.toml` once no longer needed
- Consider DMARC record: `_dmarc.podbays.com TXT "v=DMARC1; p=none; rua=mailto:admin@podbays.com"`
