# Module 2: The verification stack

**Time:** ~30 min · **Prev:** [Module 1](01-why-verification.md) · **Next:** [Module 3: Anatomy of a reviewer agent](03-reviewer-anatomy.md)

## The big idea

A reviewer agent reads code. It does **not** run it. The Code Review docs say so plainly: reviews "do not build or run code," and "a review with no findings doesn't mean your code is error-free or secure."

So the reviewer is one layer in a stack. Each layer catches what the others can't.

## The seven layers

The companion post describes seven layers around every change:

```mermaid
flowchart TB
  L1["1 · Pre-PR verification<br/><i>tests, build, lint, verify skill, fresh-context self-review</i>"]
  L2["2 · AI review<br/><i>reviewer agent reads diff + repo + intent</i>"]
  L3["3 · Security review<br/><i>can block merge while findings are open (internal setup)</i>"]
  L4["4 · Required CI"]
  L5["5 · Risk-scaled human approval<br/><i>0, 1 or 2 humans by tier</i>"]
  L6["6 · Merge queue + flags + staged rollout"]
  L7["7 · Fast rollback"]
  L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> L7
  classDef cheap fill:#e6f4ea,stroke:#2e7d32
  classDef ai fill:#e8eaf6,stroke:#3949ab
  classDef human fill:#fff3e0,stroke:#ef6c00
  class L1,L4 cheap
  class L2,L3 ai
  class L5 human
```

**Rule of thumb: run cheap checks first.** Claude runs tests, build and lint *before* opening a PR. You can enforce this with a hook that runs tests when the agent finishes, or with a goal of "test suite passes."

## Layer 1 in depth: tests worth investing in

The anchor post names three kinds of test that give the most leverage when agents write code. It also names two general failure modes:

- **Flaky tests** teach the agent to rerun tests instead of fixing code.
- **Slow tests** mean fewer runs per batch, which makes failures harder to pin down.

### E2E tests (the post's first pick if you can only have one)

- They describe real user flows: add to cart → checkout → confirmation.
- They are "difficult to fool," have clear done criteria, and say *what* is broken.
- Unit tests can pass while the feature is broken, because "both the agent and the test don't understand the full task."
- Build a **feature map**: features from the user's point of view, with paths, selectors and expected results. It acts as both a guide and a grading rubric.

### Property tests: constraints that can't break

State something that must always hold, then let a library generate thousands of inputs that try to break it. Libraries: Hypothesis (Python), fast-check (JS/TS), proptest (Rust).

| Property | Example |
|---|---|
| Round-trip | `decode(encode(x)) == x` |
| Invariant | Balance never goes negative |
| Idempotency | Running a migration twice has the same effect as once |
| Equivalence | Optimised code matches naive but correct code |

**Why this matters for AI code:** property-test inputs aren't generated "with the same bias as the agent that wrote the code." The agent can't unknowingly write tests that only exercise the cases it already handles.

### Differential tests: the old system is the oracle

Run the old and new systems on the same inputs and compare. Any difference is either a bug or an intended change. Normalise irrelevant differences such as whitespace, and keep a list of approved changes. These are best for migrations and rewrites.

## Layer 1 in depth: "verification means running the change"

> "Running the app end to end matters more than tests."

The team asks Claude to test the app like a QA engineer would: the Chrome extension for web, an iOS or Android MCP for mobile, computer use for desktop, a local run for backends. They package this as a **`verify` skill** in `.claude/skills/`, placed as close to the relevant code as possible, and Claude uses it before committing. "This is how we test nearly all changes at Anthropic."

```mermaid
sequenceDiagram
  participant Agent
  participant Verify as verify skill
  participant App
  participant Human
  Agent->>Agent: write change
  loop up to 3 rounds
    Agent->>Verify: run
    Verify->>App: boot, drive, inspect
    App-->>Verify: trace + screenshots
    Verify-->>Agent: pass / fail with evidence
    Agent->>Agent: fix if failing
  end
  alt still failing after 3 rounds
    Agent->>Human: escalate with evidence
  else passing
    Agent->>Agent: commit, stamped "verified-by: verify"
  end
```

**Failing checks go to humans.** If a check still fails after three rounds, Claude stops and brings in a person. Failures are the signal for what needs human attention.

## Going further: one software factory's verification layers

