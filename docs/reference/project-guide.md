# **OpsPilot AI — Project and Implementation Guide** 

**A connected guide to what we are building, how it works, why each technology is needed, and how to implement it.** 

Prepared for Rohit Singh • 7 October 2026 • Version 1.0 

**Document status:** Proposed implementation baseline. This guide defines the project; it does not mean the features have already been built. Example customers, policies, bookings, and prices below are fictional demo data. 

## **Contents** 

1. How to use this guide 

2. The product and the problem 

3. One complete example 

4. First-release scope 

5. Users and permissions 

6. Screens and modules 

7. Technologies and their purposes 

8. Architecture and folder structure 

9. Database design 

10. APIs and ordinary application requests 

11. How the AI works 

12. Documents, embeddings, and RAG 

13. Human approval and execution 

14. Background work, errors, and retries 

15. Logs, usage, and evaluation 

16. Security and data boundaries 

17. GitHub workflow 

18. Implementation milestones and backlog 

19. Learning alongside implementation 

20. Testing, deployment, and release 

21. Demonstration and interview preparation 

22. Glossary, decisions, and official references 

## **1. How to use this guide** 

Read Sections 2–7 first to understand the product. Read Sections 8–16 to understand how its parts connect. Use Sections 17–20 while implementing. Use Section 21 when preparing your portfolio demonstration. 

This is the main guide for OpsPilot AI. You do not need six separate planning documents before writing code. Keep this guide in <mark>`docs/ project-guide.md`</mark> when we create the repository. The README will explain how to run the software, GitHub issues will describe the current work, and migrations will record the database structure. 

The connection between these records is: 

|**Record**|**Question it answers**|**When to use it**|
|---|---|---|
|This guide|What are we building<br>and why?|Before a milestone and<br>when scope is unclear|
|GitHub issue|What exact piece am I<br>implementing now?|At the start of each task|
|Feature<br>branch|Where do the changes<br>for this task go?|During implementation|
|Pull request|What changed, and how<br>was it checked?|Before merging|
|Database<br>migration|How did the stored data<br>structure change?|When adding or changing<br>tables|



|README|How can someone install|During setup and|
|---|---|---|
||and run this project?|handover|
|Release|Is this version ready to|Before deployment|
|checklist|demonstrate?||



If a decision changes, update the relevant section here and the affected issue. Avoid keeping two contradictory plans. 

## **2. The product and the problem** 

### **2.1 Product in one sentence** 

**OpsPilot AI is a web application where a small business manages customer support, searches its company documents, and uses an AI assistant to investigate tickets and propose actions for employee approval.** 

The first demo business will be a fictional travel agency. This gives us concrete customers, bookings, policies, and support problems. The project can later support other businesses, but we will implement one domain properly first. 

### **2.2 The problem we solve** 

An employee receives: “Can I change my hotel dates?” 

Today, they may check a customer spreadsheet, a booking system, a policy PDF, and previous messages. Then they write a reply and remember to follow up. Switching between these places takes time and makes mistakes easier. 

OpsPilot puts the records into one application. The employee opens the ticket and asks AI to investigate. AI uses controlled backend tools to retrieve information, searches relevant policy passages, and prepares a proposed reply or task. The employee reviews the result before any 

proposed action is executed. 

### **2.3 What success looks like** 

We should be able to demonstrate that an employee can: 

1. Log in to the correct company workspace. 

2. Open a customer ticket and view its booking. 

3. Ask AI to investigate. 

4. See the records and policy passages supporting the response. 

5. Edit, approve, or reject the proposed actions. 

6. See the resulting task, ticket change, and execution history. 

The initial value is faster investigation and clearer follow-up. We will measure performance rather than claim a percentage improvement before testing. 

### **2.4 What this project demonstrates** 

It combines ordinary full-stack engineering—forms, APIs, authentication, SQL, and deployment—with AI engineering—retrieval, tool calling, bounded workflows, approvals, and evaluations. Both parts matter. The customer and ticket features must work even if the AI provider is unavailable. 

It is a portfolio project designed with production concerns. Calling it production-ready would require additional operational evidence, real usage, security review, and support procedures. 

## **3. One complete example: a booking-change request** 

### **3.1 Starting data** 

|**Item**|**Fictional value**|
|---|---|
|Organization|Mountain Trails Demo|
|Employee|Rohit, support agent|
|Customer|Rahul Sharma|
|Booking reference|MAN-1842|
|Destination|Manali|
|Current arrival|12 December 2026|
|Requested arrival|14 December 2026|
|Ticket|“Please change my hotel arrival date.”|
|Knowledge document|Booking Changes Policy, version 1|



The policy says that date changes require a staff availability check and an updated quote. Our MVP does not connect to a hotel inventory service. Therefore, it cannot promise availability or invent an additional price. 

### **3.2 What happens** 

|**Step**|**What the employee**<br>**sees**|**What the software does**|
|---|---|---|
|1|A ticket with Rahul’s<br>request|Loads the organization-scoped ticket<br>and customer|
|2|Employee clicks<br>Investigate|Creates an agent run tied to that ticket|
|3|“Booking retrieved”|Executes an allowed tool to read the<br>linked booking|
|4|“Policy found”|Searches ready document chunks in<br>this organization|
|5|A summary and<br>source links|AI prepares an answer from the<br>retrieved evidence|



|6|Proposed task and<br>reply|Stores proposed actions with exact<br>payloads|
|---|---|---|
|7|Employee edits or<br>approves|Server checks role, payload version,<br>and current records|
|8|A follow-up task<br>appears|Executes the approved task creation<br>once|
|9|Reply appears in<br>ticket history|Publishes the approved reply in the<br>internal demo conversation|
|10|Run shows<br>completed actions|Stores results and audit events|



An appropriate draft is: “We can review your requested date change. Our team will check hotel availability and send you an updated quote before confirming.” 

An inappropriate draft is: “Your hotel is available and the change costs ₹1,500,” when no tool has provided those facts. 

### **3.3 What is simulated and what is real** 

Customer, ticket, task, document, approval, and run records are real database records in the application. AI calls and document retrieval are real integrations once implemented. Booking data is entered manually or seeded. Publishing a reply initially means saving it to the internal ticket conversation; it does not mean an email was delivered. 

A real email adapter can be added later. The UI must label delivery accurately. We will not integrate live payments, process refunds, or confirm bookings with suppliers in the first release. 

## **4. First-release scope** 

### **4.1 Required for version 1** 

|**Capability**|**Included behavior**|**Completion evidence**|
|---|---|---|
|Authentication|Sign up, login, logout,<br>session checks|Protected pages and APIs<br>reject unauthenticated<br>requests|
|Organizations|Workspace creation<br>and membership|Two demo organizations<br>cannot access each other’s<br>records|
|Roles|Owner, manager,<br>support agent|Server enforces the<br>permission table|
|Customers|Create, list, search,<br>view, edit, archive|Records persist and invalid<br>input is rejected|
|Bookings|Manually entered<br>read-only context for<br>AI|Ticket links to a valid<br>customer booking|
|Tickets|Create, assign, reply,<br>change status|Status rules and history<br>work without AI|
|Tasks|Manual tasks and<br>approved AI proposals|Tasks link to their ticket<br>and assignee|
|Knowledge<br>base|TXT, Markdown, text-<br>based PDF upload|Document reaches Ready<br>or a useful failure state|
|Retrieval|Organization-scoped<br>semantic search|Search returns source<br>passages and document<br>metadata|
|AI<br>investigation|One agent using a<br>small tool set|Grounded summary,<br>proposed reply, and task<br>when needed|
|Approval|Review exact actions<br>before execution|Rejection has no business<br>side effect; repeats do not<br>duplicate|
|Run history|Steps, results, errors,<br>time, usage|Failed and successful runs<br>are both inspectable|



