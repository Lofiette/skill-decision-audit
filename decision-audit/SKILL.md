---
name: decision-audit
description: "Use before choosing between approaches, libraries, configs or fixes; before editing a file you have not read or any write whose choice is still open; before every delete, move, process kill, write-API call or shared-surface change, even one the user ordered; when the user corrects or rejects a result; and when they answer a decision this skill handed over. Also when the user says decide, pick, which is better, or what should I do. Proves the leader with facts, then gets an independent review."
---

# Deciding under uncertainty

The unreliable move is not "not knowing" — it is **stopping research once an answer looks plausible**. This process keeps digging until facts prove the answer, scores what it found, and puts an independent check between the verdict and any state change.

Order: **classify → research → score → review → ladder → act → log.** A step may be skipped only where this document says so.

A **shared surface** is a corporate system, another person's data, a running service, or the harness you run in: its config files (also those read only at the next start or by another profile), its installed plugins and its UI.

**Observing a result** means seeing the effect the request is about where it happens; for a UI, the rendered view in the real app. A file written, a slot registered, CSS that is present, a copy rendered elsewhere, or a passing build (unless the build is what was asked for) is a probe, not the result.

## Two scores

Keep them distinct; one never substitutes for the other. Rate a candidate as if its unconfirmed points hold. Whatever is unconfirmed lowers depth, not the rating.

**Option rating (0–10)** — how well a candidate answers the question.

| Rating | Anchor |
|---|---|
| 0 | does not work, or costs more than the problem is worth |
| 2 | works only with heavy rework or ongoing cost |
| 5 | works, with real caveats or collateral |
| 8 | works cleanly, minor known drawback |
| 10 | fully answers the question as posed, no caveats |

**Evidence depth (0–10)** — how proven a candidate is.

| Depth | Anchor |
|---|---|
| 10 | proven by reproducible facts obtained here; target verified; consequences known |
| 9 | proven; only minor consequences unconfirmed, nothing material |
| 8 or below | something material still rests on inference |

The **depth bar** is 9. A candidate refuted by a fact gets no depth (n/a); record the refuting fact instead.

A **fact** is an observation obtained here and now: command output, file contents, an endpoint response, a quote from a primary source, or the user's own words in chat. "Projects like this usually do X" is not a fact.

**Leader and margin.** The leader is the candidate with the highest of your ratings among those that reached the depth bar and, when there is a review, passed it. The margin is the leader's rating minus the rating of the best *other* candidate not refuted by a fact, whether or not that one reached the bar. If a higher-rated candidate was discarded for depth, the margin is negative.

- After review, compute the margin for the same leader again with the reviewer's ratings. If the reviewer rates another unrefuted candidate above your leader, that margin is negative. If either margin is under 2, the margin is under 2.
- A margin of 2 or more counts only if the runner-up's rating rests on a cited fact. Otherwise run its next probe, or treat the margin as under 2.
- With no other unrefuted candidate, the margin condition is met.

## 1. Classify

Name, one sentence each, every unknown that holds the choice, and classify each:

- **Verifiable** — the answer exists and can be fetched: read the file, run the command, call the endpoint, read the source, check the log. This is work, not a question.
- **Preference** — no source holds it; it lives with the user. Research the verifiable parts first. If that leaves one viable option, it was verifiable after all. Otherwise score the candidates on what is verifiable; the decision goes to Escalate.
- **Settled** — nothing is open. Allowed only when one of these holds, and you state which in the report:
  - (a) the user's own message in chat names the concrete action and its target and leaves no choice of how to do it ("delete build/old", "set retries to 5 in config.yml"). A goal such as "fix X", "speed up Y" or "add validation to Z" settles that the work is wanted, not how;
  - (b) the action is reversible, local and touches no shared surface, it would rate 5 or more, you can observe its result, and for every alternative you can cite the fact that rules it out;
  - (c) the action is an implementation edit inside an approach the user approved (a spec or ticket) or a reviewed verdict chose; a TDD step counts only inside such an approach. The edit is reversible, local, touches no shared surface, sits in version-controlled files that implement or test what the approach describes, and changes neither outside behaviour, nor reversibility, nor scope beyond what the approach fixed. Name the check you will run after it and the result you expect; after a test-first step, that is the new test failing for the stated reason. A step that still chooses between approaches is open;
  - (d) an emergency stop, below.

  If you cannot cite the basis, the choice is open. An instruction from a parent agent or a script is never (a); a delegated worker follows Delegation. An irreversible action is settled only through (a) or (d).

