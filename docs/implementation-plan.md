# Implementation plan

Owner: Issue #4 — Create implementation roadmap. Source: guide §§17–20 plus the existing GitHub issue titles/milestones.

## Execution rule

Follow #1 through #30. Within each issue, finish prerequisites first, implement only its scope, verify, and merge before starting the next. The detailed specifications below expand generic issue descriptions locally; no GitHub issue has been edited. Use a playbook to update the corresponding issue when beginning work.

| Issue | Existing title | Existing milestone | Prerequisites | Playbook |
|---|---|---|---|---|
| #1 | Define MVP requirements | M0 - Foundation | None | [Read](issues/issue-01.md) |
| #2 | Document system architecture | M0 - Foundation | #1 | [Read](issues/issue-02.md) |
| #3 | Design database schema | M0 - Foundation | #1–#2 | [Read](issues/issue-03.md) |
| #4 | Create implementation roadmap | M0 - Foundation | #1–#3 | [Read](issues/issue-04.md) |
| #5 | Initialize Next.js application | M0 - Foundation | #1–#4 | [Read](issues/issue-05.md) |
| #6 | Configure linting and type checking | M0 - Foundation | #5 | [Read](issues/issue-06.md) |
| #7 | Implement authentication | M1 - Authentication | #6 | [Read](issues/issue-07.md) |
| #8 | Build organization management | M1 - Authentication | #7 | [Read](issues/issue-08.md) |
| #9 | Implement role-based access control | M1 - Authentication | #8 | [Read](issues/issue-09.md) |
| #10 | Build protected dashboard layout | M1 - Authentication | #9 | [Read](issues/issue-10.md) |
| #11 | Design customer data model | M2 - Operations | #10, design #3 | [Read](issues/issue-11.md) |
| #12 | Implement customer management | M2 - Operations | #11 | [Read](issues/issue-12.md) |
| #13 | Implement support ticket management | M2 - Operations | #12 | [Read](issues/issue-13.md) |
| #14 | Build ticket assignment and status workflow | M2 - Operations | #13 | [Read](issues/issue-14.md) |
| #15 | Build operations dashboard | M2 - Operations | #14 | [Read](issues/issue-15.md) |
| #16 | Implement activity audit log | M2 - Operations | #15 | [Read](issues/issue-16.md) |
| #17 | Implement secure document upload | M3 - AI Knowledge | #16 | [Read](issues/issue-17.md) |
| #18 | Build document processing pipeline | M3 - AI Knowledge | #17 | [Read](issues/issue-18.md) |
| #19 | Implement embeddings and vector search | M3 - AI Knowledge | #18 | [Read](issues/issue-19.md) |
| #20 | Build RAG retrieval pipeline | M3 - AI Knowledge | #19 | [Read](issues/issue-20.md) |
| #21 | Build knowledge base interface | M3 - AI Knowledge | #20 | [Read](issues/issue-21.md) |
| #22 | Integrate OpenAI API | M4 - AI Agents | #21 | [Read](issues/issue-22.md) |
| #23 | Implement AI response generation | M4 - AI Agents | #22 | [Read](issues/issue-23.md) |
| #24 | Build agent tool execution framework | M4 - AI Agents | #23 | [Read](issues/issue-24.md) |
| #25 | Implement human approval workflow | M4 - AI Agents | #24 and manual tasks #15 | [Read](issues/issue-25.md) |
| #26 | Build AI activity and execution history | M4 - AI Agents | #25 | [Read](issues/issue-26.md) |
| #27 | Implement automated testing | M5 - Production | #26; tests already added per feature | [Read](issues/issue-27.md) |
| #28 | Review security and authorization | M5 - Production | #27 | [Read](issues/issue-28.md) |
| #29 | Configure production deployment | M5 - Production | #28 | [Read](issues/issue-29.md) |
| #30 | Prepare portfolio case study and release documentation | M5 - Production | #29 | [Read](issues/issue-30.md) |

## Required guide scope assigned to existing issues

These are explicit proposed scope clarifications, not new issues or claims that remote issue bodies already contain them.

| Guide requirement not separately named in the 30 titles | Owner | Reason |
|---|---|---|
| PostgreSQL/Prisma/auth migration setup | #7 | Persistent authentication needs the data foundation |
| CI and initial test harness | #6 | Checks must run before business features grow |
| Booking model and synthetic booking context | #11–#12 | Ticket/AI investigation needs linked booking data |
| Manual follow-up task workflow | #15 | Dashboard needs real work records; #25 needs a task service |
| Private storage | #17 | Secure uploads need protected original bytes |
| Redis/BullMQ/worker and queue reconciliation | #18 | Processing must survive restarts and failed enqueue |
| Embedding API configuration | #19 | Embeddings are needed before the later generation adapter #22 |
| Retrieval fixtures | #20 | Search correctness must be checked before generation |
| AgentRun/AgentStep persistence | #24 | Execution history must exist before its interface #26 |
| AI evaluation fixtures/runner | #27 | Automated quality includes AI behavior, not only app tests |
| Recovery and deployment runbook | #28–#30 | Security and release need operating evidence |

## Mapping the source guide's task keys

| Source key | Existing GitHub issue coverage |
|---|---|
| OP-01 | #1–#4; existing repository/templates/board setup |
| OP-02 | #5 |
| OP-03 | #6; feature tests throughout; expanded #27 |
| OP-04 | #7 identity/database foundation; business seed #11 |
| OP-05 | #7–#10 |
| OP-06 | #11–#12 |
| OP-07 | #13–#14 |
| OP-08 | #15 |
| OP-09 | #17 |
| OP-10 | #18 |
| OP-11 | #19–#21 |
| OP-12 | #22–#23 |
| OP-13 | #24 |
| OP-14 | #25 |
| OP-15 | #16 business audit; #24 persistence; #26 history and usage |
| OP-16 | #27 with earlier retrieval/feature tests |
| OP-17 | #28 |
| OP-18 | #29–#30 |

## Milestone exit gates

- M0 — Foundation (#1–#6): reviewed specifications, running scaffold, repeatable checks.
- M1 — Authentication (#7–#10): sessions, workspaces, roles and protected shell with isolation checks.
- M2 — Operations (#11–#16): customers/bookings, tickets, tasks, dashboard and audit work without AI.
- M3 — AI Knowledge (#17–#21): private documents process durably into scoped, inspectable evidence.
- M4 — AI Agents (#22–#26): grounded investigations propose actions; only authorized exact-version review executes them.
- M5 — Production (#27–#30): measured evaluations, security/recovery checks, deployed demo and honest release documentation.

The guide's 12-week schedule is an estimate, not a deadline. Preserve dependency order and completion gates rather than treating issue count as progress. A prepared document does not close an issue until it has been reviewed, committed and merged through the workflow.
