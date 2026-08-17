# Podbays Acceptance Checklist

Last updated: 2026-08-17

This checklist covers the production `cloud-mail` deployment on `mail.podbays.com`.

## Current external follow-ups

- Inbound Email Routing and Resend sending are active for `podbays.com`.
- DMARC is still optional follow-up work: `_dmarc.podbays.com TXT "v=DMARC1; p=none; rua=mailto:admin@podbays.com"`.

## Latest GitHub automation status

- `Sync upstream into deploy` run `31996245559`: `Success` on 2026-08-17
- `Deploy cloud-mail to Cloudflare Workers` run `31996260673`: `Success` on 2026-08-17
- Deployed Worker version: `d9568add-64c2-4327-9132-e3ffa792636e`

## What can be verified now

- Public app reachability on `https://mail.podbays.com`
- Public settings bootstrap via `GET /api/setting/websiteConfig`
- Admin login via `POST /api/login`
- Authenticated identity via `GET /api/my/loginUserInfo`
- Admin-only data surfaces such as `GET /api/setting/query` and `GET /api/user/list`
- Internal mail send path via `POST /api/email/send`
- The production Email Routing catch-all for `@podbays.com` exists on Cloudflare

## Fast path

Run the smoke helper from the repo root:

```bash
chmod +x scripts/podbays-acceptance-smoke.sh

BASE_URL=https://mail.podbays.com \
TEST_EMAIL=admin@podbays.com \
TEST_PASSWORD='your-password' \
EXPECT_ADMIN=1 \
./scripts/podbays-acceptance-smoke.sh
```

To include an external delivery check:

```bash
BASE_URL=https://mail.podbays.com \
TEST_EMAIL=admin@podbays.com \
TEST_PASSWORD='your-password' \
EXTERNAL_TO='your-external-mailbox@example.com' \
./scripts/podbays-acceptance-smoke.sh
```

Notes:

- `INTERNAL_TO` defaults to `TEST_EMAIL`, so the script self-sends an internal message and then checks both sent mail and inbox.
- `EXTERNAL_TO` is optional. When set, the script verifies that the send API succeeds and returns a `resendEmailId`, then you still need to confirm the message actually arrived in the external mailbox.
- The script cannot validate inbound delivery by itself because Cloudflare Email Routing happens outside the Worker HTTP surface.

## Manual acceptance checklist

1. Public reachability
   - Open `https://mail.podbays.com/`
   - Confirm the login page renders and the app shell loads without redirect loops or 5xx responses.
2. Admin login
   - Sign in as `admin@podbays.com`
   - Confirm `/inbox` loads and `GET /api/my/loginUserInfo` returns the expected mailbox and permissions.
3. Admin surfaces
   - Open `/settings`
   - Open `/all-users`
   - Open `/role`
   - Open `/system-setting`
   - Open `/all-mail`
   - Confirm admin-only views load without `401` or `403`.
   - Confirm `/all-mail` can see both receive and send records for recent test messages.
4. Internal outbound mail
   - Send a message from `admin@podbays.com` to another `@podbays.com` mailbox, or to itself for a quick loopback.
   - Confirm the message appears in the sender's sent folder.
   - Confirm the recipient inbox receives the message.
   - Confirm reply, forward, and attachment upload still work from the compose drawer.
5. External outbound mail
   - Send a message to an external mailbox.
   - Confirm the API returns success, the send record has a `resendEmailId`, and the external mailbox receives the message.
6. Inbound mail
   - Send a message from an external mailbox to a `@podbays.com` mailbox.
   - Confirm the message arrives in Cloud Mail and the parsed sender/subject/content look correct.
   - Also test a nonexistent `@podbays.com` recipient and confirm behavior matches the current `noRecipient` setting.
7. Post-deployment configuration
   - Confirm `GET /api/setting/websiteConfig` reports `r2Domain=mail.podbays.com/api/oss` and `@podbays.com` in `domainList`.

## Relevant routes

- Public routes: `/`, `/login`
- User routes: `/inbox`, `/message`, `/settings`, `/starred`
- Admin routes: `/all-users`, `/system-setting`, `/analysis`, `/role`, `/all-mail`, `/invite-code`

## Relevant APIs for troubleshooting

- `POST /api/login`
- `GET /api/my/loginUserInfo`
- `GET /api/setting/websiteConfig`
- `GET /api/setting/query`
- `GET /api/user/list`
- `POST /api/email/send`
- `GET /api/email/list`
