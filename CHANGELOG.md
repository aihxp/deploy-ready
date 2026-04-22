# Changelog

## v0.1.0 (2026-04-22)

Initial pre-release of deploy-ready, the shipping-tier skill that owns the pre-prod-to-prod handoff in the [ready-suite](SUITE.md). Ships with the full SKILL.md contract, ten reference files, the research report that backs the guardrails, and full interop frontmatter. Pre-stable by convention: 0.x iterates in public until v1.0.0 is earned against real deploys.

### Added

- **SKILL.md** with the ready-suite interop standard: eleven frontmatter fields populated, six required sections present. Eleven-step workflow, four completion tiers, a have-nots list, a session state template, and explicit consume/produce contracts with sibling skills.
- **Three named failure modes** the skill owns: **paper canary**, **expand-only migration trap**, and **first-deploy blindness**. See the research report for the naming-lane survey that justified each.
- **Ten reference files** under `references/`:
  - `deploy-research.md`. Step 0 mode detection (A first deploy, B subsequent, C incident, D pipeline construction, E migration-dominated), destructive-command alert, expand-only migration trap detection procedure.
  - `preflight-and-gating.md`. Expanded 10 pre-flight questions, Mode B subsequent-deploy checklist, the four gate types with pipeline-enforcement patterns.
  - `deployment-topologies.md`. Seven topologies, per-topology first-deploy hazards and rollback characteristics, mixed-topology worked examples.
  - `pipeline-patterns.md`. The 8 pipeline gates with good and bad GitHub Actions YAML, same-artifact invariant, supply-chain pitfalls including the `pull_request_target` footgun.
  - `environment-parity.md`. The four parity gaps (time, personnel, tooling, fidelity), per-rung parity table, pre-prod parity gap as a named concept.
  - `first-deploy-checklist.md`. Eleven cold-start gates, per-platform gotchas (Vercel, Netlify, Fly.io, Cloud Run, Lambda, Kubernetes), dry-run rollback procedure, printable checklist.
  - `zero-downtime-migrations.md`. Expand/migrate/cutover/contract calendar, 10-pattern guardrail catalog with unsafe and safe SQL per pattern, worked 3-deploy column rename, expand-only migration trap deep dive.
  - `rollback-playbook.md`. Code-vs-data rollback asymmetry, compensating-forward patterns, Knight Capital flag-lineage discipline, destructive-command gate grounded in Replit and DataTalks.Club incidents, incident log template.
  - `progressive-delivery.md`. Five rollout strategies, paper-canary rule with four required fields, blast-radius rule citing CrowdStrike / Cloudflare 2019 / Facebook 2021, readiness probe discipline.
  - `secrets-injection.md`. Per-topology injection patterns, Docker layer leak class, `pull_request_target` surface, build-time vs runtime split, artifact-level audit commands.
- **Research report** (`references/RESEARCH-2026-04.md`, ~5000 words). Named incidents (Knight Capital, GitLab 2017, AWS S3 2017, Cloudflare 2019, Facebook 2021, CrowdStrike 2024, Replit 2025, DataTalks.Club 2025, timescale/pgai 2025, Docker Hub leaks), tool gap analysis (GitHub Actions, GitOps, progressive-delivery vendors, platform-native CD), zero-downtime migration literature survey, naming-lane analysis, DORA 2024 / Stack Overflow 2024-2025 / GitGuardian 2026 quantitative framing.
- **SUITE.md** at repo root, listing deploy-ready alongside the three live siblings (production-ready 2.5.1, repo-ready, stack-ready 1.1.0) and the five planned skills.
- **README.md** with install paths for Claude Code, Codex, Cursor, and Windsurf; the "what this skill prevents" incident-to-enforcement mapping; and the reference-library index.

### Why 0.1.0, not 1.0.0

Pre-stable iteration is the suite convention (stack-ready's pattern). The skill ships with a complete contract but has not yet been exercised against a real production deploy end-to-end. v1.0.0 is earned against a real cut-over, not declared at initial release. Expect minor bumps during 0.x as rough edges surface on real use.

### Known gaps (to be closed before v1.0.0)

- No sample `.deploy-ready/` artifact set from a real deploy. STATE.md and PLAN.md templates are prose-only until the first dogfood run.
- No Kubernetes-native reference guidance yet (the topology is covered but deep K8s operator and service mesh patterns are light).
- Progressive-delivery guidance biases toward Argo Rollouts and LaunchDarkly; Flagger and Unleash get named but not deep examples.

### Compatibility

- Claude Code (primary)
- Codex
- Cursor
- Windsurf (manual SKILL.md upload)
- Any agent with skill loading
