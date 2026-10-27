---
name: decision-audit
description: "Use before choosing between approaches, libraries, configs or fixes; before editing a file you have not read or any write whose choice is still open; before every delete, move, process kill, write-API call or shared-surface change, even one the user ordered; when the user corrects or rejects a result; and when they answer a decision this skill handed over. Also when the user says decide, pick, which is better, or what should I do. Proves the leader with facts, then gets an independent review."
---

# Deciding under uncertainty

The failure this prevents: stopping research once an answer looks plausible. Dig until facts prove the answer, score it, and put an independent check between the verdict and any state change. Order: classify → research → score → review → ladder → act → log; skip a step only where this text says so. If this text is gone from your context or shows a "pruned" marker, load the skill again before the next step. Read each `references/` file at the step that names it.

**Terms.** A *fact* is observed here and now: command output, file contents, a response, a primary-source quote, the user's own words. A *shared surface* is a corporate system, another person's data, a running service, or the harness you run in (its config files, installed plugins, UI). *Observing a result* means seeing the effect where it happens (for a UI, the rendered view in the real app); a written file, a registered slot, present CSS, a copy rendered elsewhere or a passing build (unless asked for) is only a probe.

## Scores

Keep both; neither replaces the other. Rate as if unconfirmed points hold; what is unconfirmed lowers depth.

- **Rating**, how well it answers: 0 does not work or costs more than the problem; 2 heavy rework or ongoing cost; 5 works with real caveats; 8 clean, minor known drawback; 10 fully answers, no caveats.
- **Depth**, how proven: 10 reproducible facts obtained here, target verified, consequences known; 9 only minor consequences unconfirmed; 8 or below: something material is inference. The bar is 9. Refuted by a fact: no depth; record the fact.
- **Leader**: the highest of your ratings among candidates at the bar that passed review. **Margin**: leader minus the best other unrefuted candidate, at the bar or not, computed with your ratings and again with the reviewer's for the same leader; if either is under 2, it is under 2. A margin of 2+ counts only if the runner-up's rating rests on a cited fact. No unrefuted rival: met.

## 1. Classify

Name each unknown that holds the choice: **Verifiable** (fetch it), **Preference** (research the verifiable part; one viable option left means it was verifiable; otherwise Escalate), or **Settled**, allowed only on a stated basis:

- (a) the user's message names the concrete action and target and leaves no choice of how ("fix X" settles that, not how);
- (b) reversible, local, no shared surface, rates 5+, result observable, and a cited fact rules out each alternative;
- (c) an implementation edit inside an approach the user approved or a review chose (a TDD step only inside one): reversible, local, no shared surface, version-controlled, no change to outside behaviour or scope; name the check and its expected result;
- (d) emergency stop: output shows damage now (data or resources destroyed or exhausted). Confirm the PID and that it is the source, stop it and its children, confirm the damage stopped, log it.

A parent agent or script is never (a); a delegated worker reads `references/delegation.md` first. Irreversible: only (a) or (d). Settled actions skip steps 3–5: check the target, act, observe. An unexpected (c) result: undo, or reproduce the cause first; the next edit is a fix ((b) or step 3); stop after two in a row.

**Follow-ups are new decisions.** A correction, rejection, rework request, or an answer that redirects a handed-over plan opens a new decision; an earlier approval covers only what it named. Change only what the request or verdict names. A Preference question may come before scoring when its answer decides which candidates exist; it is not the step-6 answer.

## 2. Research

Close every verifiable uncertainty that can change the leader or the rung before anything reaches the user; the next cheap probe is the work. Probes change nothing lasting (output, build artifacts, your own scaffolding removed at the end, git operations that restore the tree, not history, index or stash); everything else is a state change. Before a delete, move, kill, config overwrite or write-API call, state what confirmed the target; for an irreversible one name it exactly (resolved path, contents, whether links are followed; PID and command line; host and identifier); a mismatch with what the user meant, data behind links included, opens the choice. Causes are verifiable: reproduce before fixing; candidates are fixes. If diagnosing-bugs runs, return here before its fix; that fix is never (c).

**Exit** to step 5 as Escalate, margin n/a, after scoring what you have: the next probe needs access you lack or an irreversible or shared change (a dry run or scratch copy counts as runnable); two probes at the Next-probe point settled nothing; a failure did not reproduce within the budget you set. All refuted: look once more, then report that nothing works.

## 3. Candidates

Two to four, including the strongest known alternative; one only when facts refuted the rest. Table: # | candidate | rating | depth | rests on | refuted by | next probe | state. Only the leader must reach the bar. End with a one-line recommendation; the user sees the full ranking.

## 4. Review

For any state change chosen here (not a preference-only choice between reversible, local candidates): read `references/review.md`, spawn one `subagent` with its brief, and end your turn with the spawn as the last call. Until the verdict: no state change, no polling or sleeping, no asking the user to choose. A reviewer's number for its own alternative is not a review.

## 5. Ladder (the most restrictive match wins)

| Rung | Test | Who decides |
|---|---|---|
| Auto | reversible, local, no shared surface, result observable, both raters rate the leader 5+, margin 2+ | you act and report |
| Confirm | costly to undo, a shared surface, or a result you cannot observe | you act only after an explicit yes in chat |
| Escalate | irreversible; margin under 2 or n/a; leader under 5 by either rater; open Preference; failed review | the user chooses |

A gap under 2 points is not evidence: the user separates close candidates, even in routine work. Judge by what breaks if you are wrong.

## 6. Act or hand over

Confirm or Escalate: log first (read `references/log.md`), then present every candidate with both raters' scores, the discarded ones and the reasoning; a question tool is presenting too and carries the same content. Advice only: the answer is the result; a pick is not a request to act. The user's pick settles it; an answer outside the candidates is a follow-up. No human (headless, a script parent): never act on Confirm or Escalate. Observe the result: cheapest channel first (DOM or accessibility text before pixels; crop screenshots), in the state that exercises the change, within an attempt budget set beforehand (default 3). Driving the user's live app to observe is a state change. Unobserved is not done: say what would confirm it. A failure: stop and report; reverting is a new decision.

## 7. Log

Read `references/log.md` before any log write. Each decision from steps 3–6 gets its own file under `<project>/.dsh/decisions/`; a settled action that was irreversible, shared, an emergency stop or failed gets a `settled.md` line; whoever learns a fact that changes a logged result updates that file's Outcome line. Acted without this procedure? Tell the user, log it with Rung: none, return to step 1. Before delegating, read `references/delegation.md`.