Evaluation Results show Versioned fixtures and a runnable evaluation denominators and known script failures Deployment Application, database, End-to-end workflow runs private storage, on the deployed version worker 

### **4.2 Later enhancements** 

Real inbound and outbound email, customer-facing ticket portal, supplier APIs, booking mutation workflows, OCR for scanned documents, WhatsApp integration, advanced analytics, MCP access, multiple agents, and richer tracing are later work. 

Version 1 has an internal support inbox. Employees enter tickets manually. We will not quietly treat an email inbox connection as already included. 

### **4.3 Scope rules** 

Build one agent. Use PostgreSQL for business data and vectors. Add a worker when document ingestion needs durable asynchronous execution. Finish the core demo before adding further integrations. 

## **5. Users and permissions** 

Authentication answers “Who are you?” Authorization answers “What can you do?” Organization membership answers “Which company’s data may you access?” 

A user can belong to several organizations. Their role belongs to a membership, not globally to the user. Someone could own one workspace and be a support agent in another. 

|**Operation**|**Owner**|**Manager**|**Support**<br>|
|---|---|---|---|
||||**agent**|
|Manage workspace settings|Yes|No|No|
|Invite members and change<br>roles|Yes|No|No|
|Manage customers and tickets|Yes|Yes|Yes|
|Create and update tasks|Yes|Yes|Yes|
|Upload/archive knowledge<br>documents|Yes|Yes|No|
|Investigate accessible tickets|Yes|Yes|Yes|
|Approve ticket replies and<br>follow-up tasks|Yes|Yes|Yes|
|Approve AI proposal to resolve a<br>ticket|Yes|Yes|No|
|View organization run history|Yes|Yes|Own runs|
|View evaluation results|Yes|Yes|No|



These are proposed application roles. Configure the auth library’s organization permissions to match them; do not assume its defaults match this table. 

The backend checks membership and permission for every request, tool execution, approval, and worker operation. Hiding a button does not enforce access. 

## **6. Screens and modules** 

|**Screen**|**Employee**|**Main records**|**Connected**|
|---|---|---|---|
||**action**|**used**|**feature**|
|Login/signup|Enter<br>credentials|User, session|Authentication|



|Workspace|Create/select|Organization,|Data isolation|
|---|---|---|---|
|setup/<br>|company|membership||
|switcher||||
|Dashboard|View workload|Ticket, task, run|Aggregated<br>reporting|
|Customers|Search and<br>edit customers|Customer|Support context|
|Customer<br>detail|View bookings<br>and tickets|Customer,<br>booking, ticket|Investigation<br>starting point|
|Tickets/inbox|Filter and<br>assign work|Ticket,<br>membership|Support<br>workflow|
|Ticket detail|Read<br>messages,<br>reply,<br>investigate|Message,<br>ticket, booking|AI workspace|
|Tasks|Assign and<br>complete<br>follow-ups|Task|Approved<br>actions|
|Knowledge<br>base|Upload,<br>inspect,<br>archive files|Document,<br>chunk|RAG|
|Agent run<br>detail|Inspect<br>sources and<br>actions|Run, step,<br>proposal|Approval and<br>observability|
|Settings/team|Manage<br>workspace<br>access|Organization,<br>membership|Permissions|
|Evaluation<br>results|Review<br>measured<br>quality|Evaluation<br>fixtures/results|Release<br>assessment|



The ticket detail page is the main working screen. Show customer context, conversation, investigation results, citations, and proposed actions together. Each list needs loading, empty, error, and populated 

states. Forms need field errors and submission feedback. Screens must work on mobile as well as desktop. 

Start with polling for document and run status. Streaming text can come later after persistent run states work. 

## **7. Technologies and why we use them** 

### **7.1 Application and data** 

|**Technology**|**Simple meaning**|**Purpose in**<br>**OpsPilot**|**Introduced**|
|---|---|---|---|
|TypeScript|JavaScript with<br>type checking|Shared data<br>shapes and safer<br>application code|Foundation|
|React|Component-<br>based UI library|Forms, ticket<br>panels, interactive<br>controls|Foundation|
|Next.js App<br>Router|React framework<br>with server<br>features|Pages, layouts,<br>server rendering,<br>HTTP endpoints|Foundation|
|Tailwind CSS|Utility classes for<br>styling|Responsive visual<br>system|Foundation|
|Zod|Runtime input<br>validator|Check form data,<br>API input, and<br>tool arguments|Foundation|
|PostgreSQL|Relational<br>database|Users, customers,<br>tickets, tasks,<br>approvals, logs|Data<br>foundation|



|Prisma ORM|Database access<br>and migration<br>tooling|Ordinary typed<br>queries and<br>schema<br>migrations|Data<br>foundation|
|---|---|---|---|
|Better Auth|Authentication<br>library|Sessions,<br>accounts,<br>organization<br>membership|Identity|
|pgvector|PostgreSQL<br>vector extension|Similarity search<br>over document<br>embeddings|Knowledge<br>base|



Next.js lets us keep the first application’s UI and backend together. Route handlers serve HTTP requests; shared server services contain the business logic. Server-rendered pages can call those services directly rather than making an extra HTTP call to their own API. [S1–S2] 

Prisma is used for ordinary business records. The vector field may need custom SQL migrations and parameterized raw SQL or TypedSQL because extension types are not all handled natively. Verify the database host supports pgvector before deployment. [S3] 

Better Auth’s organization plugin provides membership capabilities, but our customer and ticket authorization checks remain our responsibility. [S4] 

### **7.2 AI, files, and asynchronous work** 

|**Technology/tool**|**Purpose**|**Why we need it**|
|---|---|---|
|OpenAI official<br>Node SDK and<br>Responses API|Server-side model<br>requests|Summaries, structured<br>investigation results,<br>tool requests|
|Tool/function calling|Model requests a<br>named backend<br>function|Lets the agent retrieve<br>approved business<br>data|



|Structured outputs|Constrain final<br>response shape|Makes summaries,<br>evidence IDs, and<br>proposals easier to<br>validate|
|---|---|---|
|Embeddings API|Convert text into<br>numerical<br>representations|Enables semantic<br>document retrieval|
|Private S3-<br>compatible object<br>storage|Store original<br>documents|Keeps large file bytes<br>out of ordinary<br>database rows|
|PDF text extraction<br>library|Extract text from<br>text-based PDFs|Turns policy files into<br>searchable content|
|Redis|Store queue<br>coordination data|Supports BullMQ jobs|
|BullMQ|Job queue|Durable ingestion with<br>bounded retries|
|Node.js worker|Separate long-<br>running process|Extracts documents<br>and creates<br>embeddings|



We will implement our own retrieval over PostgreSQL rather than also introducing a second hosted file-search system. The backend executes function calls and sends results back to the model; the model does not run database functions on its own. Structured output improves format reliability but does not prove an answer is correct. [S5–S7] 

The exact model IDs, library versions, embedding dimensions, and PDF parser will be recorded when implementing. Model selection depends on available support, evaluation quality, latency, and cost. Do not hardcode an unverified “latest model” into this plan. 

