---
name: decide-under-uncertainty
description: "Decide under uncertainty by digging until the evidence is sufficient, not until an answer looks plausible. Scores every option 0-10, scores evidence depth, keeps digging while depth is 8 or below, and routes sub-2-point margins to the user. An independent reviewer rates depth first and passes only options above 8/10. Everything is logged per project. Use when choosing between approaches, libraries, or configs; diagnosing an unproven failure; editing files you have not read; or before any write, delete, move, process kill, or write-API call. Also when the user says decide, pick, which is better, what should I do, or why is this broken."
---

# Deciding under uncertainty

The unreliable move is not "not knowing" — it is **stopping research once an answer looks plausible**. This process keeps digging until the evidence is sufficient, scores what it found, and puts an independent check between the verdict and the action.

Order is fixed: **name → research → score → ladder → reviewer → act → log.** Steps may be skipped only where they say so.

Two scores run through this document. Keep them distinct; they answer different questions and one never substitutes for the other.

- **Option rating (0–10)** — how well an option answers the question. 0 is complex, costly, and not working; 10 is the unambiguous answer.
- **Evidence depth (0–10)** — how proven an option is. 0 is a guess with nothing behind it; 10 is fully proven by facts.

## 1. Name the uncertainty

State in one sentence what is missing for the decision: which unknown holds the choice.

Two kinds, demanding different responses:

- **Verifiable** — the answer exists and can be fetched: read the file, run the command, call the endpoint, read the source, check the log. This is work, not a question.
- **Preference** — no source holds the answer; it lives only with the user. Do not guess it; ask.

**Done when** the uncertainty is named and classified.

## 2. Research until the evidence is sufficient

Close every verifiable uncertainty **before** options reach anyone. A cheap fact beats an elegant guess: one command replaces a paragraph of reasoning.

A **fact** is an observation you obtained here and now: command output, file contents, an endpoint response, a quote from a primary source. "Projects like this usually do X" is not a fact.

The stopping rule is the sufficiency bar, not the first plausible idea. Stop researching only when a candidate carries **all the information necessary and sufficient to act on it** — nothing material is still guesswork. Until then, the next cheap probe is the work in front of you: read the source instead of assuming its behaviour, run the failing case instead of speculating about the cause, check the actual value instead of trusting the name.

Two constraints:

- **Verify the target before touching it.** Before a delete, move, process kill, or config overwrite, state what confirmed the target is the intended one. Permission to act does not confirm the address.
- **Fix only what you understand.** When something is broken, the cause is a verifiable uncertainty too. A blind fix turns one problem into two.

**Done when** every verifiable uncertainty has a fact attached with its source, and the leading candidate clears the sufficiency bar.

## 3. Bring two to four candidates and score them

A decision is not one answer but **two to four candidates**. One candidate is acceptable only when the others were refuted by facts, and you say which fact refuted each.

Score every candidate on both scales. Anchor the option rating rather than eyeballing it:

| Rating | Anchor |
|---|---|
| 0 | does not work, or costs more than the problem is worth |
| 2 | works only with heavy rework or ongoing cost |
| 5 | works, with real caveats or collateral |
| 8 | works cleanly, minor residual risk |
| 10 | unambiguous answer to the question as posed |

Present it as:

| # | Candidate | Rating | Depth | Rests on | Refuted by | State |
|---|---|---|---|---|---|---|
| 1 | … | 9 | 9 | fact: `command` / `file:line` | … | passed |
| 2 | … | 6 | 7 | indirect sign: … | … | below bar |

Depth carries the definition from the header: 10 is proven by facts, 0 is a guess. A candidate whose depth is **8 or below is not yet researched enough** — go back to step 2 and dig, do not present it as settled. Name in "refuted by" the single action that would settle a candidate you have not finished proving.

Close with a one-line **recommendation**. It never hides the others; the user sees the full ladder.

**Done when** every candidate carries both scores, no candidate is presented as settled below the depth bar, and the recommendation is stated.

## 4. Place the decision on the autonomy ladder

The ladder decides who chooses. Place the decision **before** presenting anything.