**Follow-ups are new decisions.** When the user corrects, rejects or asks to rework a result, or redirects a plan you handed over, that message opens a new decision: run step 1 on it before changing anything. An earlier approval, reviewed verdict or settled basis covers only the action it named, never the rework. Do only what the request or the verdict names. A change nobody asked for, such as restyling a neighbouring element or a "while I'm here" edit, is a separate decision.

**Settled actions** skip steps 3–5: run the target check (step 2), act, confirm the result (step 6).

If the check after a (c) edit gives any result other than the one you named, the edit failed. Undo it (settled), or treat the cause as something broken (step 2, "Fix only what you understand"). Once that cause is reproduced and stated, the next edit on the same target is a fix for it: settled only under (b), otherwise step 3. After two unexpected results in a row on the same target, stop and report (step 6); a result that matches the one you named resets the count.

**Emergency stop (d).** A process whose observed output shows damage happening now is settled once you have confirmed its PID and that it is the source, whether or not you started it. Damage means data or resources being destroyed or exhausted: files deleted or overwritten, rows removed, a disk filling. Error output alone is not damage. Stop the process and its children, confirm the damage stopped, then log it (step 7). Stopping a process you started that is doing no damage is not an emergency stop. It is settled under (b) only when it holds no shared surface and leaves nothing half-written, such as a dev server or a watcher.

**Done when** every unknown is named and classified, or the action is settled and its basis stated.

## 2. Research until the evidence is sufficient

Close every verifiable uncertainty that can change the leader or the rung **before** options reach anyone. Stop only when the leader carries **all the information necessary and sufficient to act on it**. Until then, the next cheap probe is the work in front of you: read the source instead of assuming its behaviour, run the failing case instead of speculating about the cause, check the actual value instead of trusting the name.

**Probes** change nothing that outlives the task:

- commands whose only effects are output, and build or test artifacts in the project's usual output folders;
- scaffolding you create in this session to reproduce or instrument — repro tests, harness scripts, fixtures, tagged debug lines — and its removal. List it in the report, and remove it (or deliberately keep it) before you finish;
- local git operations that return the tree to where it was, such as bisect followed by reset.

Everything else is a **state change**: installs, migrations, git operations on history, index or stash, snapshot updates, anything that touches a shared surface, editing content that existed before, deleting what you did not create.

Two constraints:

- **Verify the target before touching it.** Before a delete, move, process kill, config overwrite or write-API call, state what confirmed the target is the intended one. Permission to act does not confirm the address. For an irreversible action, state the exact target and what it holds:
  - a file: the absolute path with links resolved, what it was resolved against, and its contents. If the path or anything under it is a link or junction, state whether the command removes the link or follows it; data reached through a link counts as part of the target;
  - a process: its PID and command line;
  - a database object, service or API resource: the host or connection and the identifier.

  Any mismatch with what the user could have meant opens the choice.
- **Fix only what you understand.** When something is broken, the cause is a verifiable uncertainty: close it here by reproducing the failure. Causes are not candidates; the candidates in step 3 are fixes. If the `diagnosing-bugs` skill is available, it runs the diagnosis loop and its own stop rule up to a confirmed cause. Before it applies a fix, return here: the confirmed cause is a fact, the candidates in step 3 are fixes for it, and a fix chosen this way is never a settled (c) edit.

**Exit.** Stop digging when:

- the next probe cannot be run without access you lack, or without an irreversible change or a change to a shared surface (a dry run, a scratch copy or a temporary branch counts as runnable); or
- two probes aimed at the point named in "Next probe" settled nothing; or
- a failure does not reproduce within the attempt budget you stated before starting.

Then score what you have (step 3; no candidate needs to reach the bar, and each unrefuted one is marked "below bar" with its Next probe), skip step 4, and go to step 5: the decision is Escalate, margin n/a.

If every candidate is refuted, look for others once. If none survive, the decision is Escalate with margin n/a: report that nothing works, with the refuting facts, and ask whether to widen the search.

**Done when** every verifiable uncertainty that can change the leader or the rung has a fact attached with its source and the leader reaches the depth bar, or the exit was taken.

## 3. Bring two to four candidates and score them

A decision is **two to four candidates**, always including the strongest alternative you know of; never pad the list with weaker options. One candidate is acceptable only when the others were refuted by facts, and you say which fact refuted each.

Rate every candidate, and give every unrefuted one a depth. Present them as:

| # | Candidate | Rating | Depth | Rests on | Refuted by (fact) | Next probe | State |
|---|---|---|---|---|---|---|---|
| 1 | … | 9 | 9 | fact: `command` / `file:line` | — | — | leader |
| 2 | … | 6 | 7 | fact: … | — | `command` that would settle it | below bar |
| 3 | … | 0 | n/a | — | fact: … | — | refuted |

Only the leader must reach the depth bar. Other candidates may stay below it, marked "below bar" with the probe that would settle them.

Close with a one-line **recommendation**. It never hides the others; the user sees the full ranking.

