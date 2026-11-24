# Module 9: Capstone — your rollout plan

**Time:** ~60 min · **Prev:** [Module 8](08-expanding-the-reviewer.md) · **Back to:** [README](../README.md)

## The brief

Write the plan to roll out (or fix) an AI reviewer agent for a team you know. At the end you should have five artifacts:

1. A **90-day plan**
2. A **REVIEW.md** (≤ 40 lines)
3. A **reliability checklist** with ≥ 5 diff-triggered items from real history
4. An **eval design** (positives, controls, split, scorer)
5. A **scorecard**, with objective, guardrail and budgets

## Reference 90-day plan

```mermaid
gantt
  title Reviewer agent rollout (template)
  dateFormat  YYYY-MM-DD
  axisFormat  %b %d
  section Foundations
  Commit stamping + revert tagging     :a1, 2027-01-04, 10d
  Protect base branch, REVIEW.md v0    :a2, after a1, 5d
  verify skill for main app            :a3, 2027-01-04, 20d
  section Eval
  Mine reverts/incidents → 60 cases    :b1, 2027-01-11, 15d
  Size-matched controls + split        :b2, after b1, 5d
  Calibrate grader (kappa vs humans)   :b3, after b2, 5d
  Baseline scorecard                   :milestone, b4, after b3, 0d
  section Observe
  Reviewer advisory on all PRs         :c1, after b4, 21d
  Author-response tagging              :c2, after b4, 21d
  Owner-fit loops (top 3 areas)        :c3, after b4, 21d
  section Enforce + expand
  Risk tiers in observe                :d1, after c1, 14d
  Enforce Important on medium tier     :d2, after d1, 10d
  Misses→rules cadence + first prune   :d3, after d1, 14d
```

## Scorecard template

| Metric | Baseline | Target @90d | Budget / tripwire | Owner |
|---|---|---|---|---|
| Human review min / merged PR (median) | | ↓ 30% | – | Eng lead |
| Revert rate (7-day) | | ≤ baseline | Breach → back to observe | QE |
| Sev2+ incidents / 100 PRs | | ≤ baseline | Breach → back to observe | QE |
| Held-out post recall | | ≥ 65% | −2 pts per change | QE |
| Harmful FP / control PR | | ≤ 0.10 | +0.02 per change | QE |
| Useful rate, Important | | ≥ 70% | < 60% → re-calibrate | QE + owners |
| % PRs pass after 1 round | | ≥ 75% | – | QE |
| % commits stamped "none" | | ≤ 10% | – | Platform |
| Cost / review | | – | +25% needs sign-off | Platform |

## Self-assessment

Score 0 (no), 1 (partly) or 2 (yes). Under 14 means read the linked module again.

| # | I can… | Module |
|---|---|---|
| 1 | State my team's objective **and** guardrail metric | 1 |
| 2 | Explain why the reviewer doesn't replace tests or `verify` | 2 |
| 3 | Draw Find → Verify → Rank → Post and say what each optimises | 3 |
| 4 | Name the five integrity rules | 3 |
| 5 | Write a diff-triggered checklist item from a real incident | 4 |
| 6 | Name five AI-specific failure modes | 4 |
| 7 | Split ownership between code owners and QE | 5 |
| 8 | Build an eval set with answer keys and controls | 6 |
| 9 | Compute recall per stage and say which stage to fix | 6 |
| 10 | Compute and interpret Cohen's kappa | 7 |
| 11 | Write observe → enforce exit criteria | 7 |
| 12 | Run a rule through the expansion lifecycle | 8 |

## Common pitfalls (pre-mortem)

Imagine it's day 90 and the rollout failed. The likely reasons:

- **No guardrail data.** Reverts weren't tagged, so nobody could prove the reviewer wasn't making things worse.
- **Enforced on day 1.** False positives blocked agents, people found workarounds, and trust fell.
- **Stacked reviewers.** Two bots with conflicting findings doubled the rounds.
- **The judge was never calibrated.** The eval looked great, then reality didn't.
- **REVIEW.md grew to 400 lines.** Nothing was ever pruned.
- **The PR could edit the reviewer's config.** One agent "fixed" the rule instead of the code.
- **Nobody owned the model upgrade.** Recall shifted silently.
- **Review scaled, CI didn't.** At CodeRabbit, CI spend reached about the same as token spend, a vendor quoted ~$13M/year, and GCP ran out of CPU. Put CI minutes and cost per PR on the scorecard, and keep CI under ~3–4 minutes.
- **Agents needed people around the clock.** Gill wakes at 3 a.m. because "every hour like one or two agents are going to get stuck." Agents running on one person's laptop are a single point of failure. Plan for shared, cloud-hosted agent sessions, someone on shift to answer them, and a `why_human` on every escalation.
- **Quality was claimed, not measured.** "Our defect rate... is going down... I still have to measure it." Without the guardrail data from Module 1, neither the optimists nor the sceptics can show they're right.
- **Agents ran without limits.** An agent harness "reaching too far" (the Railway incident linked from the Pragmatic Engineer episode) shows that how far agents may reach needs its own limits, not just review.

## Debate exercise: "Human code review is dead"

Harjot Gill argues that people should review **plans and releases**, not diffs. Anthropic's teams still require 1–2 humans on medium- and high-risk PRs. Write one paragraph for each side, using evidence from these modules, then a third paragraph with your position for **your** team.

<details><summary>Points to consider</summary>

- *For Gill:* line-by-line review doesn't scale to thousands of PRs, and Orosz agrees that for many teams it is already "theater" ("LGTM", with no real reading). People still review: they decide whether to ship and how risky something is, at release level. Formal models, simulation and agent-watched rollouts catch what reviewers miss. Exposure time is minimised with 10–15 minute rollbacks. The driving license keeps people accountable. His most provocative line: "humans were writing slop, not AI... AI is actually writing less slop at this point if it can validate against something."
- *For Anthropic:* some risk has to be caught **before** prod (billing, auto-updater, CI workflows). Agents have no track record to strike. Their measured result was more than 2× PRs with 40%+ fewer reverts, and they kept humans on high-risk paths to get it.
- *Common ground:* both move people away from reading every diff toward defining "done" and checking evidence. The disagreement is only about where the **remaining** human review sits: on risky diffs, or on releases.
</details>

## Final exercise

Write the five artifacts. Then review your own plan with the reviewer mindset: for each artifact, write one finding labelled Important, Nit or Pre-existing. If you can't find an Important, ask a teammate to try.
