# The Ready Suite

A composable set of AI skills covering the full arc from idea to launch. Each skill has a tight scope, distinct triggers, and explicit handoffs to its siblings. Install what you need. No skill duplicates another's work.

This file is byte-identical across every skill in the suite. If you installed one skill, you can see the whole map here.

## The ten skills, three tiers

```
PLANNING TIER                     BUILDING TIER                  SHIPPING TIER
------------------------          -----------------              ------------------
prd-ready           ->            repo-ready          ->         deploy-ready
architecture-ready  ->            production-ready    ->         observe-ready
roadmap-ready       ->                                ->         launch-ready
stack-ready         ->                                ->         harden-ready
```

## What each skill owns

| Skill | Tier | One-line purpose | Primary trigger words |
|---|---|---|---|
| **prd-ready** | Planning | Define what we're building and for whom. | "write a PRD," "product spec," "requirements doc" |
| **architecture-ready** | Planning | Design how the big pieces fit together. | "design the architecture," "system diagram," "integration shape" |
| **roadmap-ready** | Planning | Sequence work over time. | "build a roadmap," "milestone plan," "quarterly plan" |
| **stack-ready** | Planning | Pick the right tools for the job. | "what stack should I use," "pick an ORM," "framework comparison" |
| **repo-ready** | Building | Set up the repo with production-grade hygiene. | "set up the repo," "add CI," "CONTRIBUTING.md," "release automation" |
| **production-ready** | Building | Build the app to production grade. | "dashboard," "admin panel," "internal tool," "back office," "control panel" |
| **deploy-ready** | Shipping | Ship the app to real environments. | "deploy this," "CI/CD pipeline," "zero-downtime migration," "rollback" |
| **observe-ready** | Shipping | Keep the app healthy once it's live. | "add monitoring," "Datadog," "alerts when X," "SLO," "runbook" |
| **launch-ready** | Shipping | Tell the world the product exists. | "launch my product," "build a landing page," "SEO," "Product Hunt" |
| **harden-ready** | Shipping | Survive adversarial attention; prove it to an auditor. | "adversarial review," "pen-test prep," "OWASP Top 10 walkthrough," "SOC 2 / HIPAA / PCI-DSS gap check," "responsible disclosure program," "post-incident hardening" |

## Dependency flow

```
PRD -> Architecture -> Roadmap -> Stack -> (Repo || Production) -> Deploy -> Observe -> Launch
                                                                                     -> Harden
```

harden-ready runs at the shipping tier alongside deploy-ready, observe-ready, and launch-ready. It consumes the architecture trust boundaries, the production-ready threat model and RBAC matrix, the deploy-ready environment list, and the observe-ready alert catalog; it produces a findings-and-fix-plan artifact consumed by humans (security engineers, auditors, founders pre-launch), not by another suite skill.

Skills consume artifacts from upstream siblings and produce artifacts for downstream siblings. The handshake is documented per skill under "Consumes from upstream" and "Produces for downstream." Each skill degrades gracefully when upstream artifacts are missing: it falls back to its own defaults.

## Canonical dogfood target: Mirror Box