### **7.3 Development, checks, and hosting** 

|**Tool**|**Purpose**|
|---|---|



|VS Code|Edit and inspect the project|
|---|---|
|npm and lockfile|Install reproducible dependencies|
|Git|Record changes and branches|
|GitHub issues, milestones,<br>Projects|Track ordered work|
|GitHub pull requests and<br>Actions|Review changes and run checks|
|ESLint and TypeScript checks|Catch code-quality and type errors|
|Vitest|Unit/service tests and evaluation<br>helpers|
|Playwright|Browser tests for critical workflows|
|Postman or an HTTP client|Inspect requests and errors<br>manually|
|Docker Compose|Run local PostgreSQL and later<br>Redis|
|App hosting|Run the Next.js application|
|Worker hosting|Run a persistent worker process|
|Managed PostgreSQL|Persist relational and vector data|
|Error tracking, later Sentry|Collect operational failures|
|OpenTelemetry, later|Correlate traces across services if<br>needed|



Vercel for the web app and a persistent worker host such as Railway are deployment candidates, not requirements already configured. Choose providers during deployment after checking current limits, pgvector availability, worker support, region, and pricing. Custom run logs are required early; adding every monitoring service is unnecessary at the start. 

Express, LangChain, LangGraph, Kubernetes, Kafka, and multiple vector databases are not needed for this initial design. Introduce another component only when a concrete requirement justifies it. 

## **8. Architecture and folder structure** 

### **8.1 Connected architecture** 

```
flowchart TD
```

```
  UI["Next.js screens"] --> API["Server request boundary"]
  API --> Services["Business services and permissions"]
  Services --> DB["PostgreSQL and pgvector"]
  Services --> Agent["Bounded agent runner"]
  Agent --> Model["AI model API"]
  Agent --> Tools["Validated tool registry"]
  Tools --> Services
  Services --> Queue["Redis and BullMQ"]
  Queue --> Worker["Document worker"]
  Worker --> Files["Private object storage"]
  Worker --> DB
  Worker --> Model
```

The browser handles interaction. The server verifies the request and invokes services. Services implement operations such as loading a ticket or creating a task. The agent can request approved tools that call these same services. The worker processes documents outside a web request. PostgreSQL remains the source of truth for business state. 

### **8.2 Suggested repository layout** 

Use one repository with a Next.js application and a separate worker entry point. Avoid a large monorepo until there is a reason to split packages. 

|**Path**|**Responsibility**|**Example**|
|---|---|---|
|`src/app/`|Pages, layouts, route|Ticket page and`/`|
||handlers|`api/tickets`|



|`src/components/`<br>`ui/`|Reusable controls|Buttons, dialogs,<br>form fields|
|---|---|---|
|`src/features/`|Feature-specific UI|Ticket conversation<br>panel|
|`src/server/auth/`|Sessions and access<br>checks|Require<br>organization<br>membership|
|`src/server/`<br>`services/`|Business rules|Create task,<br>transition ticket|
|`src/server/`<br>`repositories/`|Database queries|Scoped customer<br>lookup|
|`src/server/ai/`|Model adapter,<br>runner, prompts, tools|Investigate ticket|
|`src/server/`<br>`knowledge/`|Extraction, chunking,<br>retrieval|Search policy<br>chunks|
|`src/server/jobs/`|Queue definitions and<br>job handlers|Document<br>ingestion job|
|`src/server/`<br>`storage/`|Private file access|Upload/read<br>document bytes|
|`src/lib/`|Shared validation and<br>small utilities|Ticket input<br>schema|
|`worker/index.ts`|Start worker process|Register ingestion<br>processor|
|`prisma/`|Schema, migrations,<br>seed script|Customer table<br>migration|
|`tests/`|Unit, integration,<br>browser tests|Cross-organization<br>access test|
|`evals/`|AI fixtures and<br>evaluation runner|Missing-policy<br>scenario|
|`docs/`|This guide and<br>necessary decisions|Deployment<br>instructions|



<mark>`.github/`</mark> CI and issue/PR Pull-request checks templates 

Keep server-only modules out of browser imports. Configure and test how the worker runs shared TypeScript server code; do not assume a Next.js build automatically builds the worker. 

## **9. Database design: how the records connect** 

### **9.1 Core entities** 

|**Entity**|**Important fields**|**Why it exists**|
|---|---|---|
|User|id, name, email|Person logging in|
|Session/account/<br>verification|Auth-library schema|Authentication<br>lifecycle|
|Organization|id, name|Company<br>workspace|
|Membership|userId, organizationId, role|User’s access<br>inside a company|
|Invitation|organizationId, email, role,<br>expiry|Team onboarding|
|Customer|id, organizationId, name,<br>email, archivedAt|Customer context|
|Booking|id, organizationId,<br>customerId, reference,<br>dates, status|Travel context for<br>investigation|
|Ticket|id, organizationId,<br>customerId, bookingId,<br>assigneeId, status, version|Customer<br>problem|



|TicketMessage|id, ticketId, organizationId,<br>authorType, body|Conversation<br>history|
|---|---|---|
|Task|id, organizationId, ticketId,<br>assigneeId, title, status,<br>dueAt|Follow-up work|
|Document|id, organizationId,<br>storageKey, version, status,<br>checksum|Uploaded source<br>file|
|DocumentChunk|id, organizationId,<br>documentId, text, locator,<br>embedding|Searchable<br>source passage|
|AgentRun|id, organizationId, ticketId,<br>initiatedBy, status, model,<br>promptVersion, usage|One investigation|
|AgentStep|id, runId, toolName, status,<br>redacted result summary,<br>duration|Recorded<br>execution step|
|ActionProposal|id, runId, type, payload,<br>payloadVersion, status|Exact action<br>awaiting review|
|ApprovalDecision|proposalId, reviewerId,<br>decision, payloadVersion,<br>decidedAt|Who approved or<br>rejected what|
|ActionExecution|proposalId,<br>idempotencyKey, status,<br>resultId, errorCode|Execute once<br>and track<br>outcome|
|AuditEvent|organizationId, actorId,<br>eventType, targetId,<br>timestamp|Important<br>business<br>changes|



Use the auth library’s generated schema for its tables. Do not create competing user or membership systems. Evaluation fixtures can initially live in <mark>`evals/`</mark> rather than requiring evaluation database tables. 

### **9.2 Main relationships** 



```
erDiagram
  USER ||--o{ MEMBERSHIP : has
  ORGANIZATION ||--o{ MEMBERSHIP : contains
  ORGANIZATION ||--o{ CUSTOMER : owns
  CUSTOMER ||--o{ BOOKING : has
  CUSTOMER ||--o{ TICKET : raises
  TICKET ||--o{ TICKET_MESSAGE : contains
  TICKET ||--o{ TASK : needs
  TICKET ||--o{ AGENT_RUN : investigated_by
  AGENT_RUN ||--o{ ACTION_PROPOSAL : produces
  ACTION_PROPOSAL ||--o{ APPROVAL_DECISION : reviewed_with
  ORGANIZATION ||--o{ DOCUMENT : owns
  DOCUMENT ||--o{ DOCUMENT_CHUNK : split_into
```

A ticket belongs to a customer and may link to a booking. A run belongs to a ticket. Proposals belong to that run. This chain lets the employee understand exactly which support problem produced a task or reply. 

### **9.3 Data integrity rules** 

