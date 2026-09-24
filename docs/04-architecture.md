# 04 · Architecture

## Stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | Next.js App Router, React, TypeScript, Tailwind | Fast iteration; server components keep secrets on the server |
| Backend | Next.js server actions and route handlers | One codebase, one deploy |
| Database | Supabase PostgreSQL | Relational CRM data with Row Level Security for multi-tenancy |
| Auth | Supabase Auth | JWT sessions tied to database policies |
| AI | Anthropic Claude API | Structured tool use for agent outputs |
| Integrations | Gmail API, Google Calendar API | Where sales work already happens |
| i18n | next-intl | Built for multiple languages from day one |
| Hosting | Vercel + Supabase | No infrastructure to run as a one-person team |
| Quality | Vitest, pgTAP, ESLint, axe-core | App tests, database tests, and accessibility checks |

## System diagram

```mermaid
flowchart TB
    subgraph Client
        UI[Rep / Manager / Admin UI]
    end

    subgraph Server["Next.js server"]
        SA[Server actions]
        RH[Route handlers<br/>OAuth callbacks · sync · export]
        ORC[Agent orchestrator]
        TPL[Tool permission layer]
    end

    subgraph Supabase
        AUTH[Auth / JWT]
        PG[(Postgres<br/>tenant-scoped tables + RLS)]
        RPC[Security-definer RPCs<br/>atomic confirm / convert / merge]
        AUD[(Audit log)]
    end

    EXT1[Claude API]
    EXT2[Gmail API]
    EXT3[Calendar API]

    UI --> SA & RH
    SA --> ORC --> EXT1
    EXT1 -->|typed tool call| TPL
    TPL --> RPC --> PG
    TPL --> AUD
    RH <--> EXT2 & EXT3
    AUTH --> PG
```

## Agent execution path

```
Trigger (user action · scheduled job · system event)
  → Orchestrator selects agent + loads tenant-scoped CRM context
  → Claude API called with tool definitions
  → Model returns a structured tool call (schema-validated)
  → Tool permission layer checks agent × mode × tenant × role
  → Draft stored as a pending agent action
  → Human confirms / edits / dismisses
  → Atomic RPC writes CRM changes
  → Audit record written
```

## Data model (simplified)

```mermaid
erDiagram
    TENANT ||--o{ PROFILE : has
    TENANT ||--o{ ACCOUNT : owns
    ACCOUNT ||--o{ CONTACT : has
    ACCOUNT ||--o{ DEAL : has
    DEAL ||--o{ ACTIVITY : logs
    DEAL ||--o{ AGENT_ACTION : receives
    AGENT_ACTION ||--o| AGENT_ACTION : "source (chained drafts)"
    PROFILE ||--o{ DEAL : "owns (rep)"
    TENANT ||--o{ AUDIT_LOG : records
```

Every business table carries a `tenant_id`, and database policies (not only application code) enforce tenant isolation.

## Key design decisions

| Decision | Alternative considered | Reasoning |
|---|---|---|
| Enforce tenancy in the database (RLS) | Filter in application code | A missed `WHERE` clause shouldn't be able to leak another customer's data |
| Critical writes as atomic RPCs | Several client calls | "Confirm draft + update deal + write audit" must all succeed or all fail |
| Draft-first agents | Direct autonomous writes | Trust and correctness matter more than saving one click |
| Start with the minimum Google access | Request full mailbox access up front | Easier OAuth review; less risk for users |
| Numbered, replayable migrations with CI | Manual schema edits | Every environment can be rebuilt and tested from zero |
