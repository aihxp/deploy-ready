# Mirror Box cutover plan (dogfood for deploy-ready v1.0.7)

**Target:** [aihxp/mirror-box v1.0.0](https://github.com/aihxp/mirror-box)
**Target environment:** Fly.io shared-cpu-1x, region IAD
**Cadence:** per-milestone cutover (per ROADMAP M1)
**Mode:** first deploy (service has not yet been promoted to a live public URL; this document is the plan, not the retrospective)
**Version:** 1.0
**Last updated:** 2026-04-23
**Owner:** h (dogfood)

## What this is

deploy-ready's first-deploy pass against Mirror Box. The implementation exists at v1.0.0 with a Dockerfile and fly.toml; the actual `fly launch` has not yet been run in this session. This document is the cutover plan plus the first-deploy checklist, the rollback playbook, and the migration posture.

Purpose: close the deploy-ready half of the Mirror Box executional gap from the suite VALIDATION report.

## Pre-deploy state

- **Image buildable:** `docker build .` produces a working image from Mirror Box's Dockerfile (verified indirectly via `npm start` succeeding on Node 22; Dockerfile is conventional node:22-alpine with non-root user).
- **Secrets unknown:** no `HONEYCOMB_API_KEY` has been set as a Fly secret. First deploy will need this set before spans export, but the service runs fine without it (per PRD R-02's misconfigured-exporter acceptance criterion).
- **App name unclaimed:** `mirror-box` is the desired Fly app name; `fly launch` will create it in the operator's organization.
- **Region:** `iad` per ARCH decision.
- **VM size:** `shared-cpu-1x`, 256 MB memory, per fly.toml.
- **Health check:** `/healthz` every 30s with a 5s timeout and 10s grace period, per fly.toml.

## First-deploy checklist

deploy-ready Step 4 first-deploy mode:

### Pre-flight

- [ ] Fly CLI installed (`flyctl` v0.x+).
- [ ] `fly auth whoami` returns the expected account.
- [ ] No existing app named `mirror-box` in the target Fly org (blocks `fly launch` with a friendly error; rename in fly.toml if taken).
- [ ] Docker daemon running (Fly builds remotely by default, so this is optional but recommended for local smoke-test).
- [ ] `HONEYCOMB_API_KEY` value is in hand (Honeycomb free tier is sufficient; see https://ui.honeycomb.io/account/api-keys).

### Launch

```bash
cd ~/Projects/mirror-box
fly launch --copy-config --name mirror-box --region iad --no-deploy
```

`--no-deploy` is intentional: we want to set secrets before the first deploy so the first span-emitting request traces correctly.

### Secrets

```bash
fly secrets set HONEYCOMB_API_KEY=hcaik_XXXXXXXXXXXXXXXXXXXXXXXX
# Optional: override the dataset name (defaults to mirror-box).
# fly secrets set HONEYCOMB_DATASET=mirror-box-prod
```

### First deploy

```bash
fly deploy
```

Expected output: build succeeds, image pushes to Fly's registry, one machine starts in IAD, health check passes, service becomes reachable at `https://mirror-box.fly.dev`.

### Post-deploy verification

deploy-ready Step 7 checklist adapted for Mirror Box:

```bash
# 1. Health endpoint reachable.
curl -fsS https://mirror-box.fly.dev/healthz
# Expect: {"status":"ok"}

# 2. Echo endpoint happy path.
curl -fsS -X POST https://mirror-box.fly.dev/echo \
  -H 'content-type: application/json' \
  -d '{"deploy":"smoke-test","at":"first-deploy"}'
# Expect: echo with trace_id populated.

# 3. Echo endpoint error path.
curl -sS -X POST https://mirror-box.fly.dev/echo \
  -H 'content-type: application/json' \
  -d 'not json' \
  -w '\nstatus: %{http_code}\n'
# Expect: {"error":"body must be JSON"} and status: 400

# 4. Service metadata.
curl -fsS https://mirror-box.fly.dev/
# Expect: {"service":"mirror-box",...}

# 5. Tracing verified in Honeycomb.
# Within 10 seconds of the above calls, spans should appear in the
# Honeycomb `mirror-box` dataset (or whatever HONEYCOMB_DATASET was set to).
# Check: https://ui.honeycomb.io/<team>/datasets/mirror-box
# Expect: one span per request, with http.route, http.status_code,
# user.ip (hashed), request.size_bytes, response.size_bytes attributes.

# 6. Logs clean.
fly logs
# Expect: boot messages, one incoming-request + request-completed pair
# per smoke-test request. No error lines. No raw IPs (only ip_hash).
```

All six must pass for first deploy to be declared done.

### Success criteria (first-deploy done)

- [ ] `https://mirror-box.fly.dev/healthz` returns 200 with `{"status":"ok"}`.
- [ ] `/echo` happy and error paths return correct shapes.
- [ ] A span from a live request is visible in Honeycomb within 30 seconds.
- [ ] Fly shows one healthy machine in IAD.
- [ ] `fly logs` shows no error or warning lines post-boot.

## Rollback playbook

### Rollback window

Fly retains the last 3 deployments by default. The fly.toml doesn't change this. Rollback is to the previous release, not to a stored backup.

### Rollback trigger conditions

- **Health check failures** for more than 3 consecutive intervals (90 seconds) post-deploy.
- **Span export broken** post-deploy (Honeycomb shows no spans from the new release 5 minutes after promotion).
- **Response-time regression** above PRD's p95 <200ms target, sustained for 10 minutes.
- **Security-relevant** anything (unexpected outbound connections, secret leakage in logs, auth-bypass if rate limiting ships).

### Rollback procedure

```bash
# List recent releases.
fly releases

# Roll back to the previous version.
fly releases rollback <release-id>

# Verify.
curl -fsS https://mirror-box.fly.dev/healthz
fly logs | head -20
```

Rollback typically completes in under 60 seconds (image already in Fly's registry; just a machine restart).

### Post-rollback

- Confirm health check passes on the rolled-back version.
- Confirm spans resume in Honeycomb (if the rollback is past the span-broken release).
- Open an issue in aihxp/mirror-box documenting what triggered the rollback.
- Do not push a `v1.0.x` tag for the broken release; delete the broken tag if already pushed.

## Migration posture

**None at v1.0.0.** The service is stateless by design (ARCH ADR-002). No database, no schema, no file store, no event log. Expand/contract patterns don't apply; there is nothing to expand or contract.

If OQ-01 (rate limiting) ever lands at v1.1 with a Redis store:

1. **Deploy 1 (expand):** add Redis connection, wire rate-limit code behind a feature flag, deploy with flag off.
2. **Deploy 2 (enable):** flip the flag, observe rate-limit hits in spans.
3. **Deploy 3 (contract):** if the flag proves stable, remove the flag. If not, roll back deploy 2.

That's the smallest-version migration discipline and it isn't needed now. Noted for completeness.

## Flag-rollout calendar

Not applicable at v1.0.0. The service ships with zero feature flags. If v1.1 introduces rate limiting with a flag, the calendar would be:

- **Day 0:** deploy with flag off.
- **Day 1-3:** soak.
- **Day 4:** flip flag in staging (if staging exists; currently there is no staging env for Mirror Box).
- **Day 7:** flip flag in production.
- **Day 10:** remove flag if stable.

Not applicable for first-deploy.

## Environment-parity note

Mirror Box v1.0.0 has no staging environment. First-deploy promotes straight to production (`mirror-box.fly.dev`). This is acceptable for a weekend-project-tier service but means:

- There is no place to test deploy-time config changes before production.
- There is no place to exercise rollback without affecting real users.

If Mirror Box graduates to a canonical teaching target used in scheduled workshops, add a staging app (`mirror-box-staging.fly.dev`) with the same fly.toml and a separate Honeycomb dataset. deploy-ready's Step 3 would then include a promote-to-staging step before promote-to-production.

Noted for future; not in scope at v1.0.0.

## Known first-deploy risks

| Risk | Mitigation |
|---|---|
| `mirror-box` app name taken in operator's Fly org | Rename in fly.toml before `fly launch`. |
| Honeycomb free-tier key not in hand | Service boots and runs regardless; spans drop. Add key post-deploy via `fly secrets set` and `fly deploy` without code change. |
| Fly IAD region temporarily unavailable | Use `--region ord` or `--region ewr` as fallback. Latency changes slightly; no functional impact. |
| Docker build layer-caching bug | `fly deploy --build-only` to confirm image build works in isolation. |
| Port mismatch between Dockerfile and fly.toml | Both should be 8080. Verify `internal_port = 8080` in fly.toml and `EXPOSE 8080` in Dockerfile. (Confirmed at v1.0.0 ship.) |
| Honeycomb host changes (e.g., `api.honeycomb.io` for US-hosted accounts; different for EU) | `src/otel.js` hardcodes `https://api.honeycomb.io:443`. EU Honeycomb users must set `OTEL_EXPORTER_OTLP_ENDPOINT=https://api.eu1.honeycomb.io:443` explicitly. |

## Verification (for when the first deploy actually runs)

When a maintainer runs the plan above end-to-end, this document's dogfood status becomes "verified." Until then, this document is deploy-ready's cutover plan for Mirror Box, not proof that the plan ran to completion.

To mark dogfood verified:
- Run the first-deploy checklist end-to-end.
- Confirm all six post-deploy verification steps pass.
- Append a "Verified 2026-MM-DD at https://mirror-box.fly.dev" line to the top of this document.

## Handoff (back to the suite)

This dogfood doesn't trigger downstream re-runs. The cutover plan references observe-ready for Honeycomb verification but does not invoke observe-ready; that skill has its own dogfood at `aihxp/observe-ready/dogfood/OBSERVABILITY.md`.

Ready-suite version table and SUITE.md are unchanged by this commit.
