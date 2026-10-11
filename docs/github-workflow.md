# GitHub implementation workflow

Introduced with #4; used for every issue. Based on guide §17 and the existing repository workflow.

## 1. Know what each item does

A milestone groups related outcomes. An issue states one piece of work. A branch isolates its changes. A commit records a meaningful change. A PR shows the proposed difference and verification. Merging brings the reviewed change into main. The board shows state, not proof of implementation.

Board: Backlog → Ready → In progress → In review → Done. Mark blockers explicitly.

## 2. Before coding

Open the matching [playbook](issues/README.md), read linked documents, check prerequisites and choose its suggested branch. Set the issue In progress. Update the issue body with the playbook's scope and measurable checks when you start; do not change its number or title just to match the guide's OP keys.

In PowerShell, first inspect your working tree:

```powershell
cd D:\opspilot-ai
git status
```

If there are changes, finish or preserve them before switching branches. Do not discard unknown work. Once clean, for Issue #1:

```powershell
git switch main
git pull --ff-only origin main
git switch -c docs/issue-01
```

If already working on the earlier suggested `docs/issue-1-mvp-requirements`, continue that branch; do not create a second branch for the same task. Substitute the playbook branch for later issues.

## 3. Explain and implement

For each feature explain: employee action → request → authorization → service → database write/read → response. Add one coherent scope, including necessary validation/error behavior and focused tests. Keep future integrations out of the PR.

Prepared documentation is introduced in separate changes:

| Issue | Stage these paths |
|---|---|
| #1 | `docs/README.md`, `docs/reference/project-guide.pdf`, `docs/mvp-requirements.md` |
| #2 | `docs/architecture.md` |
| #3 | `docs/database-design.md` |
| #4 | `docs/implementation-plan.md`, `docs/github-workflow.md`, `docs/issues/`, `docs/release-checklist.md` |

The #1 index intentionally lists upcoming documentation; its later links become available as #2–#4 merge. The delivered complete pack has all targets. This staging sequence avoids committing all issue deliverables under #1.

## 4. Verify and commit

Read the diff and run `git diff --check`. Documentation review checks scope, consistency and links. Before scaffold/tooling exists, mark application lint/typecheck/build/tests N/A with the reason. Once installed, run the actual scripts in package.json and relevant feature tests; do not invent passing checks.

Example for #1 after copying its three paths:

```powershell
git diff --check
git add docs/README.md docs/reference/project-guide.pdf docs/mvp-requirements.md
git diff --cached --stat
git commit -m "docs: define MVP requirements"
git push -u origin HEAD
```

## 5. Pull request and merge

Open a PR into main. Explain the problem, result, changed files, checks actually run and limitations. Include `Closes #1` for Issue #1, replacing the number for subsequent issues. Review the diff and CI; solo review is self-review, not independent team approval. Merge only when applicable checks pass and the issue's criteria are met.

Update the board to Done after the merge. Confirm the linked issue closes. Then, once your working tree is clean:

```powershell
git switch main
git pull --ff-only origin main
```

Delete the completed local branch only after confirming its work is merged. Git's safe `git branch -d <branch>` may refuse after a squash merge; inspect before taking any stronger action. Start the next issue from updated main.

## 6. Keep documentation attached to implementation

Update relevant schemas, API decisions and setup commands in the same PR as the feature. Record model IDs, limits and provider decisions when selected. Preserve the PDF as the original reference; record deliberate deviations with date, reason and issue number in the owning document. Never mark a feature complete because its playbook has been written.