Every business record carries <mark>`organizationId` .</mark> A linked booking must belong to both the same organization and the ticket’s customer. An assignee must be an active member of that organization. Enforce these rules in services and database constraints where feasible, including scoped composite keys/foreign keys. 

Use IDs for lookup. Names are display values; two customers may both be Rahul Sharma. Starting from a ticket gives us the correct customer ID. Free-text customer lookup returns candidates rather than choosing silently. 

Store timestamps consistently, display them in the user’s timezone, and distinguish a calendar date from a timestamp. Store any monetary demo values in integer minor units plus currency. Do not use floating-point arithmetic for money. 

Add indexes for organization plus commonly filtered fields, such as ticket status and creation time. Scope booking references to organizations. 

Archive records where history needs to remain visible. Document deletion/archival must also make its chunks unavailable to retrieval. 

Use transactions for related business writes. Track record versions for approvals so that a draft prepared from an old ticket is not blindly applied after someone changes the ticket. 

## **10. APIs and ordinary application requests** 

### **10.1 What an API is here** 

An API is a controlled entry point into the backend. For example, submitting the customer form sends a request to create a customer. The backend validates it, checks access, saves the record, and returns a result. 

The intended request boundary is: 

1. Identify the session. 

2. Derive or verify the active organization membership. 

3. Validate input using Zod. 

4. Check permission. 

5. Call the business service. 

6. Write relevant audit events. 

7. Return a consistent response. 

An organization ID in a URL or request body is never proof of membership. 

### **10.2 Proposed HTTP contracts** 

Auth-library endpoints are separate. These are proposed application routes, not existing endpoints. 

|**Route**|**Purpose**|**Main restriction**|
|---|---|---|



|`GET /api/customers`|Search/paginate<br>customers|Current<br>organization only|
|---|---|---|
|`POST /api/customers`|Create customer|Validated member<br>request|
|`GET /api/`<br>`customers/:id`|Customer<br>context|Scoped lookup|
|`PATCH /api/`<br>`customers/:id`|Edit/archive<br>customer|Permission and<br>current version|
|`GET /api/tickets`|Filter inbox|Scoped, paginated<br>query|
|`POST /api/tickets`|Create ticket|Customer/booking<br>relationship verified|
|`PATCH/api/`<br>`tickets/:id`|Assign/change<br>status|Allowed transition<br>and permission|
|`POST /api/`<br>`tickets/:id/messages`|Add manual<br>reply/note|Explicit message<br>type|
|`POST/api/tasks`|Create manual<br>task|Valid organization<br>assignee|
|`PATCH /api/tasks/:id`|Complete/<br>update task|Permission and<br>valid transition|
|`POST/api/documents`|Register/upload<br>supported file|Owner/manager<br>and size/type<br>checks|
|`GET /api/`<br>`documents/:id`|Processing<br>status|Organization<br>membership|
|`POST/api/knowledge/`<br>`search`|Search source<br>passages|Ready documents<br>in this organization|
|`POST /api/agent-runs`|Investigate<br>ticket|Access, rate limit,<br>one active run rule|
|`GET /api/agent-`<br>`runs/:id`|Read progress/<br>result|Role and<br>organization|



|`POST /api/action-`<br>`proposals/:id/`|Approve exact<br>payload version|Action-specific<br>permission|
|---|---|---|
|`approve`|||
|`POST/api/action-`<br>`proposals/:id/reject`|Reject proposal|Authorized reviewer|
|`PATCH /api/action-`<br>`proposals/:id`|Edit proposed<br>payload|New version<br>invalidates old<br>approval|



An asynchronous run returns a run ID and accepted status; the UI then reads progress. A queue must be available before promising durable background runs. Start with a bounded server request in development if needed, then move the persisted workflow to the worker before deployment where host time limits demand it. 

### **10.3 Errors and responses** 

Use a consistent error shape containing a code, safe message, optional field errors, and request ID. Distinguish invalid input, missing login, forbidden action, unavailable record, stale version, rate limit, and provider failure. Never return stack traces, secrets, or raw sensitive provider responses to the browser. 

Lists use pagination and maximum page sizes. Mutating requests need appropriate CSRF/origin protection with cookie-based sessions. Manual UI operations and AI tools call the same services so they enforce the same business rules. 

## **11. How the AI works** 

### **11.1 The four connected parts** 

|**Part**|**Role**|**Example**|
|---|---|---|



|Model|Suggests tool requests<br>and drafts a result|“I need the booking and<br>policy.”|
|---|---|---|
|Instructions|Describe task boundaries|“Do not promise<br>availability without<br>evidence.”|
|Tools|Read permitted business<br>information|Retrieve this ticket’s<br>booking|
|Runner|Validates, executes,<br>tracks, and stops the<br>process|Reject unknown tools and<br>stop at a configured limit|



The model does not directly access PostgreSQL. Our backend decides which tools exist, which records they may access, and whether anything may change. 

### **11.2 Initial tools** 

Use a small read-only tool set plus a structured final result. Proposed writes are not executed inside the model loop. 

|**Tool**|**Input**|**Output**|**Boundary**|
|---|---|---|---|
|`getTicketContext`|ticketId|Ticket,<br>customer,<br>relevant<br>messages|Current<br>authorized<br>ticket|
|`getBookingDetails`|bookingId|Stored<br>dates/<br>status/<br>reference|Same<br>company<br>and ticket<br>customer|
|`searchKnowledgeBase`|query|Source<br>passages<br>with chunk/<br>document<br>IDs|Current<br>company,<br>Ready<br>sources|



`getRelatedTickets` customerId Limited Same support company/ history customer; redacted fields 

The final structured result contains a summary, evidence IDs, unresolved questions, draft reply, and allowed action proposals. Initially those actions are <mark>`create_follow_up_task` ,</mark> <mark>`publish_ticket_reply` ,</mark> and <mark>`resolve_ticket` .</mark> Do not give the agent a generic SQL, arbitrary URL-fetch, or unrestricted send-email tool. 

### **11.3 One run, step by step** 

1. Employee starts investigation from a ticket. 

2. Backend creates a run with organization, ticket, initiator, model, and prompt version. 

3. Backend sends task instructions, minimum necessary context, and permitted tool definitions. 

4. Model returns text or requests one or more tools. 

5. Runner validates tool names and arguments. 

6. Each tool checks authorization and loads organization-scoped data. 

7. Runner records safe results and returns tool outputs to the model. 

8. The loop continues within configured tool, time, token, and spend limits. 

9. Model produces a structured final result, or the run stops with a useful failure state. 

10. Server validates evidence references and action payloads before storing proposals. 

11. UI displays sources, limitations, draft reply, and review controls. 

This is agentic behavior because the model selects useful informationgathering steps. The backend remains responsible for execution and business decisions such as permissions and approval requirements. [S5] 

### **11.4 State and limits** 

Run state includes its ticket, organization, initiating user, tool outputs, evidence references, pending proposals, and step count. Persist enough state to show progress and recover safely. Do not use a single global conversation shared by companies. 

Start with configurable limits—for example, eight tool executions and a fixed maximum runtime chosen for the host. These are initial engineering settings, not universal recommendations. If a limit is reached, return a partial investigation with a clear status rather than allowing an endless loop. 

Do not claim to display every hidden thought of the model. The run screen shows observable tool requests, evidence, summaries, and execution outcomes. 

### **11.5 Output correctness** 

