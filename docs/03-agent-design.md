# 03 · Agent Design

## The four MVP agents

| Agent | Input | Output | Human step |
|---|---|---|---|
| **AI Record** | Typed notes, meeting transcript, email thread | Structured visit log + suggested deal-field updates | Rep edits and confirms before the CRM write |
| **Follow-up** | Deal activity history, last-touch date, stage | Who to follow up with, why, and a suggested next step | Rep accepts or dismisses |
| **Deal Coach** | Deal record + activity log | BANT / MEDDIC score, risk level, missing information, next-best actions | Rep and manager use it as a coaching agenda |
| **Message Agent** | An accepted follow-up suggestion | An email draft grounded in the deal context | Rep reviews and sends |

Agents chain: an accepted **Follow-up** can become the input to a **Message Agent** draft. The link is stored, so the audit trail shows the whole chain.

## Three operating modes

| Mode | Behavior | MVP status |
|---|---|---|
| **Suggest** | Shows a recommendation. Nothing is written. | On |
| **Draft** | Creates a draft that a person confirms, edits, or dismisses | On (default) |
| **Autopilot** | Acts automatically when tenant rules are met | Designed, **kept off** |

Autopilot is off on purpose. For customer-facing actions, one wrong autonomous email costs more trust than a hundred correct drafts earn.

## Deal-scoring inputs (Deal Coach)

- **BANT:** Budget, Authority, Need, Timeline
- **MEDDIC:** Metrics, Economic buyer, Decision criteria, Decision process, Identify pain, Champion
- **Engagement signals:** how often the customer is contacted, and whether their reply times are slowing
- **Manager-defined indicators:** tenant-specific signals that are tuned over time

## Guardrails built into every agent

1. **Structured output only.** The model returns a typed tool call that is validated against a schema, never free text that gets written straight to the database.
2. **Permission check before action.** Each agent × mode × tenant has explicit capabilities (read, write CRM, draft email, send email, trigger alert).
3. **Duplicate protection.** A new draft is blocked if a pending draft of the same type already exists for the same deal or source.
4. **Audit record.** Each action records which data was read, what was proposed, who confirmed it, and when.
5. **Graceful failure.** If the model or a provider fails, the user sees an error. The CRM is never left half-written.
