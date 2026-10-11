# OpsPilot AI — MVP Requirements

Issue: #1 — Define MVP requirements

Milestone: M0 — Foundation

Status: Proposed requirements for review; features are not yet implemented.

## 1. Purpose

OpsPilot AI helps a small business manage customer support, search company
documents, and use an AI assistant to investigate tickets and propose actions
for employee approval.

The first demo business is a fictional travel agency. Employees currently switch
between customer records, bookings, policies, and messages to answer one request.
OpsPilot brings that information together and makes follow-up work traceable.

This document defines what version 1 must do. Architecture, database design, and
implementation order belong in their respective documents in `docs/`.
The baseline is the [original project guide](reference/project-guide.pdf), version 1.0. The [implementation plan](implementation-plan.md) maps its requirements to the existing GitHub issues #1–#30.

## 2. Main workflow

1. An employee signs in and selects an authorized company workspace.
2. They create or open a customer ticket and link the relevant booking.
3. They ask the AI assistant to investigate the ticket.
4. The assistant retrieves permitted records and relevant policy passages.
5. The employee sees a summary, sources, unanswered questions, and proposed actions.
6. An authorized employee edits, approves, or rejects each proposal.
7. The application executes approved actions and records their outcomes.

Example: a customer asks to change a hotel arrival date. The assistant finds the
booking-change policy and drafts a reply explaining that staff must check
availability and obtain an updated quote. It can propose a follow-up task. It
cannot confirm hotel availability, invent a fee, or change the booking.

## 3. Users and permissions

A user may belong to multiple organizations. Their role applies separately in
each organization. All access is checked on the server.

| Operation | Owner | Manager | Support agent |
|---|---|---|---|
| Manage workspace settings, invitations, and roles | Yes | No | No |
| Manage customers, tickets, and tasks | Yes | Yes | Yes |
| Upload and archive knowledge documents | Yes | Yes | No |
| Investigate accessible tickets | Yes | Yes | Yes |
| Approve proposed replies and follow-up tasks | Yes | Yes | Yes |
| Approve an AI proposal to resolve a ticket | Yes | Yes | No |
| View organization run history | Yes | Yes | Own runs only |
| View evaluation results | Yes | Yes | No |

## 4. Required features and acceptance checks

| ID | Feature | Required behavior and completion evidence |
|---|---|---|
| MVP-01 | Authentication | Employees can sign up, log in, and log out. Protected pages and APIs reject unauthenticated access. |
| MVP-02 | Workspaces and team | Owners can create a workspace, invite members, and manage roles. Members can select authorized workspaces. Tests with two organizations demonstrate that records, files, sources, and run history remain isolated. |
| MVP-03 | Customers | Employees can create, list, search, view, edit, and archive customers. Invalid input is rejected. Archived records retain historical links. |
| MVP-04 | Booking context | Manually entered or seeded booking records provide customer, reference, destination, dates, and status. A ticket can link only to a booking for the same customer and organization. AI access is read-only. |
| MVP-05 | Support tickets | Employees can manually create tickets, assign an active workspace member, filter the inbox, add internal notes or replies, and change status. Allowed status changes are validated and recorded. This works without AI. |
| MVP-06 | Follow-up tasks | Employees can create, assign, update, and complete tasks linked to a ticket. Approved AI proposals can create tasks through the same validation and permission rules. |
| MVP-07 | Knowledge documents | Owners and managers can upload TXT, Markdown, and text-based PDF files and archive documents. Processing exposes Pending, Processing, Ready, Failed, and Archived states. Invalid files and scanned PDFs produce useful errors. |
| MVP-08 | Knowledge retrieval | Search returns relevant passages and real source metadata from Ready, active documents in the authorized organization. Incomplete and archived content is excluded. Missing or conflicting evidence is shown honestly. |
| MVP-09 | AI investigation | One bounded agent investigates a ticket using permitted read-only tools. Results contain a summary, retrieved evidence references, unresolved questions, a draft reply, and applicable action proposals. Invalid output is rejected or shown as a failure. |
| MVP-10 | Approval and execution | An authorized person reviews the exact action payload before execution. Editing requires fresh approval. Rejection has no business effect. Stale proposals are blocked. Repeated approval or job delivery does not duplicate effects. |
| MVP-11 | Run history and audit | Authorized employees can inspect successful, partial, and failed runs: initiator, ticket, timestamps, model/prompt version, observed tool calls, sources, decisions, results, errors, and actual usage when available. Business changes have audit records. |
| MVP-12 | Dashboard and usability | A workspace dashboard summarizes ticket, task, and run workload. Main screens work on desktop and mobile and provide loading, empty, error, validation, and success states. Lists are paginated. |
| MVP-13 | Evaluation | Versioned fixtures and a runnable evaluation script cover grounded answers, missing evidence, unsafe requests, authorization, approvals, duplicate execution, and failures. Reports include dataset version, sample sizes, and known failures. Start with 20 curated scenarios. |
| MVP-14 | Deployment | The deployed application, database, private document storage, and durable worker support the complete demo workflow. Setup, required environment variables, migrations, and checks are documented. |

