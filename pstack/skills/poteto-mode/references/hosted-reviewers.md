# Hosted reviewers

A hosted reviewer is a GitHub app that reviews a PR on its own infrastructure, such as Bugbot or Codex. Treat a reviewer as active on a PR when it has posted there, or when the repository's instruction files or review guide say it is enabled. Never request a pass from a reviewer that is not active, and never treat an inactive reviewer's silence as a missing review.

The watcher knows Bugbot natively. For any other active reviewer, its adapter below says what the watcher misses and how to fill the gap.

## Triage

Triage every hosted finding with `bugbot-triage.md`, whichever reviewer raised it. Verify each claim against callers and contracts before changing code. Start each reply with one word for how you handled the finding: `Fixed`, `Rejected`, `Intentional`, or `Deferred`, so a later look at the review setup can count dispositions.

Count passes per reviewer for the rubric's third-pass rule, using `bugbotReviewPasses` where the watcher reports it and the adapter's own count otherwise.

## Re-requesting a pass

Hosted passes cost plan usage, so request them deliberately:

- After pushing a fix for a hosted finding, make sure that reviewer covers the new head: wait for its automatic pass if it re-reviews on push, otherwise request one.
- In Claude Code, do not request a hosted pass for a merge of the base, a docs-only follow-up, or a push that touches no file a hosted reviewer flagged. The code-review loop (`code-review-loop.md`) covers those pushes, including a merge whose conflicts you resolved.
- Request a security-specific pass only when the user asks for one.

## Merge readiness

Before you report a PR merge-ready, confirm that every active reviewer's latest pass has finished and none of its finding threads stay unresolved. If you pushed a fix for one of its findings, that reviewer's latest pass covers the fix: the fix commit is an ancestor of the head that pass reviewed. Otherwise, in Claude Code, a hosted pass does not need to cover the current head, because the code-review loop's record covers each later push. Without that record, the latest pass must cover the current head.

Unresolved threads already block the watcher's `READY`, whoever wrote them. The other signals are covered only where an adapter says so.

## Reviewer contract

Each adapter answers:

- **Identity:** the GitHub account and any check run it posts.
- **Running and done:** how to tell a pass is still running, and which head commit a finished pass covered.
- **Findings:** where they appear and how severity is marked.
- **Request:** what triggers a pass and how to ask for one.
- **Cost:** what each pass draws on.
- **Watcher:** what the watcher already sees for it.

## Adapters

### Bugbot

- **Identity:** Cursor's GitHub app, `cursor[bot]` (GraphQL login `cursor`). It posts a check run whose name contains `Bugbot`.
- **Running and done:** the check run's status and conclusion on the head SHA.
- **Findings:** inline review threads with a severity line.
- **Request:** runs on PR updates when automatic review is on. Ask for a pass with a `bugbot run` comment.
- **Cost:** the owner's Cursor plan.
- **Watcher:** native. The watcher marks its threads `isBugbot`, counts `bugbotReviewPasses`, and reports `reviewAutomationRunning` while its check is pending, so no gap coverage is needed.

### Codex

- **Identity:** `chatgpt-codex-connector[bot]` (GraphQL login `chatgpt-codex-connector`). It posts no check run.
- **Running and done:** one summary comment per PR starts with `<!-- codex-pull-request-review-summary -->`. Its `<!-- codex-review-state:v1 {...} -->` line gives `code_review.status` (`completed` once a pass finishes), `code_review.commit_sha` (the head that pass reviewed), and `code_review.has_unresolved_findings`. `security_review` has the same fields and stays `null` until requested. A 👀 reaction from the bot means a pass is running; 👍 means every pass finished with no findings.
- **Findings:** inline review threads with a priority badge, where P0 is the most severe. Triage every level; a lower badge is not a dismissal.
- **Request:** automatic on PR open and on draft-to-ready; it does not re-review on push. Ask for a pass with an `@codex review` comment, in the focused form the repository's review guide gives when it has one, or for a security pass with `@codex security review`.
- **Cost:** the owner's ChatGPT Codex plan usage.
- **Watcher:** sees only its unresolved threads. `isBugbot` stays `false`, `bugbotReviewPasses` stays `0`, and `reviewAutomationRunning` never turns on. Count passes as the distinct `commit_sha` values the summary comment has reported, and read running state from the summary comment.
