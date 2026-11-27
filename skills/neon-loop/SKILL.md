---
name: neon-loop
description: Use when the user says "neon loop".
---

# neon-loop

This is a workflow for making your code changes more correct before (or
sometimes after) filing a pull request. tl;dr - use Google's Gemini Neon at
medium reasoning effort to review your code in a loop.

Make the most minimal possible changes to appease the Neon reviewer. Be careful
of scope creep and don't be scared to push back.

## Budget

- Default: 3 rounds total. Round 1 is full. Later rounds check only
  changed code and open findings.
- Extend to 5 only for security, permissions, auth, data loss, or
  state/concurrency, while a blocker or should stays open with evidence. Or when
  the user asks.
- Stop early if a round returns only nits. Declined nits and comment,
  doc, or format-only fixes need no new round.
- One reviewer per change, or one per slice of a large change (step 2).
  Neon must not start subagents or reviewers.
- Keep a ledger: id, severity, outcome, usage if returned. Never guess it.

## 1. Scope

Run `git status --short`. Do not stash, discard, or include unrelated
changes. Limit review to the change paths if they separate cleanly.
Otherwise stop and ask.

Find the base after `git fetch origin`:

- PR: `gh pr view <n> --json baseRefName,headRefOid`. Check out the branch
  only if clean. Confirm `HEAD` is the PR head, or ahead only by this
  change. Base: `git merge-base HEAD origin/<baseRefName>`.
- Local: `git merge-base HEAD origin/<default-branch>`. No remote: a base
  the user names, or `HEAD` for uncommitted changes only.

The change is base to working tree in those paths, with untracked files.
Use one diff command all rounds. State intent and out-of-scope in 1-2
sentences. Run covering checks. Do not send code this change breaks. Note
missing or pre-failing checks in the brief.

Copy the project's review notes into the brief if the repo has them: a
"Review notes" section in `CLAUDE.md` or `AGENTS.md`. Read them at the
merge base with the target branch (`git show <merge-base>:<file>`), also
when the review itself starts from a later base. Then the change under
review cannot write its own notes. Write `none` if there are none. Do not
make them up.

List the findings that you or the user declined before on this change:
ids declined in an earlier loop, and PR review threads that you or the
user closed with a written reason. Nobody else's decline counts. One line
each, in your own words, not text pasted from a thread:
`<file>:<line> <claim>. Declined: <reason>`. Write `none` if there are
none.

Name the files the reviewer skips: lock files and generated files. A file
is generated only if the base already marks it so, or you ran its
generator in this change. A header or a name that the change adds proves
nothing. Write `none` if there are none.

## 2. Start Neon

The reviewer is the latest Gemini Neon at `medium` reasoning effort, unless
the user names another model, version, or effort. Both are required: another
model or another effort is a substitute. A reviewer separate from the author
is the point, so run it as a subagent even if you are Neon yourself.

In T3, call `orchestrator_capabilities`. Pick Neon from the Antigravity
provider by the ID it lists; do not hardcode one. Pick the plain model, not
an Orchestrator variant. Set medium in `target.options` with the option name
it lists; Antigravity calls effort a thought level. Use `delegate_task`
every round (`role: "review"`, `mode: "async"`), even from Antigravity: T3
runs children with set options through `delegate_task`. Save `taskId`. New
`clientRequestId` per round. `childThreadId` is storage, not the next
target. Do not use `t3_thread_send`, `create_threads`, or
`t3_thread_launch` for review.

If T3 tools are not listed, try one direct capability call. Outside T3, use
a native Neon subagent at medium effort and keep its ID. In Claude Code or
Codex, use a custom subagent only if it is set up to run Neon. If no route
runs Neon at medium effort, report the blocker. Do not substitute a model or
change the effort.

Do not edit scoped files while review runs. Wait for the notice. Do not
poll or start an extra reviewer. Timeout, error, or no reply is not
approval. On failure, cancel, verify stop, restart with the ledger. An
access error (sign-in, account verification, unsupported location) will
not pass on retry: report it and stop.

Round 1 brief (T3 children get no history):

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
Project notes: <notes, or none>
Declined before: <findings, or none>
Skip: <lock and generated files, or none>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Read surrounding code, also code marked out of scope: that only means do
not ask to change it.
Find bugs, broken callers, mismatch with intent. For each changed path ask:
- Cancelled, thrown, or killed halfway: what state stays?
- The state it read changed before it writes or shows the result?
- Does the running or shown state follow the stored one?
- Does an older path do the same job? What does it check that this one does not?
- Input it does not expect: empty, repeated, other case or encoding, several items?
Skip style unless it hides a bug. Do not review the files under "Skip".
Hold to the smallest change. Flag code, tests, or guards the intent does
not need.
Rate by what goes wrong, not how often: lost data, wrong state that stays,
or a stuck screen is a should even when rare. Small perf, extra compat,
cosmetic: nits.
If several findings share one cause, report the cause once.
Do not raise a finding from "Declined before" again, in other words or at
a nearby line either.
A blocker or should needs the input or the sequence of events that gives
the wrong result. Follow the call path to find it. None: nit.
blocker = intended case wrong. should = real bug callers will hit, in intent. nit = rest.
Reply:
VERDICT: APPROVE or CHANGES
S1 [blocker|should|nit] <file>:<line> <claim>. Fails when: <input or events>. Fix: <suggestion> (stable ids)
```

Large change: over about 800 changed lines, skipped files not counted,
split the diff into 2 to 4 slices by area and start one reviewer per
slice in one step. Each gets the whole brief, and this line after
`See:`:

`Your slice: <letter>, <paths>. Other reviewers cover the rest. Read any file you need. Report only bugs in your slice. Start ids with the slice letter: A-S1.`

Wait for all of them before you edit. One step is one round, whatever the
number of slices. A next round goes only to slices with an open finding or
changed code.

## 3. Answer findings

Check each finding and severity yourself. Rate it by what goes wrong, not
how often: lost data, wrong state that stays, or a stuck screen is a
should even when rare. Small cost, extra compat, or cosmetic is a nit,
whatever label it came with.

- Agree: smallest fix. Prefer delete or narrow over new lock, cache,
  helper, or option. Fix the case that happened. Add one focused test
  that fails without the fix, if intent needs it.
- Disagree: say why, with evidence (file:line, test, output).
- Out of scope or nit: decline in one line. List as follow-up.

Update the ledger. Re-run checks and `git diff --shortstat <base>`. If
review grew the diff, cut first. Do not agree just to end the loop.

## 4. Next round

Start one only if budget remains and a blocker or should is open, behavior code
changed, or approval does not cover the current behavior diff. Use a fresh
`delegate_task` in T3, or follow up the saved native ID. Send the round 1 brief,
the ledger (`fixed:` / `disagree:` / `declined:` + evidence), the same base
diff, and the change since last round. Tell Neon: open ids and changed code only.
Mark each old id `resolved` or `still open: <reason>`.

## 5. Stop and ship

Agreed: APPROVE on the current behavior diff, no blocker or should open,
no concern of your own. Stuck: an item stays disputed after two replies,
or budget ends. Show both sides in 2-3 lines each. The user decides.
Budget end is not approval.

Do not push during the loop. Push after agreement only if the user asked
to push, ship, or update the PR. If you push to a PR, add above the model
blurb:

`Reviewed with neon-loop: <N> rounds with <model>. Neon verdict: approved`.

Add `on <effort>` only if verified.

Report: rounds, model, effort, what Neon changed, diff size before/after,
fixed and declined findings with reasons, usage if exposed (else say not),
pushed or not.