## 5. AI boundaries

The initial agent may request these backend tools:

- `getTicketContext`: read the authorized ticket and relevant customer messages.
- `getBookingDetails`: read the linked customer's booking in the same organization.
- `searchKnowledgeBase`: retrieve permitted policy passages.
- `getRelatedTickets`: read limited support history for the same customer.

The only initial proposed business actions are:

- `create_follow_up_task`
- `publish_ticket_reply`
- `resolve_ticket`

Investigation must not execute these business actions automatically. Storing a
draft or proposal is not publication or execution. Approval of a reply does not
approve ticket resolution; each action is reviewed separately.

The backend validates tool names, arguments, retrieved evidence IDs, and proposal
payloads. It enforces configurable tool-call, time, token, and spend limits.
The model never receives unrestricted database access or approval authority.
Ticket text and uploaded documents are untrusted evidence, not instructions
that can override permissions or system rules.

## 6. Validation, failure handling, and security

- Validate forms, API input, file content/type/size, tool arguments, and state changes on the server.
- Check active organization membership and action-specific permission for requests, tools, approvals, file access, and worker execution.
- Filter organization data before retrieval results reach the model.
- Keep credentials server-side and original documents in private storage. Do not expose secrets, stack traces, or unnecessary customer data in logs or provider requests.
- Reject invalid customer/booking links and assignees outside the organization.
- Recheck permissions, target records, and relevant source versions before approved execution. Changed context requires a new review.
- Apply approved database writes, execution results, and audit records transactionally with duplicate-execution protection.
- Use durable background jobs for ingestion, bounded retries for temporary errors, and idempotent handlers. Worker crashes must not silently lose accepted work.
- Show useful errors for invalid files, missing records, rate limits, provider failures, and stale approvals. Never display a failed action as completed.
- Keep manual customer, ticket, and task workflows usable when the AI provider is unavailable.
- Protect cookie-authenticated mutations against cross-site request forgery and enforce rate limits on expensive operations.

## 7. Screens

- Login and signup
- Workspace setup and switcher
- Dashboard
- Customer list and customer detail with booking context
- Ticket inbox and ticket detail with conversation, sources, and proposals
- Tasks
- Knowledge base with processing status
- Agent run detail and permitted run history
- Workspace settings and team management
- Evaluation results

## 8. Outside version 1

Real inbound/outbound email, WhatsApp, customer-facing portals, live supplier or
hotel APIs, booking changes, payments, refunds, OCR, advanced analytics, MCP,
multiple agents, and richer tracing are later enhancements.

Tickets are entered manually. Publishing a reply saves it to the internal demo
ticket conversation; the interface must not claim that an email or WhatsApp
message was delivered. Demo customers, policies, and bookings are fictional.

## 9. Release demonstration and verification

Version 1 is ready to demonstrate when an employee can complete the main
workflow on the deployed application and the following evidence is available:

- A booking-change ticket produces an evidence-backed reply and follow-up proposal without inventing availability or pricing.
- Approving a reply publishes one internal message; approving a task creates one task, including under duplicate submissions.
- Rejecting an action creates no business effect, and a stale proposal cannot execute.
- A support agent cannot approve AI ticket resolution or access another organization's records, files, or evidence.
- Missing policy information, a scanned PDF, and AI-provider failure each produce an honest, useful result.
- Run history records sources, review decisions, execution outcomes, and failures.
- Applicable automated tests, lint, type checking, and production build pass once application tooling exists.
- Evaluation results report actual measurements and known failures without unsupported accuracy or performance claims.

## 10. Completion of Issue #1

This issue delivers the requirements document, not the implemented application.
Review that the purpose, users, scope, exclusions, failure handling, security,
and acceptance checks above match the intended first release. The pull request
must reference `Closes #1` and describe the documentation review performed.

For a documentation-only repository with no application scaffold, application
tests, lint, type checking, and build are not applicable yet; do not report them
as passing. They become required for the relevant implementation issues.

Exact ticket/task transitions, upload limits, retention settings, model IDs,
runtime budgets, package versions, and hosting choices must be recorded in the
relevant design or implementation issue before those features are built.