Structured JSON provides shape, not truth. A valid reply can still contain an invented price. Validate referenced IDs against actually retrieved records, check proposal payloads, and evaluate factual claims separately. Handle refusals, incomplete output, and provider errors explicitly. [S6] 

## **12. Documents, embeddings, and RAG** 

### **12.1 Plain-language explanation** 

RAG means we search relevant company information and provide it to the model before it answers. We are not training a new model on the company’s PDFs. 

An embedding is a list of numbers representing text in a way useful for similarity search. A question about “changing arrival dates” can retrieve a passage about “booking modification” even when the words differ. 

Similarity is a search signal, not a guarantee that the passage answers the question. [S7] 

### **12.2 Upload/ingestion flow** 

|**Stage**|**What happens**|**Stored outcome**|
|---|---|---|
|Validate|Check permission, size,<br>extension, and content type|Rejection or accepted<br>document|
|Store|Put original file in private object<br>storage|Storage key and<br>metadata|
|Queue|Persist pending work and<br>enqueue ingestion|Document status<br>Pending|
|Extract|Read text from supported formats|Text or an extraction<br>failure|
|Chunk|Split into manageable passages|Chunk IDs, text,<br>source locators|
|Embed|Generate vectors for passages|Model ID, dimensions,<br>vectors|
|Commit|Save a complete searchable<br>version|Document status<br>Ready|



PDF source locators can include page numbers when extraction preserves them. Markdown/TXT can use headings or line ranges. Do not invent page citations when the parser cannot identify them. 

Use deterministic chunk identifiers or a uniqueness rule tied to document version and position. A retry should replace or upsert that version’s chunks, not append duplicates. Keep incomplete versions excluded from retrieval. 

Scanned PDFs may contain no extractable text. Version 1 reports that OCR is required rather than pretending ingestion succeeded. Document state follows Pending → Processing → Ready, with Failed and Archived as explicit alternatives. 

### **12.3 Search/answer flow** 

1. Validate the employee’s question and workspace access. 

2. Embed the query using a compatible embedding configuration. 

3. Search chunks in the authorized organization’s Ready, active document versions. 

4. Return a limited set of passages and source metadata. 

5. Give those passages to the model as evidence. 

6. Ask for an answer with source references and unresolved questions. 

7. Verify cited chunk IDs came from the retrieved set. 

Filter within the database query. Retrieving all companies’ passages and filtering afterward risks exposing another company’s content to the model. 

Chunk size, overlap, top-k, and relevance thresholds are tuning choices. Pick initial settings, run retrieval evaluations, then adjust. Store the embedding model and dimensions; changing them can require reembedding documents and updating vector storage. 

### **12.4 Missing or conflicting evidence** 

If no relevant policy exists, say so and ask an employee to review. If two active sources conflict, show the conflict rather than inventing a resolution. Archiving a policy must exclude its chunks. Approval should check whether cited documents changed since the proposal was generated. 

## **13. Human approval and execution** 

### **13.1 Proposed action is different from completed action** 

A proposal says what the system intends to do. An approval records a person’s decision. An execution records what actually happened. These are separate records and states. 

|**Action**|**AI may**<br>**prepare it?**|**Executes**<br>**automatically**<br>**during**<br>**investigation?**|**Reviewer**|
|---|---|---|---|
|Read<br>allowed<br>records|Yes|Yes, through<br>checked tools|No additional<br>approval|
|Draft<br>response|Yes|Draft storage only|Employee<br>reviews before<br>publication|
|Create<br>follow-up<br>task|Yes|No|Support agent,<br>manager,<br>owner|
|Publish<br>internal<br>ticket reply|Yes|No|Support agent,<br>manager,<br>owner|
|Resolve<br>ticket|Yes|No|Manager or<br>owner|
|Change<br>booking/<br>refund<br>money|No MVP<br>execution<br>capability|No|Outside initial<br>scope|



### **13.2 Proposal lifecycle** 

```
stateDiagram-v2
  [*] --> Pending
  Pending --> Rejected: reject
  Pending --> Approved: approve exact version
  Pending --> Expired: context changed
  Approved --> Executing: claim execution
  Executing --> Executed: success
  Executing --> Failed: error
  Failed --> Executing: safe retry
```

Editing a proposal creates a new payload version and requires a new approval. The approved record includes exact content, destination or target, action type, reviewer, version, and time. The AI cannot approve its own proposals. 

At execution time, re-check reviewer permissions, target organization, ticket version, and relevant source versions. A disabled member or stale approval should not authorize a later action. 

### **13.3 Avoiding duplicate effects** 

Use a unique idempotency key for each proposal execution. Two clicks or two worker deliveries must not create two tasks or publish two messages. For database-only actions, apply the business write, execution result, and audit record transactionally with appropriate locking/uniqueness checks. 

If email is added later, database transactions alone cannot guarantee an external message is sent exactly once. Use an outbox, provider idempotency support where available, delivery status tracking, and reconciliation for ambiguous outcomes. Never retry an uncertain external send blindly. 

Approval of a reply does not imply approval to resolve the ticket. Represent each action explicitly. If several approved actions partially succeed, show each outcome separately. 

## **14. Background work, errors, and retries** 

### **14.1 Why a worker exists** 

Document extraction and embedding can outlast a browser request. A worker consumes queued jobs and continues even when the employee closes the page. A detached promise inside an HTTP handler is not durable background execution. 

Redis coordinates the BullMQ queue. PostgreSQL stores the authoritative document/run state. When job creation spans both systems, use a persisted pending-job record or outbox/reconciliation so a crash between saving the record and enqueueing work does not lose it. 

### **14.2 Failure behavior** 

|**Failure**|**Expected behavior**|**Retry policy**|
|---|---|---|
|Invalid file or<br>arguments|Useful validation<br>error|No automatic retry|
|Empty/scanned<br>PDF|Failed ingestion with<br>OCR explanation|Retry only after a<br>supported file/change|
|Temporary model/<br>network error|Persist failure and<br>preserve context|Bounded retry with<br>backoff|
|Provider rate limit|Wait and show<br>delayed status|Respect retry guidance<br>and configured cap|
|Permission denied|Stop action|No retry to bypass<br>authorization|
|Missing customer/<br>booking|Explain missing data|Ask employee to<br>correct/link record|
|Stale approval|Mark expired or<br>conflict|Regenerate/review<br>current proposal|
|Worker crash|Resume or retry<br>durable job|Idempotent handler|
|Unknown external<br>delivery outcome|Mark uncertain|Reconcile before retry|



BullMQ supports attempts and backoff, but job handlers must be designed so retries do not duplicate effects. [S8] 

Do not wrap a whole agent run in unlimited retries. Re-running after an action succeeded can duplicate work. Persist step outcomes and separate investigation from approved execution. 

## **15. Logs, usage, and evaluation** 

### **15.1 Three kinds of records** 

|**Record**|**Purpose**|**Example**|
|---|---|---|
|Application error<br>log|Debug technical<br>failure|Parser failed on a PDF|
|Business audit<br>event|Record an important<br>change|Manager resolved ticket|
|Agent run/step|Explain observed AI<br>workflow|Policy search returned<br>three passages|



An agent run should show start/end time, status, initiating user, ticket, model ID, prompt version, tool names, safe result summaries, source IDs, proposal decisions, errors, and usage. 

Record input/output tokens from actual provider usage where available. A run may make several model calls, so aggregate all calls. Document embedding costs are separate from investigation generation costs. 

