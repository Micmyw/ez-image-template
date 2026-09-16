# AI media SaaS foundation

Project website: [EzPic - AI Image Editor](https://ezimageai.com/).

A reusable Supastarter-based subscription foundation for private AI image and video generation. Supastarter remains the product base; only reviewed request/status mapping and upload UX patterns were adapted from ShipAny. It is production-oriented, but a checkout is not live-certified until its own cloud accounts, credentials, quotas, Webhooks, alerts, and staging load have been verified.

## Foundation capabilities

- **Stable product catalog and Provider abstraction:** clients submit public product keys and validated parameters; server-only routes moderate prompts and map approved requests to Replicate, Fal, Kie, or Gemini adapters. Provider names, model IDs, credentials, raw errors, and arbitrary result URLs stay off the public contract.
- **Durable background work:** PostgreSQL is the only business source of truth. Job creation, input binding, credit reservation, and the initial Outbox event commit atomically. Trigger.dev is the first-release task engine, while polling, Webhooks, and reconciliation recover work from persisted state.
- **Auditable credits and subscriptions:** immutable ledger entries, expiring credit lots, atomic reservation/charge/release, zero charge when no usable output exists, monthly grants for monthly and annual Stripe plans, cancellation, refunds, and refund debt are modeled explicitly.
- **Private media pipeline:** direct single-part or multipart upload, aggregate per-owner storage and active-session quotas, exact part constraints, signed reads, streamed Provider transfer, moderation, quarantine, soft delete, and durable object cleanup avoid buffering large videos through Vercel.
- **Reusable product surfaces:** authenticated creator, history, job detail and asset library, plus an anonymous marketing draft handoff that transfers only after sign-in and one-time claim.
- **Operational controls:** generation, moderation, billing, Provider/model and queue gates; redacted structured logs; Sentry hooks; protected diagnostics, replay, stage retry and uncertain-submission resolution; CI, Provider smoke budgets, load profiles, and invariant verification.

## Local development

Requirements: Node.js 22+, pnpm 11, Docker, and PostgreSQL. Copy `.env.local.example` to `.env.local`, keep mock/test adapters enabled locally, then run:

```bash
docker compose up -d
pnpm install --frozen-lockfile
pnpm db:migrate:deploy
pnpm dev
```

The SaaS app is at `http://localhost:3000`; marketing is at `http://localhost:3001`. Media objects use the private `MEDIA_BUCKET_NAME` bucket (`media-private` locally). Do not use the legacy `S3_BUCKET` variable.

## Verification

```bash
pnpm lint --deny-warnings
pnpm format:check
pnpm type-check
pnpm test:unit:contracts
pnpm test:integration
pnpm e2e:media:ci
pnpm verify:invariants
```

PostgreSQL integration commands require an explicit loopback `TEST_DATABASE_URL` whose database name contains `test` or `testing`; they never fall back to `DATABASE_URL`. Mock E2E fails when its database, test user, Chromium, or test adapters are missing.

The current local production-build browser suite covers 11 scenarios with no skips: nine authenticated SaaS generation/storage/credit workflows and two marketing draft-handoff workflows. It uses deterministic test Provider and moderation adapters, an isolated PostgreSQL database, and private local MinIO; this is evidence for the application workflow, not for external Provider or cloud connectivity. The production dependency audit currently has no high or critical advisories; low and moderate advisories remain subject to normal dependency maintenance.

Load profiles are `smoke`, `steady` (200 jobs/min for 30 minutes), `peak` (400 jobs/min for five minutes), and `active-1000`. Remote targets require both `ALLOW_REMOTE_LOAD_TARGET=true` and an exact `LOAD_TARGET_CONFIRMATION` origin.

```bash
pnpm load:media
pnpm verify:invariants
```

The guarded local route smoke is implemented and has passed against the isolated test database; its post-run invariant checks remained at zero violations. This workstation does not have k6 installed, so no real k6 result is claimed. The five-minute peak, 30-minute steady, and active-1000 profiles still require a dedicated staging-equivalent deployment and have not been certified from this checkout.

Real Provider smoke tests are excluded from PR CI. The protected `Provider smoke` workflow validates its certified route allowlist, maximum invocation count, and maximum expected cost before a Provider call; no live result is implied by local mocks or a dry run.

Before calling a deployment live-ready, record successful staging checks for Trigger.dev task deployment, Stripe test-mode Webhook delivery, every enabled Provider route, Sightengine, private S3/R2 multipart and streamed transfer, Sentry ingestion/alerts, and the documented staging load profiles.

## Operations

Use [docs/operations/ai-media-runbook.md](docs/operations/ai-media-runbook.md) for production accounts, environment validation, migration safety, Trigger deployment, storage IAM/CORS, Stripe Webhooks, moderation, Sentry, model certification, load verification, replay/reconciliation, refunds, backup/restore, rollback, secret rotation, and incident response.
