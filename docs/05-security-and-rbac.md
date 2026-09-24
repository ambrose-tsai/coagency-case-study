# 05 · Security & RBAC

A CRM holds a company's customer relationships. Security was part of the product from the first sprint, not added later.

## Access model

| Layer | Control |
|---|---|
| **Tenant** | Every row is scoped by `tenant_id`; Row Level Security blocks reading or writing across tenants |
| **Role** | Rep / Manager / Admin, checked in database helper functions *and* in server actions |
| **Ownership** | Reps work their own deals; managers see their team; reassignment is an explicit, audited action |
| **Agent** | Tool permission layer: each agent × mode has explicit capabilities |
| **Platform** | Internal platform admin sees aggregates only, never customer content by default |

### Role matrix (simplified)

| Capability | Rep | Manager | Admin |
|---|:-:|:-:|:-:|
| View own deals & activities | ✅ | ✅ | ✅ |
| View team pipeline | — | ✅ | ✅ |
| Confirm AI drafts on own deals | ✅ | ✅ | ✅ |
| Reassign deal owner | — | ✅ | ✅ |
| Bulk import / export | — | — | ✅ (audited) |
| Manage users & roles | — | — | ✅ |
| Configure pipeline stages | — | — | ✅ |

---

## Hardening cases

These three cases came up during development. They show how I handle security issues.

### Case 1: An export that could skip its audit record

- **Found by:** manual QA during a sprint release gate.
- **Issue:** A CSV export could finish successfully even when writing its audit record failed. Customer data could leave the system with no record that it did.
- **Root cause:** A missing database grant for the audit write path, plus export logic that treated the audit write as optional.
- **Fix:** Added the missing grant in a new migration. Changed every export route to **fail closed**: no audit record, no file.
- **Lesson:** Audit is part of the transaction, not a side effect. The release gate was marked *remediation required* until the fix was verified.

### Case 2: A core table that never had Row Level Security

- **Found by:** security review of the schema.
- **Issue:** The tenants table had been created without Row Level Security. Its rows held workspace settings, including an integration webhook URL that should be secret, and a `plan` field that users must not be able to change.
- **Why row policies weren't enough:** Any policy that let users read their own tenant row would also expose the secret field, and allowing updates would let users change their plan directly.
- **Fix:** A hotfix migration turned on RLS and **revoked all public API access** to the table, which now fails closed. Every tenant read and write now goes through server-side code that checks authorization first.
- **Lesson:** Every new query or mutation must ship with a tenant-isolation test *and* a role-authorization test. This became a release rule.

### Case 3: Support access without standing access

- **Tension:** Support needs to troubleshoot customer issues, but a permanent super-admin that can read everything is too risky.
- **Decision (architecture decision record):**
  - Platform admin stays **aggregate-only** by default.
  - Access to customer content requires a grant that is **tenant-bound, purpose-bound, scope-bound, time-bound, and audited**.
  - Read, impersonate, and write are separate scopes.
  - Emergency "break-glass" access is short-lived, notifies the owner, and requires a review afterward.
  - Platform telemetry excludes contact details, email bodies, prompts, and AI payloads.
- **Lesson:** Treat support access as a security product with its own design, not something that comes free with the `admin` role.

---

## Release rules that came out of this

- Every change that touches data ships with tenant-isolation and role tests.
- Before production, each migration is replayed on a clean database and must pass the full database test suite.
- A production release requires a backup reference, a dry run, approval, and a check after deploy.
- AI actions that reach outside the system stay human-in-the-loop.
