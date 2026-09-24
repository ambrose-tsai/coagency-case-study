# 06 · How It Was Built

I'm not a career software engineer. I'm a commercial leader who learned to direct AI coding agents with the same discipline I'd use to run a product team.

## Operating model

I made the product decisions and approved every release. AI coding agents did most of the implementation. Each agent had a defined role and limits:

| Role | Responsibility |
|---|---|
| Product owner (me) | Problem framing, scope, priorities, final go/no-go |
| Product manager agent | Specs, acceptance criteria, product contracts |
| Tech lead agent | Architecture, schema, server logic |
| UX agent | Design system, accessibility, responsive layouts |
| QA agent | Test plans, regression, release evidence |
| Compliance review agent | OAuth scopes, privacy, data-handling review |

Each role worked in its own git worktree with clear file ownership, so parallel work didn't collide. Nothing was merged until it passed a release gate.

## Sprint cadence and gates

- Two-week sprints, each with a written **exit gate**.
- Decision gates at key milestones to decide go, re-scope, or stop.
- A sprint can close as **Pass**, **Pass with approved exceptions**, or **Remediation required**. Each exception is written down with the reason it doesn't block the release.

## Quality bar for every sprint

- TypeScript, ESLint, and Vitest pass.
- Relevant pgTAP database tests pass.
- Migrations replay cleanly on a fresh database.
- New features include loading, empty, error, and success states; keyboard access; and a 375 px mobile layout.
- Accessibility regression checks with axe-core, aiming for WCAG 2.2 AA.

## Scale of the build

| Signal | Approximate size |
|---|---|
| Database migrations (versioned, replayable) | ~100 |
| Database test files (pgTAP) | 40+ |
| Commits across app + backend | 600+ |
| Sprints delivered | 35+ |

## What this taught me

- **Specs matter more with AI.** Vague requirements produce plausible code that's wrong. Clear contracts and acceptance criteria were what made the agents useful.
- **Gates beat heroics.** The security issues in [05](05-security-and-rbac.md) were caught by review and QA gates, not reported by users.
- **Run it like a business.** Scope, sequencing, risk, and saying no are the same skills whether you're managing a sales team or an AI engineering team.