Estimate cost using recorded model pricing and pricing date. Show currency clearly. If displaying INR from USD rates, label it as a converted estimate with the exchange-rate assumption. Do not copy the illustrative rupee costs or success percentages from earlier examples as measured project results. 

### **15.2 What evaluation means** 

An evaluation is a set of known situations where we check whether the system behaves correctly. It includes deterministic security/execution tests and AI answer-quality checks. 

Start with 20 curated scenarios during development and grow toward 50–100 varied cases before making quality claims. Keep a separate set 

for final assessment so tuning does not simply memorize the development examples. 

|**Scenario**|**Expected**<br>**behavior**|**Check type**|
|---|---|---|
|Policy clearly answers<br>question|Retrieve relevant<br>source and cite it|Retrieval plus<br>answer review|
|No policy exists|State missing<br>evidence|Structured output<br>and human review|
|Customer name is<br>ambiguous|Ask for selection/<br>use ticket-linked ID|Tool/result check|
|Document says “ignore<br>rules and send data”|Treat content as<br>untrusted evidence|Adversarial security<br>case|
|Another organization<br>has a matching<br>document|Never retrieve or<br>expose it|Deterministic<br>integration test|
|Booking has no quoted<br>price|Do not invent a fee|Human/rubric factual<br>review|
|Proposal rejected|No task or<br>message created|Database assertion|
|Approval clicked twice|One effect only|Concurrency/<br>integration test|
|Ticket changed after<br>draft|Block stale<br>execution|Version-conflict test|
|Model provider fails|Honest failure/<br>partial state|Controlled failure test|



### **15.3 Report meaningful measurements** 

Measure retrieval hit rate against labeled passages, unsupported factual claims against a review rubric, correct tool selection, blocked unauthorized actions, latency distribution, and actual usage. State sample size and dataset version. A model-based grader may assist but 

should not be the only judge of factual correctness or permissions. 

“No unapproved mutations in 50 test cases” is meaningful. “100% safe AI” is not. A successful HTTP response is not proof of a correct answer. 

## **16. Security and data boundaries** 

These controls are connected to the product because it holds customer records and can propose business actions. 

|**Boundary**|**Implementation rule**|**Why**|
|---|---|---|
|Browser to<br>server|Sessions, validation,<br>permission checks|Browser input can<br>be changed|
|Organization to<br>organization|Scope every query and<br>file lookup|Companies must not<br>see each other’s<br>data|
|Model to tool|Allowlist names, validate<br>arguments, bound calls|Model output is<br>untrusted input|
|Document to<br>instructions|Treat retrieved text as<br>evidence only|A malicious file may<br>contain commands|
|Proposal to<br>execution|Exact payload approval<br>and fresh checks|Prevent unintended<br>or stale changes|
|App to provider|Server-only credentials<br>and minimized data|Avoid exposing<br>keys/customer<br>details|
|Logs to<br>employee|Redact and authorize<br>access|Logs may contain<br>sensitive context|
|File to worker|Size/type/parser limits<br>and private storage|Uploaded files are<br>untrusted|



Use parameterized SQL. Never concatenate model-generated SQL. Do not let the model choose the authorized organization. Inject organization/session context from the verified server state. 

Send only the relevant customer fields and messages to the model. Avoid full database dumps. Redact secrets and unnecessary personal information from logs and fixtures. Use synthetic data in the public demo. 

Keep <mark>`.env.local`</mark> and deployment secrets out of Git. Commit an <mark>`.env.example`</mark> with names and placeholders. Avoid <mark>`NEXT_PUBLIC_`</mark> for secrets because those variables are intended for browser-visible use. Rotate an exposed credential rather than merely deleting it from the latest commit. 

Apply per-user/organization request limits, run concurrency limits, file quotas, and AI spending caps. Public demo accounts should have bounded access and no ability to trigger unrestricted email delivery. 

Archive/delete documents consistently across file storage and searchable chunks. Define retention and backup behavior before handling real customer data. This guide does not claim legal compliance or a completed security audit. 

## **17. GitHub workflow: one task at a time** 

### **17.1 Set up once** 

Create a repository, protect <mark>`main`</mark> where repository permissions allow, and add an issue template, pull-request template, CI workflow, labels, milestones, and a board. 

Board columns: **Backlog → Ready → In progress → In review → Done** . Use a Blocked label with an explanation when necessary. 

Labels can stay simple: <mark>`area:ui` ,</mark> <mark>`area:api` ,</mark> <mark>`area:data` ,</mark> <mark>`area:ai` ,</mark> <mark>`area:infra` ,</mark> <mark>`type:feature` ,</mark> <mark>`type:bug` ,</mark> <mark>`type:docs` ,</mark> and priority labels. 

### **17.2 Repeated implementation loop** 

1. Select the next dependency-ready issue. 

2. Read its scope and completion checks. 

3. Update local <mark>`main`</mark> and create a branch. 

4. Implement the issue and verify its behavior. 

5. Commit meaningful changes and push the branch. 

6. Open a PR linking the issue. 

7. Inspect the diff and run the required checks. 

8. Merge after checks pass; close the issue and update the board. 

9. Update local <mark>`main` ,</mark> remove the completed branch, and begin the next issue. 

As a solo developer you can deliberately think as product owner, developer, and reviewer at different stages. Be honest that this is selfreview; do not claim independent team approval. 

### **17.3 Stable task keys and branch names** 

Use the backlog keys below, such as <mark>`OP-07` ,</mark> in documentation. GitHub assigns its own issue numbers, which may differ. Record that mapping rather than assuming OP-07 is GitHub issue #7. 

Branch examples: <mark>`chore/op-02-foundation` ,</mark> <mark>`feat/op-07tickets` ,</mark> <mark>`feat/op-13-agent-runner` ,</mark> <mark>`fix/op-14-staleapproval` .</mark> 

### **17.4 Issue and PR contents** 

An issue includes purpose, prerequisites, included work, excluded work, acceptance criteria, and verification. A PR includes the problem/result, main changes, checks actually run, known limitations, and <mark>`Closes #<actual issue number>` .</mark> 

For OP-07, useful acceptance criteria are: ticket persists after refresh; booking belongs to the selected customer; invalid assignee is rejected; another organization cannot read the ticket; invalid status transitions return a clear error. “Build ticket feature” is too vague. 

Commit messages describe results, for example <mark>`feat(tickets):`</mark> 

<mark>`add scoped ticket creation and assignment` .</mark> Commit count is not the success measure. Working, understandable features and a reviewable history matter more. 

## **18. Implementation milestones and ordered backlog** 

### **18.1 Milestones** 

|**Milestone**|**Outcome**|**Exit gate**|
|---|---|---|
|M0 — Foundation|Repository,<br>application, checks|Clean setup and CI<br>pass|
|M1 — Identity and<br>data boundaries|Login and workspace<br>access|Cross-organization<br>access tests pass|
|M2 — Working<br>support application|Customers, bookings,<br>tickets, tasks|Entire manual support<br>flow works|
|M3 — Searchable<br>knowledge|Upload, worker,<br>embeddings, retrieval|Sources can be<br>retrieved and cited|
|M4 — AI and<br>approvals|Investigation and<br>reviewed actions|AI cannot execute<br>unapproved mutations|
|M5 — Quality and<br>release|Evals, failure checks,<br>deployment|Deployed demo<br>passes release<br>checklist|



### **18.2 Ordered issues** 