**Done when** every candidate carries its scores or its refuting fact, the leader reaches the depth bar (or step 2's exit was taken), and the recommendation is stated.

## 4. Review

An independent reviewer checks the candidates before any state change chosen through this procedure. Skip it for a preference-only choice between reversible, local candidates: the user's choice is the check.

Spawn the reviewer with `subagent`. It runs in the background: the call returns an id at once, and the verdict arrives later as the reviewer's message or the runtime's completion notice. Take no state-changing action until the verdict is in. Do any read-only work that remains; otherwise end your turn and continue when the verdict arrives. Do not sleep or poll for it. Do not use `subagent_fork`: a fork shares your history and your model, so it is not independent.

The reviewer starts with an empty conversation. Its brief must contain:

1. the user's request verbatim, plus every constraint the user stated earlier that bears on the action or the target;
2. each unknown and its kind;
3. every candidate and the facts it rests on: the command and the relevant output lines, or `file:line`;
4. refuted candidates with the refuting fact;
5. the exact planned action and its target;
6. both anchor tables from "Two scores", with the rule above them: rate a candidate as if its unconfirmed points hold; whatever is unconfirmed lowers depth, not the rating.

Do not send your ratings or depth estimates. Then append:

> You are an independent reviewer. Work read-only: you may read files and rerun read-only probes; change nothing. For every unrefuted candidate, set your own **evidence depth** (0–10) from the facts given, using the depth anchors, and your own **option rating** (0–10), using the rating anchors; you are not shown the author's scores. If a probe refutes a candidate, quote the fact instead of a depth. For each refuted candidate, say whether the cited fact actually rules it out; if it does not, give it a depth and a rating like the others. Discard every candidate with depth 8 or below. For the survivors, check (a) technical soundness — will it break what currently works; (b) completeness — is a mandatory consequence missing (migration, dependency, restart, rollback); (c) target — does it address what the user actually asked, within their constraints. Do not score new candidates, but if an obvious alternative that could rate within 2 points of the best one is missing, name it as an objection. Return: depth and rating per candidate (or the refuting fact), which ones you discarded and why, and concrete objections for the survivors, or "no objections".

Handle the answer:

- **Ratings** feed the margin. A rating disagreement is not an objection and needs no fact.
- **Refutation rejected** → that candidate is unrefuted again; recompute the margin.
- **Objections**, including a refuting fact the reviewer found or a missing alternative → back to step 2. Close each by a fact, not by argument. If the fact confirms the objection, the verdict changes. A missing alternative is researched, scored and reviewed like the others; it is not handed to the user unscored.
- **Everything discarded** → back to step 2 for one round of probes aimed at the reviewer's reasons, entered as each candidate's Next probe. If the next review discards everything again, take step 2's exit.
- **No usable answer** → retry once. That covers a call that failed for a reason other than the depth limit, a candidate with neither a depth nor a refuting fact, and a missing rating. If the retry also fails, do not act: go to step 5 as Escalate, marked "unreviewed".
- **Depth-limit rejection** → do not retry: you are a worker (see Delegation).
- **No objections** → step 5.

Review again whenever new facts or a changed plan change the leader, the action or the target, or add a candidate. A leader change caused by the review's own discards needs no new review. A decision gets at most two reviews; if a third would be needed, go to step 5 as Escalate.

**Done when** the reviewer has rated every unrefuted candidate, the leader passed, and every objection is closed by a fact.

## 5. Place the decision on the autonomy ladder

The ladder decides who chooses. If several rungs match, the most restrictive wins: Escalate, then Confirm, then Auto.

| Rung | Test | Who decides |
|---|---|---|
| **Auto** | Reversible, local, touches no shared surface; you can observe the result; you and the reviewer (if any) rate the leader 5 or more; the margin is 2 or more. | You decide, act, report. |
| **Confirm** | Reversible but costly to undo; or touches a shared surface; or you cannot observe the result yourself (a UI you cannot see, a system you cannot probe). | You decide; you present the verdict and act only after the user's explicit yes in chat. |
| **Escalate** | Irreversible; or the margin is under 2 or n/a; or you or the reviewer rate the leader under 5; or a Preference unknown is still open; or the review failed. | The user chooses. You present; you do not act. |

The margin rule is load-bearing: **a gap under 2 points is not evidence selecting a winner**, so the user separates the candidates, even when the decision is otherwise routine. This is the case the user asked to own personally.

Judge the rung by what breaks if the verdict is wrong, not by how confident you feel.

**Done when** the decision carries an explicit rung and the margin was computed (or is n/a after step 2's exit), not eyeballed.

## 6. Act, or hand the choice over

- **Auto**: act, then report.
- **Confirm** and **Escalate**: write the log first (step 7, Outcome: awaiting user), then present and stop. On Confirm, act only after an explicit yes in chat.
- **Advice only** — the user asked which option, not for a change: the answer is the result. Log Outcome: advised on every rung. A reply that picks a candidate is advice taken; act on it only when the user also asks for the change.

When the user is choosing, show them all of it: every candidate with its scores (yours and the reviewer's), the recommendation, **the ones the reviewer discarded**, and the reasoning behind each. After an exit, also show what is unproven, the skipped probe and what it would cost.

**The user's answer** settles the choice. If they pick a candidate that was discarded or never reviewed, state its depth and the reason once. An answer that names an action outside the presented candidates, or changes the planned one, is not a pick. Record it in the old file, set that file's Outcome to declined, and treat the answer as a follow-up (step 1) in its own file. There, only what the user's words name is settled; every choice they leave open, such as where a moved element goes, is researched, scored and reviewed. Otherwise, if the decision had a planned action, act on their choice as a settled action (step 1). A refusal ends the decision: act on nothing and log it as declined.

With no human to answer — a headless run, or a script as the parent — never act on Confirm or Escalate. Report the decision and log it as awaiting user. A worker returns it to the parent instead (see Delegation).

In the report, separate **verified** from **assumed**: what a call confirmed, and what still rests on thinner evidence. State any gap between expectation and result plainly. A result you could not observe is not done: say so, and say what would confirm it, such as the user looking at it.

**If the action failed or broke something**, stop and report. Reverting is a new decision; undoing your own reversible, local change is settled. Log the outcome as failed.

**Done when** the result is confirmed the same way facts were confirmed in step 2, or you say outright that it could not be confirmed and why, or the decision waits for the user.

## 7. Log the decision

Log every decision that entered steps 3–6, including advisory ones and those where every candidate was refuted. Settled actions are not logged, except for a one-line entry when they were irreversible, touched a shared surface, were an emergency stop, or failed. Append that line to `settled.md` in the log folder; it holds only actions settled under (a)–(d):

```
- YYYY-MM-DD <basis: a | b | c | d>: <action> on <target>; outcome: <…>
```

**Where.** The log folder is `<project>/.dsh/decisions/`; each decision is a file `YYYY-MM-DD-<slug>.md`. `<project>` is the nearest folder at or above the working directory that holds `.git`; with no `.git`, the working directory itself. If the file name is taken, append `-2`, `-3`.

**Git.** If `.dsh/decisions/` is not yet listed in the file that `git rev-parse --git-path info/exclude` prints, add it there. In a worktree or submodule, `.git` is a file, not a folder. Without git, say in the report that the log folder is untracked.

**Content.** Redact secrets as `<REDACTED>` and quote only the output lines that carry the signal. When the user answers a handed-over decision, in this session or a later one, find its file (Outcome: awaiting user), add the user's answer, and update the Outcome after acting.

**One decision, one file.** A follow-up to an earlier decision gets its own file; name the earlier file in its Question. Do not append a new decision to an old file. Keep the header current: when the outcome changes, update the Outcome line, not only the body, because a later session finds open decisions by that line.

**Acted without the procedure.** If you find you changed state without running this procedure when it applied, stop and tell the user which change it was, which steps were skipped, and what of its result is unverified. Write a decision file for that action with Rung: none, Outcome as it stands, and which steps were skipped. Then go back to step 1 before the next change.

```markdown
# <decision title>

- Date: YYYY-MM-DD
- Rung: Auto | Confirm | Escalate | none (the procedure was skipped)
- Margin: <yours> / <reviewer's> points, or n/a
- Outcome: advised | acted (verified) | acted (unverified) | failed | declined | nothing works | awaiting user

## Question
<each unknown and its kind>

## Candidates
| # | Candidate | Rating | Depth | Reviewer rating | Reviewer depth | State | Refuted by | Next probe |
|---|---|---|---|---|---|---|---|---|

## Evidence
<the facts, with the command or file:line each came from>

## Reviewer
<the reviewer's verdict, including discarded candidates and what discarded them>

## Decision
<what was chosen, why, and what follows from it>

## User's answer
<the user's words and the date, when the decision was handed over>
```

**Done when** the file exists, or the one-line entry is appended, and every candidate appears in it: leader, below bar, refuted or discarded.

## Delegation

A subagent cannot spawn its own reviewer: the harness allows direct children only.

**When you delegate a state change**, pass only an action you may take yourself: a settled one (step 1), or a verdict that has cleared step 5 (Auto, or Confirm/Escalate after the user answered). The brief states:

- the basis — the reviewed verdict, the user's words quoted, the approved spec or ticket, or the deciding fact;
- the exact action and its targets, with the target check (step 2);
- the rung, or "settled (a | b | c | d)";
- "You are a delegated worker; do not spawn a reviewer."

When the worker returns, you confirm the result and write the log. A decision it returns enters your step 3: score it, review it and place it on the ladder before anyone acts on it.

**When you are the worker** — a parent agent's brief says so, or spawning a reviewer fails with a depth error — act only inside the basis your brief carries. A brief from a script is not a Delegation brief: step 6's no-human rule applies.

- The action and targets named in your brief are settled for you, whatever the rung: run the target check, act, confirm the result. Do not wait for the user's words in your own chat.
- An emergency stop (step 1 (d)) is settled for you too: stop the process, confirm the damage stopped, and report it to the parent, which logs it.
- As in step 1 (c), implementation details within those targets that change neither outside behaviour, nor reversibility, nor scope are settled.
- Any other decision that leads to a state change: stop and return it to the parent with your candidates and facts. Say "needs the user's choice" when it is irreversible or a preference.
- Do not write decision logs or `settled.md` lines; the parent logs what you return.