The ready-suite ships a single reference implementation that every skill has a dogfood artifact against: [aihxp/mirror-box](https://github.com/aihxp/mirror-box). Mirror Box is a tiny traced HTTP echo service (Fastify + OTel SDK + OTLP/gRPC to Honeycomb, deployable to Fly.io free tier) intentionally small enough that the full nine-skill arc fits on a single page at each step.

- Planning-tier artifacts: [PRD.md](https://github.com/aihxp/prd-ready/blob/main/dogfood/PRD.md), [ARCH.md](https://github.com/aihxp/architecture-ready/blob/main/dogfood/ARCH.md), [ROADMAP.md](https://github.com/aihxp/roadmap-ready/blob/main/dogfood/ROADMAP.md), [DECISION.md](https://github.com/aihxp/stack-ready/blob/main/dogfood/DECISION.md).
- Building-tier artifacts: [HYGIENE.md](https://github.com/aihxp/repo-ready/blob/main/dogfood/HYGIENE.md), [SLICES.md](https://github.com/aihxp/production-ready/blob/main/dogfood/SLICES.md).
- Shipping-tier artifacts: [CUTOVER.md](https://github.com/aihxp/deploy-ready/blob/main/dogfood/CUTOVER.md), [OBSERVABILITY.md](https://github.com/aihxp/observe-ready/blob/main/dogfood/OBSERVABILITY.md), [N-A.md](https://github.com/aihxp/launch-ready/blob/main/dogfood/N-A.md) (launch correctly N/A), [FINDINGS.md](https://github.com/aihxp/harden-ready/blob/main/dogfood/FINDINGS.md).

See [VALIDATION.md](https://github.com/aihxp/production-ready/blob/main/VALIDATION.md) for the suite-level end-to-end validation report, including the gap that these artifacts closed.

## Install locations

Every skill installs the same way on every platform. On a machine where the dev copy lives at `~/Projects/<skill-name>/`, symlinking is recommended so updates in the dev copy propagate instantly.

| Platform | Install path |
|---|---|
| Claude Code | `~/.claude/skills/<skill-name>/` |
| Codex | `~/.codex/skills/<skill-name>/` |
| Cursor | `~/.cursor/skills/<skill-name>/` |
| Windsurf | Project rules or system prompt |
| Other agents | Upload `SKILL.md` and relevant reference files to the project context |

## Composition principles

1. **Tight scope beats combined scope.** Each skill does one thing. Splitting is always the first move.
2. **The harness is the router.** The skill does not route to other skills; it tells the user (or the harness) what to invoke next and why.
3. **Artifacts are the contract.** Skills do not call each other. They write files to `.<skill-name>/` paths that downstream skills read.
4. **Version tracking is mandatory.** Every skill carries `version`, `updated`, `suite`, `tier`, `upstream`, and `downstream` in frontmatter.
5. **No skill duplicates another.** If work overlaps, one skill owns the canonical content and siblings reference it.
6. **Graceful degradation.** Every skill works standalone. Upstream absence is handled with sensible defaults, not errors.
7. **Byte-identical SUITE.md across siblings.** When any sibling releases, every sibling's SUITE.md row bumps. This keeps the known-good-versions table authoritative from any entry point.

## Why a ready suite at all

AI-generated apps fail in predictable ways: hollow buttons, placeholder READMEs, missing CI, unshipped half-migrations, unmonitored production, silent launches, and security surfaces that pass SAST/SCA and fail adversarial review. One giant skill covering everything becomes unfocused and bloats past the point of usefulness. Separate tight skills composing through explicit handoffs stays sharp.

## Known-good versions

| Skill | Current version | Repo |
|---|---|---|
| **production-ready** | 2.5.10 | https://github.com/aihxp/production-ready |
| **repo-ready** | 1.6.6 | https://github.com/aihxp/repo-ready |
| **stack-ready** | 1.1.9 | https://github.com/aihxp/stack-ready |
| **deploy-ready** | 1.0.8 | https://github.com/aihxp/deploy-ready |
| **observe-ready** | 1.0.7 | https://github.com/aihxp/observe-ready |
| **launch-ready** | 1.0.5 | https://github.com/aihxp/launch-ready |
| **prd-ready** | 1.0.4 | https://github.com/aihxp/prd-ready |
| **architecture-ready** | 1.0.3 | https://github.com/aihxp/architecture-ready |
| **roadmap-ready** | 1.0.2 | https://github.com/aihxp/roadmap-ready |
| **harden-ready** | 1.0.1 | https://github.com/aihxp/harden-ready |

The suite is additive. A skill not yet released does not block any other skill from functioning. When a skill reaches v1.0.0, update this table and ship the change across all installed siblings. With harden-ready v1.0.0 the core suite is complete: ten skills across planning (four), building (two), and shipping (four). Mirror Box ([aihxp/mirror-box](https://github.com/aihxp/mirror-box)) is the canonical dogfood target every skill has an artifact against.
