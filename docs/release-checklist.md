# Release checklist

Draft introduced in #4. Executed in #27–#30. Source: guide §§15–16, 20–21.
All boxes begin unchecked. Add the command/test/demo evidence and date when completing each item.

## Product workflow

- [ ] A synthetic employee signs in, chooses a workspace and opens a customer ticket.
- [ ] Linked booking data belongs to the same customer and organization.
- [ ] Customer, ticket and task workflows remain usable when AI is unavailable.
- [ ] Ready policy passages support an investigation with real citations.
- [ ] A missing policy or unknown price produces an honest limitation.
- [ ] Approved internal reply and task execute once; rejected proposals do nothing.
- [ ] Stale payload/context and unauthorized resolution approval are blocked.
- [ ] Run history shows actual evidence, decisions, errors and outcomes.

## Quality and security

- [ ] Applicable lint, typecheck, build, unit, integration and critical browser checks pass.
- [ ] Organization isolation is tested across records, files, retrieval, tools, jobs and logs.
- [ ] Concurrent duplicate approvals cannot duplicate effects.
- [ ] Prompt injection in documents cannot change tool permissions or authorize writes.
- [ ] File size/type/content limits, rate limits, concurrency and AI budget caps are configured.
- [ ] Secrets are server-only, excluded from Git, and sensitive log fields are redacted.
- [ ] Versioned evaluation results show sample size, dataset version and known failures.
- [ ] Live paid evaluation remains opt-in and separate from deterministic CI.

## Deployment and operation

- [ ] Compatible app, worker, PostgreSQL/pgvector, Redis and private storage are configured.
- [ ] Fresh migration and idempotent synthetic seed succeed.
- [ ] Worker start/build commands are documented and verified independently.
- [ ] Lost enqueue, worker restart and provider timeout recover without duplicate effects.
- [ ] Backup restoration is tested in a safe environment; destructive migration rollback strategy is documented.
- [ ] Deployed smoke test covers login, isolation, upload, retrieval, investigation and approval.
- [ ] Monitoring/diagnostic instructions explain how to inspect a failed run or ingestion job.

## Handover and case study

- [ ] Repository README includes actual installation/run/check commands and environment variable names without secrets.
- [ ] Deployment runbook records chosen providers, release procedure, recovery steps and operational limits.
- [ ] Demo uses fictional customers and accurately labels internal replies as internal.
- [ ] Walkthrough includes one successful workflow and one controlled failure.
- [ ] Case study distinguishes implemented features, measured results, known limitations and future work.

Evidence record for each completed check: date, commit/deployment, method, result, and remaining limitation. Do not claim production readiness or legal compliance solely from checking this list.
