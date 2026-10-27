# Code review loop

Claude Code's bundled `/code-review` reviews a diff with several agents and verifies each finding before it reports. In Claude Code, run this loop before every push, so most bugs are fixed before a hosted reviewer or a person reads the PR. Cursor has no `/code-review`. There, Bugbot fills this role and the loop does not apply.

The Opening a PR playbook (`../playbooks/opening-a-pr.md`) runs the loop before the push that opens a PR. The Babysit playbook (`../playbooks/babysit.md`) runs it before every later push that changes the PR's own diff. Run it on the final diff: after `/deslop` and `/no-comments`, before `interrogate` and the push.

1. **Commit first.** A pass reviews commits, so its record names an exact SHA.
2. **Pick the range.** The first pass reviews the whole PR with `/code-review <level> <base>...HEAD`, where `<base>` is the freshly fetched base branch: trunk for a root PR, the parent branch for a stacked one. Each later pass reviews only the commits since the last reviewed SHA, with `/code-review <level> <last-reviewed-sha>..HEAD`, so it does not re-report what an earlier pass raised. The last reviewed SHA is the newest SHA in the record. A clean merge of the base needs no pass, but add it to the record so the next range starts after it and leaves out the base's changes. Review the whole PR again after you resolve merge conflicts by hand, or when the last reviewed SHA is no longer an ancestor of `HEAD`, as after a rebase.
3. **Pick the level.** Each path the range touches takes the level that the repository's instruction files, or the review guide they link, name for it, or `high` when they name none. The highest of those levels wins. Always type the level, because a bare `/code-review` reuses the last level typed, even from another session. Use `max` only when the user or the repository's instructions ask for it.
4. **Fix, then review the fixes.** Verify each finding against the code before you act on it, with the same skepticism as `bugbot-triage.md`. Fix each real finding before the push. When a finding names a class of bug, fix every instance in the diff, not only the flagged line. Commit the fixes and review those commits with the next pass. For a finding you reject, write down why.
5. **Stop at three passes.** If the third pass over the same change still finds new problems, do not run a fourth. Re-evaluate the approach, add a stop line to the record, and tell the user. The PR is not merge-ready until the user decides and you replace the stop line with their decision.
6. **Keep one record.** Until the PR exists, keep the record in `$(git rev-parse --git-dir)/pstack-code-review.md`, so it survives a new session or a compacted context in the same checkout. When you open the PR, post the record as one comment through the resolved forge, and edit that comment after each later pass. Do not copy pass counts into the PR description, where they go stale.

   ```text
   <!-- pstack-code-review:v1 -->
   Claude /code-review passes:
   - <level> over <range> at <head-sha>: <n> found, <n> fixed, <n> rejected, <n> open
     - Rejected <path:line>: <reason>
   - Clean merge of <base> at <merge-sha>: no pass needed
   - Stopped after three passes at <head-sha>: waiting on the user
   ```

   A finding that stays real but unfixed is open. Post each open finding as an inline review thread with the reason it stays. Only the user resolves that thread, so the watcher holds `READY` until they do. Post it through the resolved forge yourself. `--comment` would post every finding of the pass, including the ones you fix.

If the session cannot run `/code-review`, record that the pass was skipped and why. Never record a pass that did not run.

## Merge readiness

Before you report a PR merge-ready in Claude Code, confirm:

- The last pass in the record covers the PR's head commit, or every later commit is a clean merge of the base that the record lists.
- The user has resolved every open finding's thread, and the record has no stop line waiting on them.

A clean pass is not evidence that tests ran. CI remains the validation record.
