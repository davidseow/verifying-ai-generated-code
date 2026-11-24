# Sources

Gathered 24 Nov 2026. **Primary** means Anthropic's own posts or docs. **Secondary** means vendors, blogs or aggregators. Treat secondary figures as directional.

## Primary (Anthropic)

- [How the Claude Code team thinks about verification](https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification). Addy Osmani, 16 Nov 2026. The anchor post: test types, the `verify` skill, the 39-item reliability checklist, one reviewer with multiple rounds, observe mode, testing the reviewers, turning misses into rules.
- [How the Claude Code team does code review](https://claude.dev/blog/solving-code-review-at-anthropic/). 15 Oct 2026. Find/Verify/Rank pipeline, risk tiers, round caps, incident/revert eval set, convergence metrics.
- [Code Review docs](https://code.claude.com/docs/en/code-review). REVIEW.md, severity labels, base-branch reading rules, presentation of findings.

## Practitioner perspective

- [Pragmatic Engineer: Software factories and code reviews with Harjot Gill](https://newsletter.pragmaticengineer.com/p/software-factories-and-code-reviews). Gergely Orosz, 18 Nov 2026. CodeRabbit's "software factory": 5,000+ PRs a month from ~40 engineers, ~$250K/month AI spend, verification layers (TLA+, deterministic simulation, API emulation, dev envs, agent-watched rollouts), release-level review, the "review exempt" list with strikes, and the decision to stop fine-tuning review models. The full transcript was read in the browser (the page loads it client-side). It is auto-generated and has transcription errors: "hardness" means *harness*, "Harjit" means Harjot, and model names are garbled. Quotes in these modules are lightly corrected for that. Clarifications from the transcript: the ~$250–275K/month is **Gill's personal** spend, while the company spends "several million dollars a month." 5,000+ PRs/month is Gill's personal count, and 5,000+/week is company commits. The CI saving (~50%) is unconfirmed ("I have to see"). His falling-defect-rate claim is explicitly unmeasured.

## Secondary: AI review accuracy and benchmarks

- [Augment: AI code review accuracy](https://www.augmentcode.com/guides/ai-code-review-accuracy)
- [Augment: Deep code review, recall vs precision](https://www.augmentcode.com/guides/deep-code-review-recall-vs-precision)
- [Signal65: Evaluating AI Code Review Tools (PDF, Mar 2026)](https://signal65.com/wp-content/uploads/2026/03/Signal65-Insights_Evaluating-AI-Code-Review-Tools.pdf)
- [Atlas: AI-assisted code review in 2026, risks and limits](https://laoujin.github.io/Atlas/research/2026-06-09-ai-assisted-code-review-in-2026-landscape-tools-evidence-and-limits/risks-limitations-and-adoption-barriers/)
- [DEV: We ran 4 AI reviewers in parallel for 3 weeks](https://dev.to/_vjk/best-ai-code-reviewer-in-2026-we-ran-4-in-parallel-for-3-weeks-146-prs-679-findings-1c0f)
- [Redwerk: Claude Code Review, what it catches and what it misses](https://redwerk.com/blog/claude-code-review/)

## Secondary: LLM-judge calibration

- [MLflow: LLM judge calibration](https://mlflow.org/articles/llm-judge-calibration/)
- [OneUptime: Calibrate an LLM judge with Cohen's kappa](https://oneuptime.com/blog/post/2026-08-31-calibrate-llm-judge-cohens-kappa)
- [Arize: Measuring human–LLM judge alignment](https://arize.com/?p=30225)
- [A Survey on LLM-as-a-Judge (arXiv 2411.15594)](https://arxiv.org/pdf/2411.15594)
- [Judge's Verdict: LLM judge capability through human agreement (arXiv 2510.09738)](https://arxiv.org/pdf/2510.09738)

## Secondary: Quality Engineering and the verification bottleneck

- [O'Reilly Radar: AI is writing our code faster than we can verify it](https://www.oreilly.com/radar/ai-is-writing-our-code-faster-than-we-can-verify-it/)
- [Qodo: 2026 State of AI Code Quality](https://www.qodo.ai/?p=22120)
- [Hackernoon: Why the auditor cannot be the creator](https://hackernoon.com/product-quality-in-the-ai-sdlc-why-the-auditor-cannot-be-the-creator)
- [Sonar: Quality assurance in the AI era](https://www.sonarsource.com/blog/quality-assurance-in-the-ai-era/)
- [Test Quality: Shift left on AI-generated code](https://testquality.com/?p=5717)
- [Entrepreneur: AI wrote your code, did anyone check it?](https://www.entrepreneur.com/building-a-business/ai-wrote-your-code-did-anyone-actually-check-it-heres-the-verification-problem-most-companies-arent-prepared-for)

## Known gaps in the primary sources

The Anthropic posts do **not** publish: precision or false-positive rates, recall per stage, the verifier vote threshold, model names, agent counts, or the contents of the 39-item checklist beyond one example. Where these modules give example numbers for those things, the numbers are illustrative and labelled that way.