Harjot Gill (CodeRabbit) described the layers his agents use to check their own work, in the [Pragmatic Engineer episode](https://newsletter.pragmaticengineer.com/p/software-factories-and-code-reviews). Many of them go beyond tests:

```mermaid
flowchart LR
  FM["Formal models<br/><i>TLA+ found races that<br/>code review missed; Lean explored</i>"] --> DS["Deterministic simulation<br/><i>fuzz concurrent interleavings locally</i>"]
  DS --> EM["Emulated real APIs<br/><i>e.g. cloud APIs, offline</i>"]
  EM --> DEV["Dev environments<br/><i>agent deploys and checks the service</i>"]
  DEV --> PROD["Progressive rollout<br/><i>flags + an agent watching perf/logs,<br/>widening the rollout when stable</i>"]
```

How these map onto the seven layers:

| Gill's layer | Seven-layer slot | What it catches that a reviewer can't |
|---|---|---|
| Formal models (TLA+) | Pre-PR (design) | Race conditions and protocol bugs in the *design*. He reports TLA+ found races that code review missed |
| Deterministic simulation | Pre-PR (tests) | Interleavings too rare to hit in normal tests |
| API emulation | Pre-PR / CI | Integration behaviour without real credentials or quotas |
| Dev-env deploy | `verify` skill | "Does it actually run": the anchor post's "run the change" |
| Agent-watched rollout | Layers 6–7 | Production-only regressions, caught early and limited to a small share of traffic |

Details from the transcript:

- **When TLA+ is worth it:** it is "not great for things which are algorithms," but strong for "messages and actors and interactions." CodeRabbit used it on a Slack bot product with queues, message steering and sandbox lifecycle states. They **modelled the states before writing any code**, and TLA+ found "race conditions which we had overlooked." A simulation harness that fuzzes interleavings found more. **Lean** is used for things that need to be provably correct, like the billing service.
- **Counterpoint:** Orosz cites Hillel Wayne, a TLA+ expert, saying formal methods are probably overkill for almost all software. Use them for distributed state machines, not everywhere.
- **Agents in production:** agents "own health" of components such as billing and the knowledge base. On every deploy they either **add telemetry if they're "flying blind"** or fix something. Gill's argument: "alerts are limited by your imagination on the failure scenarios you imagine," while agents can read logs and traces directly.

The lesson for reviewer design: when expensive checks like formal models and simulation cover **concurrency and protocols**, the reviewer agent can stop guessing about races and point to those results instead.

## Test hygiene at factory scale: "deslopping"

Agents write a lot of tests, and many of them are useless. CodeRabbit runs **janitorial agents** ("Roombas") all the time:

- **Deslop tests.** Remove useless and redundant unit tests and move coverage toward **integration and E2E tests**. This agrees with the anchor post's preference for E2E tests.
- **Remove flaky tests continuously**, because "each flaky test resets the merge queue."
- **Keep a CI time budget:** ideally ≤ 3 minutes (they're at about 4). Slow CI means fewer agent iterations and a higher compute bill.
- **"Let the chaos reign, then rein in the chaos."** Accept that the codebase gets messier as agents ship, including a weaker security posture when auth layers aren't reused. Then run agents to clean up flaky tests, bad abstractions and refactors.

Compare this with the anchor post's approach of no sleeps or retries, deleting and documenting flaky tests, and pruning routinely. Both treat the test suite as something that keeps growing unless you tend it. CodeRabbit automates the tending. Anthropic writes it down as policy.

## Where the reviewer agent sits, and what it uniquely does

| Question | Tests | `verify` skill | Reviewer agent | Human |
|---|---|---|---|---|
| Does it run? | ✅ | ✅ | ❌ | – |
| Does it do what the user flow needs? | partly | ✅ | partly | ✅ |
| Does the diff match the stated intent? | ❌ | ❌ | ✅ | ✅ |
| Does it break things far from the diff, like old data or other callers? | if tested | rarely | ✅ (reads whole repo) | sometimes |
| Does it violate team rules and lessons from past incidents? | ❌ | ❌ | ✅ (REVIEW.md, checklist) | if they remember |
| Should we build this at all? | ❌ | ❌ | ❌ | ✅ |

The reviewer's niche is **reading with context at scale**: intent, the whole repo, team rules, incident history. The niche for tests and `verify` is **execution**. Don't ask a reviewer to do execution's job.

## Commit stamping and gate modes

Every commit is stamped with how it was verified, **including "none"**, so unverified commits can be queried. A commit gate can block commits from sessions that didn't run `/code-review` or `/verify`. Module 7 covers the gate's three modes: off → observe → enforce.

## Key takeaways

- The reviewer reads; tests and `verify` run. You need both.
- Order layers from cheapest to most expensive, and escalate to people based on failures.
- E2E, property and differential tests give the most leverage against AI blind spots.
- Stamp each commit with its verification so you can measure coverage.

---

## Exercises

**2.1 Pick the test.** Choose the best test type for each case.
1. Rewriting a pricing engine from Java to Go; behaviour must not change.
2. A new CSV import/export feature.
3. A checkout redesign.
4. A retry wrapper around payment capture.

<details><summary>Answer</summary>

1. **Differential.** The old engine is the oracle; replay production samples.
2. **Property, round-trip.** `import(export(rows)) == rows`, using generated rows with unicode, commas, quotes and empty fields.
3. **E2E**, driven from the feature map: add, pay, confirm, with screenshots.
4. **Property, idempotency.** Capturing twice with the same idempotency key equals capturing once. Add an invariant: a charge never exceeds the order total.
</details>

**2.2 Draw your stack.** Copy the seven-layer diagram and mark each layer for your team as ✅ exists, 🟡 partial or ❌ missing. Which missing layer would remove the most human review time?

<details><summary>Hint</summary>

For most teams the answer is layer 1 (a reliable `verify` skill) or layer 5 (risk tiers, so low-risk PRs need no human). A reviewer agent with no layer 1 underneath spends its rounds on bugs a test run would have caught for free.
</details>

**2.3 Should we use TLA+?** For each, decide whether formal modelling is worth it: (a) a job queue with retries, cancellation and worker leases; (b) a tax calculation function; (c) a React settings page.

<details><summary>Answer</summary>

(a) **Yes.** Messages, actors and states are where TLA+ found races at CodeRabbit. (b) **No.** It's an algorithm, so use property and differential tests (or Lean if it must be provably correct, as CodeRabbit does for billing). (c) **No.** Use E2E tests from a feature map.
</details>

**2.4 Spot the anti-pattern.** An agent hits a flaky test, adds `sleep(2)`, and the test passes. What does the anchor post say to do?

<details><summary>Answer</summary>

Claude may **not** skip flaky tests, add sleeps, or retry until they pass. Find the root cause. If you can't, **delete the test and write down why**. A flaky test that stays in the suite teaches the agent that rerunning is a fix.
</details>
