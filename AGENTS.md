# Repository Collaboration Protocol

These instructions are mandatory for Codex and other agents working in this repository. They apply to the entire repository tree unless a more specific `AGENTS.md` exists in a subdirectory.

## Source of truth

- GitHub is the shared source of truth.
- Before starting a task, read the complete issue and its comments, the related canonical documents, and the current state of `main`.
- Run `git pull --ff-only` before creating the task branch.
- Review `git status` and preserve any existing user changes.
- Do not use ZIP files or external copies as a source when GitHub contains a newer version.
- Do not duplicate documents or rules that already exist.
- If two sources contradict each other, stop and report the contradiction before editing.

## Branch workflow

- Never work directly on `main`.
- Use one branch per issue:
  - Documentation: `docs/issue-<number>-<short-name>`
  - Feature: `feat/issue-<number>-<short-name>`
  - Fix: `fix/issue-<number>-<short-name>`
- Every commit must remain within the issue scope.
- Do not mix unrelated tasks in the same branch or pull request.

## Pull request workflow

When a task is finished:

1. Run every required validation.
2. Review the complete diff.
3. Confirm that no secrets, temporary files, or unrelated changes are included.
4. Create a descriptive commit.
5. Push the branch to GitHub.
6. Create or update a Draft Pull Request.
7. Include `Closes #<issue-number>` in the pull request description.
8. Do not close the issue manually before merge.
9. Do not report the task as completed when the changes exist only locally.

## Mandatory PR handoff

The pull request description must always contain these sections.

### Summary

Describe what was implemented and the problem it solves.

### Source documents reviewed

List the issues and canonical documents reviewed before editing.

### Files changed

List every file created, modified, or deleted and explain why it changed.

### Decisions preserved

Describe the domain and architecture decisions that remain unchanged.

### Validation

List every command executed and the result of each validation.

### Open questions

List pending decisions, assumptions, or risks that were not resolved silently.

### Out of scope

List the items intentionally left unchanged.

### ChatGPT Work handoff

Provide a self-contained summary that allows ChatGPT Work to continue without reconstructing the work from scratch. It must include:

- repository;
- issue number;
- branch name;
- pull request number and URL;
- latest commit SHA;
- validation status;
- changed files;
- relevant decisions;
- open questions;
- recommended next step.

## Issue synchronization

After publishing the pull request, add a comment to the issue containing:

- the pull request link;
- status: `Ready for review`;
- a brief summary;
- validations executed;
- open questions.

Do not publish duplicate comments. If the pull request already exists, update the existing comment when possible or add only new information.

## Final Codex response

Codex must always finish repository tasks with this exact structure:

```text
TASK HANDOFF

Repository:
Issue:
Branch:
Pull request:
Latest commit:
Status:
Files changed:
Validations:
Open questions:
Next recommended action:
```

A task may be reported as `Ready for review` only after its branch and pull request are published on GitHub.

If pushing, creating the pull request, or commenting on the issue is not possible, report `Blocked` and state exactly which operation remains. Never claim that GitHub is updated when it is not.

## Documentation rules

- Write canonical repository documentation in English.
- Follow `docs/domain/UBIQUITOUS_LANGUAGE.md`.
- Link specialized sources instead of copying their content.
- Never translate technical failures into credit outcomes.
- Keep `Favorable` and `Unfavorable` as the only credit outcomes.
- Keep `PendingEvidence`, `PendingRetry`, and `OperationalException` as operational dispositions.
- Preserve this sequence: `Signed Package -> Loan Reservation (PendingDisbursement) -> Disbursement -> Loan Activation`.
- Do not use “Application Approved” as a canonical term.
- Do not create implementation repositories before their entry criteria are satisfied.
