# Issue #4 — Create implementation roadmap

GitHub: https://github.com/rohitx-dev/opspilot-ai/issues/4

Milestone: M0 - Foundation

Prerequisites: #1–#3

Suggested branch: `docs/issue-04`

Reference: original guide §§17–20; [supporting design](../implementation-plan.md).

## Purpose and expected result

Adopt this exact 30-issue roadmap. This issue contributes the next working or reviewed piece while retaining the product boundaries in [MVP requirements](../mvp-requirements.md).

## Implementation sequence

1. Adopt this exact 30-issue roadmap.
2. Review additional scope assignments and OP-key mapping.
3. Add workflow, playbooks and draft release checklist.

## Outside this issue

Renumbering issues or marking future work complete.

## Acceptance and verification

- [ ] All 30 existing titles and milestones are retained.
- [ ] Booking, task, worker, CI and evaluation work have owners.
- [ ] Each issue has meaningful completion checks.

For implementation issues, add focused tests with the feature and run the available lint/typecheck/build and relevant test commands. For documentation issues before scaffolding, review consistency, references and `git diff --check`; mark application checks N/A, never passed. Record actual commands/results in the PR.

## Finish and handoff

Self-review the changed files, commit only this issue’s scope, push `docs/issue-04`, and open a PR containing `Closes #4`. Keep acceptance boxes unchecked until verified. Follow the [GitHub workflow](../github-workflow.md) for merge and cleanup.

Next: [Issue #5](issue-05.md).