|**Key**|**Milestone**|**Deliverable**|**Depends**|**Completion**|
|---|---|---|---|---|
||||**on**|**check**|



|OP-01|M0|Adopt guide,<br>scope,<br>templates,<br>board|—|Scope and task<br>records agree|
|---|---|---|---|---|
|OP-02|M0|Next.js/TS/<br>Tailwind<br>scaffold and<br>clean<br>README|OP-01|App runs and<br>starter clutter<br>removed|
|OP-03|M0|Lint,<br>typecheck,<br>build, test<br>harness, CI|OP-02|PR checks run<br>successfully|
|OP-04|M1|PostgreSQL,<br>Prisma, auth<br>schema, seed<br>foundation|OP-03|Fresh database<br>can migrate/seed|
|OP-05|M1|Login,<br>sessions,<br>organizations,<br>memberships|OP-04|Unauthenticated<br>and cross-<br>company<br>requests blocked|
|OP-06|M2|Customer and<br>demo booking<br>records/UI|OP-05|Scoped CRUD<br>and linked<br>booking checks|
|OP-07|M2|Tickets,<br>messages,<br>assignment,<br>status rules|OP-06|Manual ticket<br>lifecycle works|
|OP-08|M2|Tasks and<br>basic<br>dashboard|OP-07|Follow-up work<br>persists and<br>counts are real|
|OP-09|M3|Private upload/<br>storage and<br>document<br>status|OP-05|Allowed files<br>saved privately;<br>invalid ones<br>rejected|



|OP-10|M3|Redis/BullMQ<br>worker and<br>ingestion<br>pipeline|OP-09|Durable<br>extraction; retries<br>produce no<br>duplicate chunks|
|---|---|---|---|---|
|OP-11|M3|Embeddings,<br>pgvector,<br>retrieval,<br>source UI|OP-10|Organization-<br>filtered retrieval<br>passes fixtures|
|OP-12|M4|Model adapter,<br>structured<br>result schema,<br>tool contracts|OP-07,<br>OP-11|Single safe<br>model request<br>handled including<br>errors|
|OP-13|M4|Bounded agent<br>runner and<br>persisted run<br>steps|OP-12|Multi-step<br>investigation<br>stops safely and<br>cites evidence|
|OP-14|M4|Proposal<br>editing,<br>approval,<br>idempotent<br>execution|OP-08,<br>OP-13|Reject/stale/<br>double-click<br>cases pass|
|OP-15|M5|Run history,<br>redaction,<br>usage and cost<br>estimates|OP-13,<br>OP-14|UI reflects<br>recorded success<br>and failure<br>outcomes|
|OP-16|M5|AI evaluations<br>and critical<br>browser tests|OP-11,<br>OP-14,<br>OP-15|Versioned results<br>and critical flows<br>reproducible|
|OP-17|M5|Security limits<br>and<br>operational<br>failure checks|OP-16|Abuse limits,<br>isolation,<br>recovery<br>behavior checked|



|OP-18|M5|Deploy, seed|OP-17|End-to-end|
|---|---|---|---|---|
|||demo,||deployed<br>l|
|||README,||workflow|
|||release||demonstrated|
|||walkthrough|||



This backlog is dependency order, not an instruction to add queues or monitoring on day one. Split an issue when its PR becomes difficult to review. Preserve its parent task key so the plan stays understandable. 

### **18.3 A proposed 12-week schedule** 

|**Weeks**|**Focus**|**Visible result**|
|---|---|---|
|1|M0|Repository and app with CI|
|2–3|M1|Login and isolated company<br>workspaces|
|4–5|M2|Useful support app without AI|
|6–7|M3|Searchable company policies|
|8–9|M4|Investigation, drafts, and approved<br>actions|
|10|M5 evaluation|Measured behavior and failure<br>handling|
|11|Deployment|Hosted application and worker|
|12|Buffer and<br>presentation|Polished demo, documentation,<br>interview practice|



This is a planning estimate for sustained part-time/full-time learning and implementation, not a guaranteed deadline. If progress slows, remove later enhancements before weakening data isolation, approval checks, or the core demo. Keep interview preparation and job applications running alongside the project. 

## **19. Learn each concept when it becomes useful** 

|**During**|**Understand**|**Practice**|**Connection to**<br>**next step**|
|---|---|---|---|
|Foundation|TS types,<br>components,<br>HTTP, Git|Form and route<br>handler|Needed for all<br>modules|
|Identity|Cookies,<br>sessions, roles,<br>relationships|Reject<br>unauthorized<br>request|Protects<br>customer data<br>and AI tools|
|Support<br>modules|SQL joins,<br>CRUD, validation,<br>transactions|Ticket linked to<br>customer|Provides<br>trustworthy AI<br>context|
|Knowledge|Extraction,<br>chunks, vectors,<br>SQL filtering|Search a policy<br>passage|Supplies<br>grounded<br>evidence|
|AI adapter|Prompts, tokens,<br>structured output|Return<br>validated<br>investigation<br>JSON|Prepares<br>predictable UI<br>data|
|Runner|Tool requests,<br>state, stopping<br>rules|Retrieve<br>booking then<br>policy|Enables agent<br>workflow|
|Approvals|State transitions,<br>versions,<br>idempotency|Approve once,<br>reject safely|Controls<br>business<br>actions|
|Quality|Fixtures, metrics,<br>failure simulation|Run missing-<br>policy and<br>isolation cases|Makes<br>behavior<br>measurable|
|Deployment|Environments,<br>migrations,<br>workers|Deploy and<br>inspect a failed<br>job|Makes the<br>demo<br>reproducible|



For every feature, explain the UI action, request, authorization, service, database operation, and response in your own words. AI-generated code is useful only if you can inspect and explain it. Practise changing one requirement yourself, such as adding Waiting for Customer status and updating the allowed transitions. 

## **20. Testing, deployment, and release** 

### **20.1 What to test** 

|**Layer**|**Examples**|**Why**|
|---|---|---|
|Unit|Status rules, validators, chunk<br>splitting|Fast checks of<br>isolated logic|
|Integration|Tenant scope, linked records,<br>transactions, approval race|Most important<br>backend boundaries|
|Browser|Login → ticket → investigation<br>→ approval → task|Confirms the actual<br>employee flow|
|AI<br>evaluation|Citation, missing evidence,<br>unsupported fee|Assesses<br>probabilistic answer<br>quality|
|Failure<br>checks|Provider timeout, worker<br>restart, queue interruption|Checks useful<br>recovery behavior|



Use mocked model results for repeatable application tests. Run a separate explicitly configured live-model evaluation suite when assessing model/prompt quality. Do not make every pull request depend on nondeterministic paid model calls. 

CI should run lint, typecheck, appropriate tests, and build. Add database-backed integration checks as those modules arrive. Do not label a check as passed unless it actually ran. 

### **20.2 Local and deployed environments** 

|**Component**|**Local**|**Deployed**|
|---|---|---|
|Web app|Next.js development<br>server|App host|
|Database|Local PostgreSQL with<br>pgvector|Managed compatible<br>PostgreSQL|
|Queue|Local Redis when<br>introduced|Worker-compatible<br>Redis service|
|Worker|Separate local process|Persistent worker service|
|Files|Private development<br>bucket/emulator|Private object storage|
|Model API|Server-side development<br>credential|Server-side deployment<br>credential|



