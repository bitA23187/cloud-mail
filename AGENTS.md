# Cloud Mail Repo Notes

## Purpose

This repo hosts the `cloud-mail` deployment on Cloudflare, serving production email at `podbays.com`.

## Current deployment state

- Production service:
  - Worker: `cloud-mail`
  - URL: `https://mail.podbays.com/`
  - Secondary URL: `https://cloudmail.podbays.com/` (alias, can be removed)
  - Workers URL: `https://cloud-mail.seunosleep.workers.dev`
  - Mail domain: `podbays.com`
  - Admin email: `admin@podbays.com`
- GitHub fork automation:
  - Fork repo: `https://github.com/bitA23187/cloud-mail`
  - Local remotes: `origin=bitA23187/cloud-mail`, `upstream=maillab/cloud-mail`
  - Deploy branch: `deploy`
- Sync workflow: `Sync upstream into deploy`
- Deploy workflow: `Deploy cloud-mail to Cloudflare Workers`
- Sync cadence: weekly on Monday at 11:00 Asia/Shanghai (`0 3 * * 1` UTC)
- Fork-owned automation and deployment docs are sourced from `main` during sync; conflicts in those paths are resolved automatically before upstream application changes are merged into `deploy`.
- Deployment runtime: Node.js 24 and pnpm 11.

## Cloudflare resources

- Account ID: `b532fa3dc7f4e9cc0f4528f6a2d4dd47`
- Zone ID (podbays.com): `dc8cae09338626064d21fb722bc8edba`
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

- Avoid committing secrets into the repo.
- Keep `mail-vue/.env.podbays` and `mail-worker/wrangler.podbays.toml` local-only; they are intentionally ignored by git.

## Cutover history (2026-03-10)

The old `freemail` service (`mailfree` Worker) was deleted and replaced by `cloud-mail`:

1. Deleted `mailfree` Worker, `maill_free_db` D1 database.
2. R2 buckets `mail` and `mail-eml` (old freemail data) still exist but are unused — can be deleted after emptying.
3. Deployed `cloud-mail` on `mail.podbays.com` (primary) and `cloudmail.podbays.com` (alias).
4. Migrated D1 records: user/account emails from `@lab.podbays.com` to `@podbays.com`, updated `r2Domain` and `resendTokens`.
5. Updated KV cache to match D1.
6. Email Routing catch-all for `@podbays.com` now points to `cloud-mail` Worker.
7. GitHub Actions variables updated: `DOMAIN=["podbays.com"]`, `ADMIN=admin@podbays.com`, `CUSTOM_DOMAIN=mail.podbays.com`.

## Notes for the next session

- Start by checking the latest Actions runs on the fork before changing workflow logic.
- Read this file plus `doc/podbays-parallel-rollout.md` for the deployment setup.
- If GitHub-side changes are needed, continue using the existing deploy workflow rather than inventing a second deployment path.
- `r2Domain` is set to `mail.podbays.com/api/oss`; images are served via the Worker `/api/oss/*` proxy, not a public R2 domain. If the setting reverts (e.g. after a re-deploy that reinitializes the DB), it must be restored in both D1 (`setting.r2_domain`) and KV cache (`setting:` key).
- Resend is fully working; both `lab.podbays.com` and `podbays.com` are verified domains. The active token maps `podbays.com`.
- Consider adding DMARC record: `_dmarc.podbays.com TXT "v=DMARC1; p=none; rua=mailto:admin@podbays.com"`
- Old freemail R2 buckets (`mail`, `mail-eml`) can be cleaned up when convenient.
- The 2026-08-17 upstream workflow conflict was fixed in commit `1e00591`; sync run `31996245559` and deploy run `31996260673` both succeeded.

## Knowledge System Layer

This repo now also has a lightweight local knowledge structure:

- `raw/`
- `notes/`
- `wiki/`

Default behavior for knowledge work:

1. inspect new material in `raw/`
2. use `notes/` for working synthesis if needed
3. promote durable understanding into `wiki/`
4. keep `wiki/index.md` and `wiki/log.md` current

Never ingest secret-bearing material such as `.env` values, tokens, or credential-bearing config into `wiki/`.