| Rung | Test | Who decides |
|---|---|---|
| **Auto** | Reversible, local, no shared or corporate state, and the top candidate leads the runner-up by **2 points or more**. | You decide, act, report. |
| **Confirm** | Reversible but costly to undo, or touches a shared surface: a corporate system, another person's data, a harness config, a running service. Also any case where the top two candidates are **less than 2 points apart**. | You decide and act, but only after the user has seen the verdict. |
| **Escalate** | Irreversible, or the user's preference is the only thing that can separate the leaders. | The user chooses. You present; you do not act. |

The margin rule is load-bearing: **a gap under 2 points is not evidence selecting a winner**. Scores that close mean the research cannot separate them, so the user separates them — even when the decision is otherwise routine and would have been Auto. This is the case the user asked to own personally.

Judge the rung by what breaks if the verdict is wrong, not by how confident you feel.

**Done when** the decision carries an explicit rung and the margin has been measured rather than eyeballed.

## 5. Hand the verdict to the reviewer

Before every **state-changing** action, an independent reviewer examines the verdict. Reading, searching, and read-only probes need no reviewer; writing, deleting, moving, killing, or calling a write API do.

Spawn it with `subagent` (or `subagent_fork` when it needs the task context), once. The reviewer works in two passes, in this order:

**Pass 1 — rate the evidence depth.** For each candidate, assign 0–10 for how fully it is proven, independently of what you claimed: does a reproducible fact back it, is the target verified, are the consequences known. The reviewer sets this number; your own depth estimate is an input, not the verdict.

**Pass 2 — filter and check.** Drop every candidate scoring **8 or below**. For the survivors only, check (a) technical soundness — will it break what currently works; (b) completeness — is a mandatory consequence missing (migration, dependency, restart, rollback); (c) target — does it address what actually needed solving.

Brief:

> You are an independent reviewer. Rate the **evidence depth** of each candidate below from 0 to 10 (0 = guess with nothing behind it, 10 = fully proven by reproducible facts, verified target, known consequences). Set the rating yourself; the author's number is an input, not the verdict. Then discard every candidate scoring 8 or below. For the survivors only, check technical soundness, completeness of mandatory consequences, and whether the target is right. Do not propose alternatives and do not re-litigate the reasoning. Return: the depth rating per candidate, which ones you discarded and why, and concrete objections for the survivors, or "no objections".

**Only candidates with depth above 8/10 pass.** Discarded candidates are not deleted — they go to the log with their rating, and if the decision reaches the user they are shown among the options.

Handle the answer:

- **Objections** → return to step 2 and fetch a fact against each. Closed by a fact, not by argument.
- **Everything discarded** → back to step 2; there is no verdict to act on.
- **No objections** → cleared to act. This removes gross error; it does not promise correctness.

**Done when** the reviewer has rated every candidate, the survivors are above 8/10, and every objection is closed by a fact or folded into the verdict.

## 6. Act, or hand the choice over

On the Auto rung, execute and report. On Confirm, show the verdict and then execute. On Escalate, stop and present.

When the user is making the choice, show them all of it: every candidate with both scores, the recommendations, **the ones the reviewer discarded**, and the full reasoning behind each. The discarded options are part of the evidence, not noise.

In the report, separate **verified** from **assumed**: what a call confirmed, and what still rests on thinner evidence. State a gap between expectation and result plainly.

**Done when** the result is confirmed the same way facts were confirmed in step 2, or you say outright that it could not be confirmed and why.

## 7. Log the decision

Every decision that reached step 5 — taken alone or handed over — is written to the project log at:

```
<project>/.dsh/decisions/YYYY-MM-DD-<slug>.md
```

Rejected candidates are recorded with the rating that rejected them. The log is the durable record; the chat is the summary. Use this shape:

```markdown
# <decision title>

- Date: YYYY-MM-DD
- Rung: Auto | Confirm | Escalate
- Margin: <gap between leaders> points
- Outcome: acted | awaiting user

## Question
<the uncertainty and its kind>

## Candidates
| # | Candidate | Rating | Reviewer depth | State | Note |
|---|---|---|---|---|---|

## Evidence
<the facts, with the command or file:line each came from>

## Reviewer
<the reviewer's verdict, including discarded candidates and their ratings>

## Decision
<what was chosen and what follows from it; or that it awaits the user>
```

**Done when** the file exists for the decision and every candidate appears in it, passed or discarded.