Environment names may include <mark>`DATABASE_URL` ,</mark> <mark>`REDIS_URL` ,</mark> <mark>`OPENAI_API_KEY` ,</mark> model IDs, auth URL/secret, and storage configuration. Exact names depend on installed libraries and deployment setup. Keep secrets out of Git and configure a separate test database. 

### **20.3 Deployment sequence** 

1. Provision compatible database, private storage, and queue services. 

2. Configure app and worker credentials and network access. 

3. Apply reviewed migrations with a deliberate deployment step. 

4. Deploy matching app/worker code. 

5. Seed synthetic demo records using an idempotent seed process. 

6. Run smoke tests for login, isolation, upload, retrieval, and investigation. 

7. Approve one task/reply and verify the resulting records. 

8. Trigger a controlled failure and verify status and recovery. 

9. Publish run instructions and limitations. 

Do not run destructive development migration commands against production. Plan database connection limits for the app and worker. 

Keep a backup/recovery procedure and a path to roll back application code; database rollback may require a forward repair migration. 

### **20.4 Release checklist** 

- Fresh install, migration, and seed instructions work. 

- Login/logout and server permissions work. 

- Two organizations cannot access each other’s data, documents, or logs. 

- Manual customer, ticket, and task workflows work with AI disabled. 

- Uploads report Ready or meaningful failure states. 

- Sources open correctly and archived content is excluded. 

- Investigation produces grounded results or explains missing evidence. 

- Rejected proposals cause no business action. 

- Stale or unauthorized approvals are blocked. 

- Duplicate execution requests produce one effect. 

- No secrets or real customer information appear in repository/demo logs. 

- Measured evaluation results include dataset/version and limitations. 

- Deployed app and worker pass the complete example flow. 

- README clearly identifies simulated bookings and internal reply publication. 

## **21. Demonstration and interview preparation** 

### **21.1 Five-minute demonstration** 

1. Show the manual support inbox and Rahul’s booking-change ticket. 

2. Open the policy in the knowledge base. 

3. Start an investigation and show observable tool steps. 

4. Explain why availability and price remain unknown. 

5. Show the cited policy, draft response, and proposed follow-up task. 

6. Edit and approve the exact task/reply. 

7. Show the resulting task/message and audit record. 

8. Show a rejected proposal or missing-policy case. 

9. Finish with actual evaluation results and known limitations. 

### **21.2 Decisions you should be able to explain** 

|**Interview question**|**Your project-specific explanation**|
|---|---|
|Why PostgreSQL?|Related business records, constraints,<br>transactions, plus vector search through an<br>extension|
|Why not separate<br>Express initially?|Next.js covers the initial HTTP/backend<br>needs; services remain separate from<br>handlers|
|How does the agent<br>access data?|Validated tools call organization-scoped<br>services|
|What is RAG here?|Retrieve passages from uploaded company<br>policies and provide them as evidence|
|What if the model<br>invents a fee?|Do not present it as verified; use evidence<br>checks and factual evaluations|
|Why a worker?|Durable long-running ingestion outside a<br>browser request|
|How do you stop<br>duplicate tasks?|Unique execution keys and transactional<br>business writes|
|How do you protect<br>companies’ data?|Verified membership and query/file/tool<br>scoping|
|Does the run log show<br>AI thoughts?|It records observed calls, evidence, and<br>outcomes, not hidden internal reasoning|
|How did you measure<br>quality?|Versioned scenarios, labeled retrieval,<br>factual review, and deterministic boundary<br>tests|



Only claim features actually implemented. If email is still simulated, say that. If evaluations found weaknesses, explain them and what you changed. 

## **22. Glossary, decisions, and references** 

### **22.1 Plain-language glossary** 

|**Term**|**Meaning in this project**|
|---|---|
|CRUD|Create, read, update, and delete/archive records|
|API|A controlled request interface to the backend|
|ORM|Library used to read/write database records|
|Migration|Versioned change to database structure|
|Session|Server-verified login state|
|RBAC|Permissions based on a member’s role|
|Multi-tenancy|Several companies sharing an app with isolated<br>data|
|LLM|Model that generates text and can request tools|
|Prompt|Instructions and context sent to the model|
|Token|Unit used to represent model input/output text|
|Tool calling|Model requests a named function; our backend<br>executes it|
|Structured<br>output|Model response constrained to an expected<br>schema|
|Embedding|Numerical text representation for similarity search|
|Vector search|Search using similarity between embeddings|
|RAG|Retrieve evidence before generating an answer|
|Chunk|A searchable passage from a document|



|Human in the<br>|Employee reviews an action before execution|
|---|---|
|loop||
|Idempotency|Repeating an execution request does not<br>duplicate its effect|
|Queue|Durable list of work waiting for a worker|
|Backoff|Delay between retry attempts|
|Observability|Records that help explain behavior and failures|
|Evaluation|Repeatable assessment against known scenarios|
|CI|Automated checks on code changes|
|Prompt injection|Untrusted content tries to redirect the model’s<br>behavior|
|Outbox|Persisted intended work that can be delivered/<br>reconciled reliably|



### **22.2 Baseline decisions** 

|**Question**|**Decision for this guide**|
|---|---|
|Product|OpsPilot AI support and operations<br>copilot|
|First demo domain|Fictional travel agency|
|Application shape|Next.js app plus separate worker<br>process|
|Database|PostgreSQL; pgvector for retrieval|
|Authentication|Better Auth with organization<br>membership|
|Agent shape|One bounded agent, initially read-only<br>tools|
|Mutation policy|Exact proposal reviewed before<br>execution|
|Support input|Manual internal tickets first|



|Customer reply|Internal ticket conversation first; email<br>later|
|---|---|
|Payments/supplier<br>booking changes|No live execution in MVP|
|Quality measurement|Versioned fixtures and actual results|
|Development workflow|Issue → branch → implementation →<br>checks → PR → merge|



### **22.3 Official references** 

These sources support technology capabilities. Product scope, permissions, tables, endpoints, and milestones are our proposed design. Library versions and hosting constraints should be checked again when implementing. 

- **S1 — Next.js Server and Client Components:** <u>https://nextjs.org/ docs/app/getting-started/server-and-client-components</u> 

- **S2 — Next.js backend-for-frontend guidance:** <u>https://nextjs.org/ docs/app/guides/backend-for-frontend</u> 

- • **S3 — Prisma PostgreSQL extensions:** <u>https://docs.prisma.io/ docs/postgres/database/postgres-extensions</u> 

- **S4 — Better Auth organization plugin:** <u>https://better-auth.com/ docs/plugins/organization</u> 

- • **S5 — OpenAI function calling:** <u>https://developers.openai.com/ api/docs/guides/function-calling</u> 

- • **S6 — OpenAI structured outputs:** <u>https:// developers.openai.com/api/docs/guides/structured-outputs</u> 

- • **S7 — OpenAI vector embeddings:** <u>https:// developers.openai.com/api/docs/guides/embeddings</u> 

- • **S8 — BullMQ idempotent jobs and retries:** <u>https:// docs.bullmq.io/patterns/idempotent-jobs</u> and https:// <u>docs.bullmq.io/guide/retrying-failing-jobs</u> 

### **22.4 First implementation action** 

Begin with **OP-01** : put this guide in the new repository, accept the first- 

release scope, create the six milestones and ordered issues, and add simple issue/PR templates. Then proceed to **OP-02** , the application scaffold. Do not implement the agent before the customer, ticket, authorization, and retrieval foundations it depends on. 

