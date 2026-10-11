# Database design

Owner: Issue #3 — Design database schema. Prerequisites: #1–#2.
Source: guide §§5, 9, 12–16. This is a logical design, not an executable migration.

## 1. How to read this design

An entity is a kind of stored record. A foreign key connects records; a constraint rejects invalid combinations. The application checks business rules, and database constraints protect those rules against mistakes and concurrent requests.

Organization is the data boundary. A user is global to authentication, while a membership assigns that user a role inside one company. Every business record must carry an organization ID or an equally enforced scoped relationship; the reference preference is explicit organization IDs for business data.

## 2. Records and ownership

| Record | Important fields | Relationship / purpose | Implement |
|---|---|---|---|
| User, session, account, verification | Auth-library generated fields | Authentication lifecycle; do not hand-roll competing tables | #7 |
| Organization | id, name | Company workspace | #8 |
| Membership | id, organizationId, userId, role | User's company-specific permission | #8–#9 |
| Invitation | organizationId, email, role, expiry | Controlled team onboarding | #8 |
| Customer | id, organizationId, name, email, archivedAt | Customer identity and retained history | #11 |
| Booking | id, organizationId, customerId, reference, dates, status | Seeded/manual read-only context for AI | #11–#12 |
| Ticket | id, organizationId, customerId, bookingId?, assigneeId?, status, version | Support request; version protects stale changes | #13–#14 |
| TicketMessage | id, organizationId, ticketId, authorType, body, createdAt | Distinguish internal note and demo reply | #13 |
| Task | id, organizationId, ticketId, assigneeId?, title, status, dueAt? | Follow-up work | #15 |
| AuditEvent | organizationId, actorId?, eventType, targetId, timestamp | Trace important business writes | #16; earlier mutations instrumented |
| Document | id, organizationId, storageKey, version, status, checksum | Private source file and searchable version | #17–#18 |
| DocumentChunk | id, organizationId, documentId, version, position, text, locator, embedding | Searchable passage with provenance | #18–#19 |
| AgentRun | id, organizationId, ticketId, initiatedBy, status, model, promptVersion, usage | One bounded investigation | #24 |
| AgentStep | id, organizationId, runId, toolName, status, redacted summary, duration | Observable execution record | #24 |
| ActionProposal | id, organizationId, runId, type, payload, payloadVersion, status | Exact action awaiting review | #24–#25 |
| ApprovalDecision | id, organizationId, proposalId, reviewerId, decision, payloadVersion, decidedAt | Review history, retained when drafts change | #25 |
| ActionExecution | id, organizationId, proposalId, idempotencyKey, status, resultId?, errorCode? | Duplicate protection and outcome | #25 |

The generated auth-library schema controls exact identity table names and relationships. Adapt this logical mapping rather than creating a second identity system. An outbox/pending-job record is also needed for durable queue handoff in #18; evaluation fixtures initially live in files under `evals/`.

## 3. Relationship examples

One organization has many customers. A customer has many bookings and tickets. A ticket may refer to one booking and has many messages, tasks, and investigations. A run has steps and proposals. A proposal has review decisions and an execution outcome. A document has versioned chunks.

For Rahul's hotel-date request, the ticket identifies Rahul by ID, links his booking, and stores the requested change as a message. A run retrieves that booking and a policy chunk. Its proposal points back to the run, so an employee can trace the resulting task to the ticket and evidence.

## 4. Required integrity rules

1. Unique membership per `(organizationId, userId)`; a user can have different roles in different organizations.
2. Booking references are unique within an organization, not necessarily globally.
3. Customer, booking and ticket organization IDs must agree. A ticket's booking must belong to its selected customer.
4. Assignees must be active members of the same organization. A foreign key alone does not establish active membership; recheck in the service.
5. Use scoped composite keys/foreign keys where feasible. Proposed pattern: Customer unique `(organizationId, id)`; Booking references that pair. A booking composite key including customerId can support a ticket link that enforces customer consistency.
6. Ticket messages, tasks, runs, steps, proposals, decisions and executions must remain within the parent organization's boundary.
7. Add a unique chunk key for `(organizationId, documentId, version, position)` or an equivalent deterministic identifier. Retries must upsert/replace a version, not append duplicate chunks.
8. Add unique execution idempotency keys and ensure one business effect for one approved proposal version, including simultaneous approval requests.
9. Keep mutation versions on records used for approval preconditions. Check expected versions in the same transaction that applies a change.
10. Archive customer/document records when history needs preservation. Archiving a document must exclude its chunks immediately from retrieval.

Rules 5 and 8 are implementation patterns to review against Prisma and PostgreSQL capabilities in the owning issue. Enforce the behavior even if the final constraint representation differs.

## 5. States and concurrency

Document states from the guide: Pending → Processing → Ready, with Failed and Archived alternatives. A new document version must not expose partially processed chunks.

Proposal states from the guide: Pending → Approved → Executing → Executed; rejection yields Rejected, changed context yields Expired, and execution error yields Failed. Only a safe, still-authorized retry may move Failed back to Executing. Editing creates a new payload version requiring fresh review.

**Proposed implementation detail, decide in #14:** tickets use Open, In progress, Waiting for customer, Resolved, and Closed. Write an explicit allowed-transition and role table before code. Resolution through AI still requires owner/manager approval regardless of the manual transition policy.

**Proposed implementation detail, decide in #15:** tasks use Open, In progress, Completed, and Cancelled. Define reopening and due-date behavior explicitly. Run/step state enums are finalized in #24; they must represent in-progress, completed, partial/limited, and failed outcomes honestly.

Approval execution transaction: lock/claim the proposal, check payload/context versions and current authority, ensure no previous successful effect, perform the business write, save execution result and audit event, commit. A failure must not leave a success label without the effect. No external message delivery is part of version 1.

## 6. Storage conventions and indexes

Store timestamps consistently and display in the employee's timezone. Booking calendar dates are distinct from timestamps. Store money as integer minor units with a currency, never a binary floating-point amount. Nullable booking/assignee fields must be handled as missing context rather than guessed.

Index common scoped queries: tickets by organization/status/creation time; tasks by organization/assignee/status; customers by organization/search fields; runs by organization/ticket/time; audit by organization/time. Choose vector indexes from measured dataset/query behavior, not as a substitute for organization filtering. Record embedding model and dimensions alongside version metadata.

## 7. Migration and seed sequence

| Issue | Database work |
|---|---|
| #3 | Review logical design only |
| #7 | Configure PostgreSQL/Prisma; generate auth tables and initial migration |
| #8–#9 | Add organization membership/invitation/permission integration |
| #11 | Add customer/booking models, scoped constraints and two-organization seed |
| #13–#16 | Add ticket/message, assignment/version, task and audit records |
| #17–#19 | Add documents, chunks, durable-job handoff and pgvector SQL migration |
| #24–#25 | Add runs, steps, proposals, decisions and executions |

Use reviewed migrations committed with their feature, never undocumented production schema edits. Use a dedicated test database. Seeds must be idempotent and synthetic. Verify fresh migration/seed and cross-organization rejection. Custom vector SQL must be parameterized and supported by the database host.

## 8. Issue #3 completion

Explain the booking→ticket→run→proposal→execution chain, check every required capability has a record owner, review isolation and uniqueness rules, and identify staged migrations. No runtime database/build claim is made until those checks actually exist and run.

Next: [implementation plan](implementation-plan.md), Issue #4.
