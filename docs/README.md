# OpsPilot documentation — start here

Prepared 11 October 2026 from Rohit's supplied OpsPilot AI Project and Implementation Guide v1.0 and the existing GitHub issues #1–#30.

## What to follow

Keep the existing issue numbers, titles, milestones, and sequence. Read only the documents linked to the current issue before implementing it. These files are prepared specifications, not evidence that features have been built or issues completed.

The original [project guide](reference/project-guide.pdf) is preserved unchanged. It remains the product reference. The user's chosen workflow is now to maintain separate, explained documents aligned with existing issues; this supersedes the original guide's suggestion to use one document only. The guide's OP-01–OP-18 keys are not GitHub issue numbers.

| Read order | Document | Purpose | Introduce with issue |
|---|---|---|---|
| 1 | [MVP requirements](mvp-requirements.md) | What version 1 includes and excludes | #1 |
| 2 | [Architecture](architecture.md) | How components connect and why each exists | #2 |
| 3 | [Database design](database-design.md) | Records, relationships, constraints, and migration stages | #3 |
| 4 | [Implementation plan](implementation-plan.md) | Exact existing issue order, dependencies, and missing-scope assignments | #4 |
| 5 | [GitHub workflow](github-workflow.md) | Branch, implementation, verification, PR, merge, and cleanup | #4 |
| As needed | [Issue playbooks](issues/README.md) | Purpose, steps, exclusions, and acceptance criteria for all 30 issues | #4 |
| Before release | [Release checklist](release-checklist.md) | Security, deployment, recovery, and demonstration gates | #4 draft; complete #27–#30 |

## Using this pack without mixing issues

Extract it outside your repository first. Copy the reference PDF, this index, and `mvp-requirements.md` in issue #1. Copy `architecture.md` in #2 and `database-design.md` in #3. Copy the implementation plan, workflow, playbooks, and draft release checklist in #4. Stage exact files for each PR; do not commit all prepared documents under #1.

The top-level repository README remains the installation/run entry point; do not replace it with this docs index. No source application or existing GitHub issue has been changed by generating this pack.

## Maintaining accuracy

Product scope belongs in MVP requirements; component responsibilities in architecture; constraints in database design; issue sequencing in the implementation plan. The playbooks reference these decisions rather than overriding them. When a requirement changes, update the owner document and affected issue/playbook in the same PR. Do not let the PDF and current decisions silently disagree: record a dated change and why it was made.

Concrete defaults not specified by the reference are labelled **Proposed implementation detail**. Review them in the assigned issue before coding. Package versions, model IDs, provider prices, and auth APIs must be verified when installed; this pack does not claim current vendor compatibility.
