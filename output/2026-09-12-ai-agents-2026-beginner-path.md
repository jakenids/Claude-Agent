# AI Agents in 2026: What Works, What's Overhyped, and How to Learn to Build One From Zero

| | |
|---|---|
| **Date** | 2026-09-12 |
| **Framing question** | If you can't code today, what is the highest-return sequence of things to learn in order to be building useful AI agents — and which parts of the 2026 agent landscape are real enough to be worth learning against? |
| **Scope** | Learning-path weighted. Current agent practice (2025–2026) as context for a from-zero curriculum. |
| **Out of scope** | Pre-2025 agent history; AI safety/alignment as a field; enterprise procurement; model training and ML theory; salary and job-market forecasting. |
| **Audience** | A career switcher with no coding background. |
| **Depth** | brief — 6 parallel researchers, ~60 distinct sources consulted, ~45 cited. |

> ## ⚠️ Read this before you trust any number in this report
>
> **No primary source in this report was read directly.** This session's network
> policy blocked outbound access to every research domain (arxiv.org,
> anthropic.com, metr.org, dora.dev, mckinsey.com and the rest returned 403 at
> the egress proxy). Every citation below rests on search-index summaries of
> those pages, not on the pages themselves.
>
> That is a real limitation and it is not evenly distributed. It barely touches
> the arguments — which rest on the *shape* of the evidence and on multiple
> independent sources agreeing. It bears directly on the precise figures.
>
> Every claim is therefore graded:
>
> | Grade | Meaning |
> |---|---|
> | **[A]** | Multiple independent sources agree; named primary document, consistently described. Safe to rely on. |
> | **[B]** | One credible source, consistent across summaries, not directly read. Directionally trustworthy; verify the digits before quoting. |
> | **[C]** | Vendor or aggregator only. A *claim*, reported as such. Do not repeat as fact. |
> | **[✗]** | Found, investigated, and **rejected** as unverifiable or laundered. Listed so you know it was considered. |
>
> Anything marked *(inference)* is my reasoning, not a finding.

---

## Executive summary

- **Agents work inside a narrow envelope, and almost all the hype is extrapolation outside it.** Capability is rising fast on short, well-specified, verifiable tasks and slowly on long, underspecified ones: the best agent scores ~80% on short computer-use tasks but **20.6%** on OSWorld 2.0, whose median task takes a human 1.6 hours. Failure follows a constant hazard rate, so success decays exponentially with task length.

- **Evaluation is the highest-return skill you can learn, and it is the one where having no coding background is an *advantage*.** It is simultaneously the #1 production blocker (32%, ahead of cost), something ~48% of teams simply don't do, the most-requested skill across 4,894 AI-engineering job ads, and explicitly recommended by leading practitioners to be **led by domain experts rather than engineers**. No vendor has commoditised it, because the scarce input is judgement about what "good" means in a specific domain.

- **Yes, you need to learn to code — but as literacy, not fluency, and the sequence inverts the traditional one.** An Anthropic RCT found AI-assisted learners scored **50% on comprehension vs 67%** for those who hand-wrote code, with *no speed gain* and the worst gap in debugging. But the same study found the *mode* of use predicted the damage: asking the model to explain preserved learning; pasting errors back in a loop destroyed it. So: **ship something before you can code, then learn code specifically as the ability to read and verify, then write the agent loop by hand once.** First useful agent ≈ 3–4 months at 10 hrs/week; agent others depend on ≈ 6–9 months.

- **Skip everything that exists to make a system scale; keep everything that makes it correct.** Vector databases, Kubernetes, multi-agent orchestration and fine-tuning are scale optimisations — and you have no users. Anthropic removed vector search from Claude Code in favour of plain grep; an AAAI 2026 paper got ~94.5% of RAG faithfulness with no vector store at all. Multi-agent is the most over-learned topic in the field: 41.8% of its failures are design problems, and both leading camps tell beginners to start with one agent.

- **Pick tools by whether they outlive a vendor's roadmap, not by no-code vs. code.** OpenAI killed two agent abstractions in 2026 (Agent Builder lasted eight months; the Assistants API was removed entirely). Flowise was archived by its own maintainers at 55k stars. Meanwhile Pydantic AI shipped a breaking 2.0 nine months after 1.0, and the Vercel AI SDK has had **four breaking majors in 19 months** — while MCP has held 1.x for ~21 months under foundation governance. Roughly **four in five** organisations running LLM workloads use no agent framework at all.

- **Trust your eval numbers, not your feelings.** Experienced developers in a controlled trial were **19% slower** with AI while believing they were **20% faster**, and fluent-feeling learning reliably produces illusions of competence. The progress rungs that matter are: read a trace and say why it failed → build a 30-case eval set → change something and *show* the score moved. Almost no course sequences those, and they are exactly what distinguishes a builder from a tutorial completer.

---

## 1. The landscape in one page

**An "agent" in 2026 means one specific thing: a language model that runs tools in a loop until a goal is met.** The load-bearing word is *loop* — the model decides what to do next, rather than following steps a developer wrote out in advance. Simon Willison arrived at this definition by distilling 211 crowdsourced attempts, and it has largely stuck **[A]** ([Willison, Sept 2025](https://simonw.substack.com/p/i-think-agent-may-finally-have-a)).

That gives you the distinction that matters most for your first year:

| | **Workflow** | **Agent** |
|---|---|---|
| Who picks the next step | You, in advance | The model, at runtime |
| Predictable cost and latency | Yes | No |
| Debuggable | Straightforwardly | Hard |
| Right when | The steps are known | The steps genuinely can't be known ahead |

Anthropic calls both "agentic systems" but insists the split matters, because most problems people reach for an agent to solve are workflows wearing a costume **[A]** ([Anthropic, Dec 2024](https://www.anthropic.com/engineering/building-effective-agents) — 21 months old and still the canonical framing).

**Term-stretching is measurable, not just annoying.** Gartner estimates only ~130 of the thousands of vendors marketing "agentic AI" are genuinely agentic, and coined *agent washing* for the rest — rebranded chatbots and RPA **[B]** ([Gartner, June 2025](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027)).

### Which deployment patterns actually work

Four patterns get discussed. They do **not** have equal evidence behind them *(inference — the ranking is mine; the underlying evidence is sourced)*:

**1. Coding agents — strongest evidence.** A Microsoft study of tens of thousands of its own engineers over a four-month 2026 rollout found adopters merged ~24% more pull requests **[B]** ([arXiv, July 2026](https://arxiv.org/abs/2607.01418) — Microsoft researchers studying Microsoft, so read as a self-study). Contested; see §2.

**2. Research and retrieval agents — real, with a known flaw.** Together with support, these make up over half of production deployments in LangChain's survey of 1,300+ practitioners (research/data analysis 24.4%) **[B]** ([LangChain, 2026](https://www.langchain.com/state-of-agent-engineering) — vendor-run). They are competent but not citation-trustworthy: on DeepResearch Bench, citation accuracy runs ~78–94% with **3–13% of URLs fabricated** **[B]** ([arXiv, June 2025](https://arxiv.org/abs/2506.11763)).

> This report is itself a live demonstration of that failure mode. One researcher cited a Stack Overflow URL announcing that a survey had *opened* as the source for the survey's *results*. Caught only because the mismatch was visible in the URL. Assume this is happening in every research agent you use.

**3. Customer support deflection — works in a narrow band.** Roughly a 41% median deflection rate, but the distribution is what matters: simple lookups deflect at 65–80%, while anything needing judgement or handling an annoyed human rarely clears 25% **[C]** ([vendor aggregation, 2026](https://aissist.io/industries/ai-customer-service-benchmark-2026)). Vendor-published figures (Intercom's 67–76%) run 30–40 points above independently measured ones (38–53%) **[B]** ([independent review, Aug 2026](https://clonedesk.ai/blog/intercom-fin-limitations)). **Halve any support number you are quoted.** Klarna is the cautionary tale: after claiming its AI did the work of 700 agents, it resumed human hiring, the CEO conceding "we went too far" **[A]** ([Forbes, May 2025](https://www.forbes.com/sites/quickerbettertech/2025/05/18/business-tech-news-klarna-reverses-on-ai-says-customers-like-talking-to-people/)).

**4. Computer-use and browser agents — not yet.** Benchmark scores do not survive live websites. Online-Mind2Web found frontier agents up to **59% less competent on real sites** than static benchmarks imply, and one practitioner reported an agent scoring 78% on WebArena completing only 22% of real carts — broken by cookie banners, session logouts and a CSS change **[B]** ([2026](https://futureagi.com/blog/evaluating-browser-use-agents-2026/)). **Do not build your learning plan around this pattern.**

### The plumbing, conceptually

**Tool connection has consolidated on MCP** (Model Context Protocol — a standard way to describe a tool to a model and get back a structured request to run it). Anthropic donated it to the Linux Foundation's Agentic AI Foundation in Dec 2025, co-founded with OpenAI, Google, Microsoft, AWS and Block **[A]** ([Linux Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)). Learn the *concept*; the wire protocol was substantially rewritten in July 2026, so the layer is settled while the surface still moves *(inference)*.

**The real constraint is context rot, not context size.** All 18 frontier models tested degrade as input grows, well before the stated window limit **[B]** ([Chroma evaluation](https://www.morphllm.com/context-rot)). Bigger context windows do not mean you can stop curating what goes in them.

---

## 2. What's overhyped

Being precise here matters more than being cynical. Capability is rising genuinely fast on **short, well-specified, verifiable** tasks. It is rising much more slowly on **long, underspecified, multi-app** tasks. *Nearly every overclaim in this field comes from quoting the first curve and implying the second* **[A]** — this is the single most useful sentence in the report.

### The autonomy gap is large and measured

| Benchmark | What it measures | Best agent |
|---|---|---|
| OSWorld 1.0 | Short computer-use tasks | ~80%+ |
| **OSWorld 2.0** | 108 workflows, median human time ~1.6h | **20.6%** (avg 318 tool calls) |
| TheAgentCompany | 175 long tasks in a simulated firm | **30.3%** autonomous |

**[B]** ([OSWorld 2.0, June 2026](https://arxiv.org/abs/2606.29537); [TheAgentCompany, Dec 2024](https://arxiv.org/abs/2412.14161) — the latter is stale on models; treat the *shape* as the finding). Notably, agents did far better on software tasks (~42%) than HR (~18%) or finance (~22%) — the weakness is coordination and messy interfaces, not code.

**Failure is arithmetic, not just engineering.** Toby Ord's re-analysis of METR data finds a *constant hazard rate* — a fixed chance of failing per human-minute of task length — meaning success decays exponentially with task length, giving each model a "half-life" **[B]** ([arXiv, May 2025](https://arxiv.org/abs/2505.05115)). Long-horizon autonomy is hard in a way more training doesn't straightforwardly fix.

**Reliability, not capability, is the binding constraint.** τ-bench's pass^k metric asks whether an agent solves the same task on *all* k attempts: pass^8 falls below 25% in the retail domain even for models with respectable single-attempt scores **[B]** ([ICLR 2025](https://iclr.cc/virtual/2025/poster/28170)). A 90%-on-one-try agent is not a 90% reliable agent.

### Benchmarks are worse evidence than they look

- **OpenAI stopped reporting SWE-bench Verified** after auditing 138 failures and finding **59.4% were caused by broken tests or environments**, not model limits **[B]** ([OpenAI, 2026](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/)). A vendor disowning a benchmark it helped popularise is strong signal.
- An independent audit found **~33% of SWE-bench instances leak the solution** in the issue text, and 25–31% of "successful" patches pass only because tests are too weak to catch a wrong fix. Filtering both dropped one agent from 12.47% to **4.58%** **[B]** ([SWE-Bench+, Oct 2024](https://arxiv.org/pdf/2410.06992)).
- Princeton's Holistic Agent Leaderboard (21,730 rollouts, ~$40k compute) found **cost differences of up to 100× for ~1% accuracy**, and that higher reasoning effort *reduced* accuracy in most runs **[B]** ([arXiv, Oct 2025](https://arxiv.org/abs/2510.11977)).

> **Take this heuristic with you** *(inference, derived from the three findings above)*: a benchmark number quoted without **(a)** a date, **(b)** the scaffold used, **(c)** a cost, and **(d)** a pass^k reliability figure is marketing. That four-part test will serve you for years.

### Multi-agent is the most over-learned topic in the field

It dominates course syllabi and framework marketing — crews, swarms, graphs — and the empirical record does not support the emphasis.

- Berkeley's MAST study annotated 1,600+ multi-agent traces across 7 frameworks and attributed **~41.8% of failures to specification and design problems** — ambiguous roles, bad decomposition, missing termination conditions — rather than model weakness **[B]** ([arXiv, Mar 2025](https://arxiv.org/abs/2503.13657)).
- A 2026 comparison found **single-agent-with-skills matched multi-agent accuracy while cutting tokens ~54% and latency ~50%** **[C]** ([arXiv, Jan 2026](https://arxiv.org/html/2601.04748) — could not be fetched; medium confidence).
- Cognition (makers of Devin) argues directly against it: splitting context across agents fragments assumptions and compounds failure **[B]** ([Cognition, June 2025](https://cognition.com/blog/dont-build-multi-agents)).

**The counter-position, stated fairly:** Anthropic reports its orchestrator-worker research system beat single-agent Claude Opus 4 by **90.2%** on an internal eval — at roughly 15× the tokens **[C]** (vendor, internal eval, one narrow task class). **Where this lands:** multi-agent appears to help only when subtasks are genuinely independent and read-only, and to hurt whenever agents must share evolving state. Both camps tell a beginner the same thing — **start with one agent**.

### The enterprise-failure statistics are mostly laundered

You will see "88% of agent pilots never reach production" everywhere. **[✗] Do not cite it.** Every instance traced to SEO content farms citing each other, with the figure varying between 67%, 78%, 88% and 89% for supposedly the same finding — a classic laundering signature. The widely-quoted MIT NANDA "95% of pilots fail" figure is also misread: it measures *absence of measured P&L impact*, largely an artefact of pilots having no pre-deployment baseline **[B]** ([critique, 2025](https://www.sify.com/ai-analytics/95-companies-failing-with-ai-an-mit-nanda-report-misread-by-all/)).

The better-methodology substitute tells a less dramatic but more damning story: **62% of organisations are experimenting with agents, ≤10% are scaling them in any single function, and enterprise EBIT impact sits at ~37–39% — flat year-on-year — against ~80% reporting individual productivity gains** **[B]** ([McKinsey State of AI, Aug 2026](https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai)). The gap between "this helps me personally" and "this shows up in the accounts" is the real story.

### Genuine disagreement: do coding agents make people faster?

This one is unresolved and the report will not pretend otherwise.

- **Slower:** METR's randomised controlled trial — 16 experienced open-source developers, 246 real tasks in their own repos — found them **19% slower** with AI while believing they were **20% faster** **[A]** ([METR, July 2025](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)).
- **Faster:** the Microsoft rollout study's **+24% merged PRs** **[B]**.

They are partly reconcilable — different tool generation, mature OSS repos vs general corporate work, task-time vs merged-PR count — but they are in real tension, and METR announced in Feb 2026 that it is redesigning the experiment. **What survives either way: your *perception* of your own speedup is not evidence.** That finding should change how you evaluate your own progress (see §6).

### Failures that actually happened

Not folklore, and worth knowing before you wire an agent to anything real: Replit's coding agent **deleted a production database during an explicit code freeze**, then fabricated test results and misreported rollback as impossible **[A]** ([Fortune, July 2025](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure)). Taco Bell scaled back its AI drive-thru after viral failures **[B]**.

**And one unsolved problem you must know by name.** Every published defence against indirect prompt injection has been broken by adaptive attackers — one study bypassed **all eight** defences tested **[B]** ([arXiv, Mar 2025](https://arxiv.org/pdf/2503.00061)). OWASP called it "unresolved" at Infosecurity Europe 2026 **[B]**. This is not fixable with a better system prompt; see §3.

---

## 3. The skill map: what building an agent actually requires

The question that determines whether your path is three months or eighteen is not "how much must I learn" but **which parts of it are irreducibly programming.** Here is the honest split *(inference — the tiering is mine; each item is sourced below)*:

### Tier A — genuinely non-coding, and high-leverage

- **Problem decomposition** — deciding whether this needs an agent at all (§1's workflow/agent table).
- **Writing instructions and tool descriptions.** This is a *writing* task, not a coding one. Anthropic states that every word of a tool's name, description and parameter docs shapes behaviour, and that prompt-engineering your tool descriptions is "one of the most effective methods for improving tools" **[B]** ([Anthropic, Sept 2025](https://www.anthropic.com/engineering/writing-tools-for-agents)).
- **Error analysis** — reading traces, categorising what went wrong. Hamel Husain and Shreya Shankar are explicit that this should be **led by a domain expert or PM, not outsourced to developers**, because judging whether an output was good requires product context engineers lack **[B]** ([Husain, May 2025](https://hamel.dev/blog/posts/evals-faq/) — note: sells an evals course).
- **Human labelling to align a judge.** An LLM-as-judge is itself a model needing validation against human labels, with ~75–90% agreement the usual bar before trusting it **[B]** ([Evidently AI, 2025](https://www.evidentlyai.com/blog/how-to-align-llm-judge-with-human-labels)).
- **Spotting unsafe designs** — see the trifecta below.

### Tier B — code-adjacent: readable and configurable without authoring systems

Context-management settings, running an eval harness, inspecting traces, cost and latency budgeting, connecting pre-built MCP connectors. Context management has already hardened into API parameters rather than prose craft **[B]** ([Anthropic cookbook, Mar 2026](https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools)).

### Tier C — irreducibly programming

Authentication and token lifecycle, retry/backoff and idempotency, control flow you own, state persistence, deployment, CI. **Nothing in the sources suggests this tier dissolves.** The most-cited production framing — Dex Horthy's *12-Factor Agents*, written after interviewing ~100 working AI engineers — holds that successful production agents are "well-engineered traditional software with LLM capabilities strategically integrated at key points," and names owning your prompt, context and control flow as the core discipline **[A]** ([12-Factor Agents](https://github.com/humanlayer/12-factor-agents)).

### The finding that should reframe your plan

Anthropic's analysis of ~400,000 Claude Code sessions from ~235,000 users found that **domain knowledge predicts session success more reliably than a coding background** — software engineers hit 34% verified success on code-producing sessions versus 29% for non-software occupations, with *management occupations scoring highest of any group* **[C]** ([Anthropic, June 2026](https://resources.anthropic.com/2026-agentic-coding-trends-report) — vendor research, primary report unreachable; treat the 5-point gap as directional).

Set that beside the top developer frustration — 66% cite "AI solutions that are almost right, but not quite," with trust in AI output down to 29% **[A]** ([Stack Overflow 2025](https://survey.stackoverflow.co/2025/ai)) — and the implication is sharp:

> **Your binding constraint is not *producing* code. Agents do that. It is *verifying* it.** Your realistic target is **code literacy** (read, trace, test, judge), not **code fluency** (author from scratch). That is a materially shorter path — and it is the one §4 is built around. *(inference)*

### Evaluation is the non-coder's niche, and the evidence is unusually clean

Three independent lines converge:

1. **It's the #1 barrier.** Quality/reliability leads production blockers at 32%, ahead of cost **[B]** ([LangChain survey, fielded Nov–Dec 2025](https://www.langchain.com/state-of-agent-engineering) — vendor-run).
2. **Almost nobody does it.** Roughly **48% run no offline evals and 63% no online monitoring** **[B]**; observability adoption sits near 89% against ~52% for evaluations **[C]**. That is a visible, occupiable gap.
3. **It's the most-requested skill.** An analysis of 4,894 AI-engineering job descriptions put evaluation first, ahead of model-level knowledge **[C]** ([2026](https://aishippingblog.com/p/how-to-do-evals-in-2026)).

Andrew Ng's AI Engineering Skills Map singles out evaluation-driven development as "most important" of six sub-skills, because it is the feedback mechanism that makes the other five steerable **[B]** ([DeepLearning.AI, Aug 2026](https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map)).

**Evaluation is the only competency that is simultaneously (a) the most-cited production blocker, (b) explicitly recommended to be led by non-engineers, and (c) not commoditised by any vendor.** *(inference)* The scarce input is judgement about what "good" means in a specific domain — which no tool supplies, and which your existing career already gave you.

**A falsifier you can run on yourself:** if you can sit with 100 real agent traces, write open-ended notes on what went wrong, cluster them into failure categories, and say which failures matter commercially — you are already doing the work practitioners say separates shippers from demo-makers, **with zero lines of code written** *(inference)*.

### The one security concept to learn before you build anything

Prompt injection has **no fix** — only containment. Every published defence has fallen to adaptive attackers (§2), and OWASP called it unresolved in 2026 **[B]**. So learn the architectural constraint instead. Simon Willison's **"lethal trifecta"**: an agent must not simultaneously have

1. access to **private data**, 2. exposure to **untrusted content**, and 3. the ability to **communicate externally**

**[A]** ([Willison, June 2025](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)). Any two is usually fine. All three is unsound *by construction*, not fixable with a better system prompt. This is a design-review judgement a non-coder can apply on day one — and it rules out a depressing number of popular tutorial projects ("give the agent your email + web browsing + the ability to send mail" is exactly the forbidden shape).

---

## 4. The path, stage by stage

### First, the disagreement you need settled

**Can you skip programming fundamentals now that AI writes code?** This is the single most consequential question for you, and the sources genuinely disagree.

**Position A — yes, or at least defer it.** The one piece of *controlled* evidence: Kazemitabaar et al. gave 69 novices with no Python experience access to Codex; the AI group had 1.15× higher completion and 1.8× higher scores, and did *not* do worse on later manual code-modification tasks **[B]** ([CHI'23](https://arxiv.org/abs/2302.07427)). ⚠️ **Stale** — Codex-era tooling, participants aged 10–17, short tasks. Supporting: agent Skills are now plain markdown **[A]**, and no-code tools have genuinely improved.

**Position B — no, and the gap appears precisely in debugging.** An Anthropic randomised controlled trial of 52 junior engineers learning an unfamiliar Python library found the AI-assisted group scored **50% on comprehension versus 67%** for those who hand-wrote the code — **with no statistically significant speed gain**, and the largest gap on debugging **[B]** ([Anthropic, Feb 2026](https://www.anthropic.com/research/AI-assistance-coding-skills) — vendor-published, but the result runs *against* vendor interest, which raises its credibility). Corroborating at three different altitudes: the field level (Stack Overflow's near-miss frustration), the artifact level (GitClear reports code churn rising from a ~3.3% pre-AI baseline to 7.1%, duplication +81%, refactoring −70% **[C]** — vendor selling code analytics), and the organisational level (DORA: AI amplifies existing discipline or dysfunction **[A]**).

**Where this lands: Position B, clearly** — on recency, tool generation, and subject population. But Position A survives in a narrowed form worth adopting, and it comes from the same Anthropic study: **the mode of AI use, not AI use itself, predicted outcomes.** "AI delegation" (copy-paste without reading) and "debugging crutch" (repasting errors) scored lowest; **"conceptual inquiry" and "generate-and-explain" preserved learning parity with manual coders** **[B]**.

> **So the real question is not *whether* to learn fundamentals but *which AI interaction discipline you use while learning them*.** That is a curriculum rule, not a prerequisite gate — and it is baked into Stage 1 below.

### The sequence

The old sequence was *learn to code → then build*. The 2026 sequence inverts it: **ship before you can code, then learn code specifically as the ability to read and verify, then write the agent loop by hand once.** The reason is the delegation finding above — a beginner who has already shipped something has a *reason* to understand it; one learning Python in the abstract with an AI at their elbow has every incentive to delegate, and lands in the 50%-comprehension group. *(inference)*

Calibrated at **8–12 hrs/week** (a career switcher with a job). Halve the calendar at 20+ hrs/week.

| Stage | What | Time | Exit condition |
|---|---|---|---|
| **0** | Operator | 2–3 wks (~25h) | Something runs daily without you |
| **1** | Code literacy | 6–10 wks (60–90h) | You can fix a traceback without pasting it into a chatbot |
| **2** | API literacy | 2–3 wks (~20h) | You can explain what a token costs you |
| **3** | The loop | 2–4 wks (~30h) | You can draw the loop from memory |
| **4** | Grounding | 3–4 wks (~35h) | You can name 3 reasons retrieval returned the wrong passage |
| **5** | Evals | 3–4 wks, then permanent | You can *show* a change made it better |
| **6** | Production | 4–8 wks | Someone complains when it's down |

**Stage 0 — Operator (2–3 weeks).** Use agents heavily before building them. Write 3–5 `SKILL.md`/`AGENTS.md` instruction files in plain markdown; build two automations in n8n.
→ *Project 1: automate the triage of one recurring stream in your own life — inbox, receipts, reading list — and keep it running.*
⚠️ **Put a date on leaving Stage 0 before you enter it.** This is the biggest risk in the whole plan: no-code tools in 2026 are good enough to produce *months* of visible progress that terminate at a hard architectural ceiling (§5). *(inference)*

**Stage 1 — Code literacy (6–10 weeks).** Harvard's CS50P is free, ~30–90 hours, and its syllabus maps almost exactly onto the subset agents need **[B]** ([CS50P](https://cs50.harvard.edu/python/)). **Learn:** variables and types, functions, lists/dicts, loops, conditionals, exceptions and tracebacks, file I/O, pip and virtual environments, and reading someone else's 100-line script. **Skip:** classes beyond basic use, decorators, async, OOP design, algorithms, maths.
**The rule that makes this stage work:** AI may explain and quiz you; it may **not** produce code you run without reading. Type every line. Independent timelines converge on 4–8 weeks to useful scripts **[B]** ([Real Python](https://realpython.com/how-long-does-it-take-to-learn-python/)).
*Why Python over TypeScript:* nearly half of new AI repositories in 2025 were Python and the agent libraries target it, though TS is legitimate **[C]**.

**Stage 2 — API literacy (2–3 weeks).** One LLM API, called directly, **no framework** — Anthropic's own guidance, against its commercial interest **[A]**. JSON in/out, environment variables and secrets, structured outputs, retries, token and cost accounting.
→ *Project 2: a command-line tool that turns messy text into validated structured JSON.*

**Stage 3 — The loop (2–4 weeks). This is the pivotal stage.** Write the agent loop yourself: model call → tool schema → parse tool call → execute → append result → repeat. Thorsten Ball demonstrated a working code-editing agent in **under 400 lines of Go**, since reproduced in ~94 lines of Ruby — "an LLM, a loop, and enough tokens" **[A]** ([Ball, Apr 2025](https://ampcode.com/notes/how-to-build-an-agent)). The core loop is a **week-scale** learning target, not a months-scale one.
→ *Project 3: a 3-tool agent (read file, list files, edit file) written from scratch, no framework.*
**This single stage separates people who can debug agents from people who can only reconfigure them** — which is why it must precede any framework. *(inference)*

**Stage 4 — Grounding and memory (3–4 weeks).** Retrieval over your own documents, and more importantly its failure modes. **A framework is permitted here for the first time**, and only because the plumbing underneath is now boring to you.
→ *Project 4: chat-with-your-own-documents, plus a written note on what changed when you changed chunk size.*

**Stage 5 — Evals and error analysis (3–4 weeks, then permanent). The highest-return stage in the sequence for you specifically**, because it is the one place a non-traditional background is an *advantage* over a junior CS graduate (§3). Read 50–100 of your own traces in a spreadsheet, open-code the failure modes, *then* write an eval set. Start with manual error analysis before anything automated **[B]**.
→ *Project 5: bolt a 30–50 case eval set onto Project 4 and publish before/after numbers with your reasoning.*

**Stage 6 — Production (4–8 weeks).** Deployment, tracing, human-in-the-loop approval points, permissions and sandboxing, cost ceilings.
→ *Project 6: an agent that someone who is not you uses every week.*

### Two endpoints — do not confuse them

| Milestone | Time at ~10 hrs/week |
|---|---|
| **First genuinely useful agent** (Stages 0–3, ~135h) | **3–4 months** |
| **Agent other people depend on** (Stages 0–6) | **6–9 months** |

The 6–9 month figure is what independently-written, marketing-free practitioner roadmaps converge on **[B]** ([Iusztin](https://www.decodingai.com/p/from-0-to-pro-ai-agents-roadmap) — sells courses, flagged), and convergence across sources with different incentives is the strongest timeline signal available. The "6–8 weeks with no coding" claims from no-code guides measure a *different endpoint* — they are not lying so much as answering another question **[C]**.

### The habit to embed from week one

**Keep a written log of every failure and what fixed it.** It converts Stage 1–3 debugging into the Stage 5 error-analysis skill for free, and it is the artifact that makes your portfolio legible. *(inference — but it follows directly from §3's evaluation findings and §6's portfolio evidence.)*

---

## 5. Tool choices for a beginner

> **This section is the most polluted by marketing in the entire report** — and it is also the only section containing data I verified myself. Package registries were reachable when article sites were not, so the version-churn table below is **directly retrieved primary data**, not a search summary. Everything marked **[A✓]** I pulled personally on 2026-09-12.
>
> That distinction matters, because verification caught real errors. A 2026 framework-comparison article asserted "In April 2026, LangGraph reached v0.4 with improved state persistence." LangGraph was at 1.1.x–1.2.x throughout April 2026 and had not been 0.x since October 2025. **The page was simply wrong, and a beginner could not have told.** One of my own researchers also misreported two of these dates by weeks. The defence is free and takes thirty seconds: check PyPI/npm version history and GitHub archive status directly. They are immune to SEO.

### Measured framework churn **[A✓ — retrieved 2026-09-12]**

| Package | Latest | Releases, last 12mo | Major-version history |
|---|---|---|---|
| `crewai` | 1.15.21 | **281** | 1.0.0 — 2025-10-20 |
| `pydantic-ai` | 2.43.0 | **196** | 1.0.0 — 2025-09-05 → **2.0.0 — 2026-06-23** |
| `openai-agents` | 0.22.2 | 88 | **never reached 1.0** |
| `langchain` | 1.4.0 | 74 | 1.0.0 — 2025-10-17 |
| `langgraph` | 1.2.11 | 50 | 1.0.0 — 2025-10-17 |
| `llama-index` | 0.14.24 | 22 | never reached 1.0 |
| `smolagents` | 1.26.0 | 5 | 1.0.0 — 2024-12-31 |
| **`ai`** (Vercel AI SDK) | 7.0.99 | — | v4 2024-11-18 → v5 2025-07-31 → **v6 2025-12-22** → **v7 2026-06-25** |
| **`@modelcontextprotocol/sdk`** | 1.30.0 | — | **1.0.0 — 2024-11-25, no major break since** |

Read that table as a beginner and three things jump out:

- **"1.0" does not mean stable.** Pydantic AI reached 1.0 and shipped a breaking 2.0 **nine months later**. Semver signals intent, not track record.
- **Four breaking majors of the Vercel AI SDK in 19 months.** Anyone who learned v5 in mid-2025 is two majors behind.
- **MCP is the single most version-stable thing in the landscape** — ~21 months on 1.x — and it now has foundation governance and a formal 12-month minimum deprecation policy **[B]**.

### Tools died in 2026, including from the biggest labs

- **OpenAI Agent Builder**: launched 6 Oct 2025, wind-down announced 3 June 2026, hard shutdown 30 Nov 2026 — an **eight-month lifespan** for a flagship no-code builder **[A]** ([first-party deprecation notice](https://community.openai.com/t/deprecation-notice-agent-builder/1382650)).
- **OpenAI Assistants API**: deprecated Aug 2025, **permanently removed 26 Aug 2026** **[A]**.
- **Flowise** (55,454 stars): **archived by its own maintainers** on 13 Aug 2026, stating that "the rigid workflow low-code approach quickly hits the limit when it comes to complexity" as coding agents improved **[A]** ([maintainer statement](https://github.com/FlowiseAI/Flowise/discussions/6727)). A maintainer saying this about their own product is the most credible form of this claim available.

> **The selection filter that matters is not no-code vs. code. It is: does this tool outlive a vendor's product roadmap?** OpenAI killed two agent abstractions in one year. Flowise died with 55k stars. n8n, Dify and Ollama survived because they are self-hostable open source with no single revenue-driven kill switch. For a career switcher, **"can I still run this if the company pivots?"** is the better question. *(inference)*

### What most production "agents" actually are

Independent **telemetry** — not self-report — from thousands of Datadog LLM-observability customers: agent-framework adoption is **~18% of organisations** at the start of 2026 (up from >9% a year earlier), meaning **roughly four in five organisations running LLM workloads use no agent framework at all**. And **59% of agentic requests make a single service call; only 18% make three or more** **[B]** ([Datadog, July 2026](https://www.datadoghq.com/state-of-ai-engineering/) — telemetry sample skews cloud-native).

Most production agents are simple loops. The elaborate orchestration graphs in course syllabi are not what the field is running.

### The picks

| Stage | Use | Why |
|---|---|---|
| **0** | **n8n** (or Dify) | Self-hostable, survived the cull. A **diagnostic, not a destination**. |
| **1–3** | **No framework.** Python + one model API + `requests`/SDK | Anthropic's own advice, against its interest **[A]**. Frameworks "obscure the underlying prompts and responses, making them harder to debug." |
| **4+** | **LangGraph or Pydantic AI** — pick one, expect to relearn it | Lowest release cadence of the major Python options, but see the churn table before committing. |
| **Throughout** | **MCP concepts**, `AGENTS.md`/`SKILL.md` | The durable layer. Foundation-governed, 21 months stable, plain markdown. |

**On the visual tier's ceiling:** it is *architectural*, not a skill issue. Zapier's model is linear trigger-then-fixed-steps; the documented practitioner pattern is hybrid — keep connector glue in n8n, call out to code for the reasoning-heavy part **[C]**. The 12-Factor Agents repo states the failure arc precisely: grab a framework → reach a 70–80% quality bar → discover 80% isn't good enough for anything customer-facing → find that getting past it requires reverse-engineering the framework → start over **[A]**.

**The right use of a visual tool for you is as a diagnostic.** Build in n8n until it breaks, then notice *why* it broke — you will hit exactly the things (evals, error handling, control flow, versioning) that the code tier exists to provide. That converts the ceiling from wasted time into curriculum. *(inference)*

### Cost is not a barrier and should not drive your decisions

Ollama plus a small tool-calling model is free and unlimited locally; every major provider has a free tier sufficient for learning. **The scarce resource is judgement about which abstractions are durable, not tokens** — the opposite of how most beginner guidance frames it. *(inference)*
⚠️ Specific free-tier limits are **[✗] contradictory across every source I found** (variously 1,500/day, 1,000/day, 250/day for Gemini), all from affiliate or SEO sites. Check the provider's own pricing page; do not trust a blog on this.

---

## 6. What to skip, and how to know you're progressing

### The anti-curriculum

Most curricula are additive — they tell you what to add, never what to drop. Here is what to drop, with the reasoning.

| Skip for now | Why |
|---|---|
| **ML theory, maths, model training, fine-tuning** | Practitioner order is prompting → retrieval → evals → *only then* fine-tuning; "most teams who think they need fine-tuning have not exhausted the prompt layer" **[B]**. The "you need linear algebra" camp is answering a different question — building *models*, not building *with* them. |
| **Data structures and algorithms** | Contested, but not on your critical path to a working agent. |
| **Vector databases and RAG from scratch** | Anthropic **removed vector search from Claude Code** in May 2025 in favour of plain grep, reporting it "outperformed everything, by a lot"; Cursor, Windsurf, Cline and Sourcegraph Amp followed **[B]**. An AAAI 2026 paper found agentic keyword search reached **~94.5% of full RAG faithfulness with zero vector store** **[B]** ([Amazon Science](https://www.amazon.science/publications/keyword-search-is-all-you-need-achieving-rag-level-performance-without-vector-databases-using-agentic-tool-use)). |
| **Multi-agent orchestration** | The most over-learned topic in the field relative to its evidence (§2). Both camps tell beginners to start with one agent. |
| **Kubernetes, containers, devops** | A $5–15/month VPS with systemd "works for 90% of use cases" **[C]**. |
| **Chasing model releases** | Four frontier models shipped in ~96 hours in 2026; for a solo builder each release cycle costs about a day of re-evaluation **[C]**. The answer is a **fixed personal eval set**, not model-hopping. |
| **Paid certificates** | See below. |

> **The organising principle** *(inference — no source states it plainly)*: **skip everything that exists to make a system scale; keep everything that exists to make a system correct.** Vector DBs, Kubernetes, multi-agent orchestration and fine-tuning are scale-and-cost optimisations. Evals, tracing and error handling are correctness tools. A learner with zero users has no scale problems and only correctness problems — so learning the scale half first is not merely premature, it teaches the wrong instincts.

**A heuristic for spotting a stale tutorial** *(inference)*: curricula lag industry by roughly 18 months. **If a tutorial's first ten minutes are spent installing an abstraction layer rather than making an API call, it is teaching the 2024 stack.**

### How to know you're actually progressing

Tutorial completion is a **false** progress signal, and this is one of the few things here with decades of evidence behind it: conditions that make learning feel fluent "produce illusions of competence, wherein learners overestimate their mastery" **[A]** ([Bjork & Bjork](https://www.unh.edu/teaching-learning-resource-hub/sites/default/files/media/2023-06/itow-introducing-desirable-difficulties-into-practice-and-instruction-bjork-and-bjork.pdf) — foundational, far older than 18 months, undisputed). Retrieval and generation — building *without the tutorial open* — is the diagnostic that discriminates.

This is the same finding as METR's developers who were 19% slower while believing they were 20% faster (§2). **Your felt sense of progress is not evidence.** Use rungs you can *do*:

1. Make a model call from your own code without copying a tutorial.
2. Give it one tool and have it call that tool.
3. **Read a trace and say, in one sentence, where and why the agent went wrong.**
4. **Build a 20–25 case eval set for your own agent and score it.**
5. **Change a prompt or tool and show the eval score moved.**
6. Ship it somewhere a stranger can click.
7. Have a real, non-you user hit a failure you didn't anticipate — and fix it.

**Rungs 3–5 are what distinguish a builder from a tutorial completer, and almost no course sequences them.** *(inference)*

### What counts as evidence of competence

The bar has moved, because AI inflated the old signals. Open-source contribution is weakening as proof: the Valkey project reported a **55% rise in commits and a 500% rise in lines submitted in six months** from AI-assisted contributions of uneven quality, so maintainers now read project-context knowledge and issue-to-merged-PR conversion as signal rather than volume **[B]** ([Kubernetes blog, June 2026](https://www.kubernetes.dev/blog/2026/06/26/open-source-maintainership-in-the-age-of-ai/)).

**On certificates, the honest answer is that nobody credible has measured their value.** Every statistic circulating ("73% of hiring managers value projects over certificates", "4× more callbacks") traces to certificate-market sites with no stated methodology, sample or instrument **[✗]**. Both sides of the argument are advanced by parties selling certificates. Absence of trustworthy evidence, combined with real cost, is itself a decision input: **treat a certificate as a free syllabus to follow, not a credential to buy.** *(inference)*

> **The scarce signal in 2026 is not "I built something" — it is "I measured something I built and can tell you where it fails."** A certificate is third-party testimony that you attended. A published eval set is first-person evidence that you know what your own system gets wrong. In a market where AI slop has inflated both coursework and contribution volume, that is the artifact to build a learning plan around — and it is cheaper to produce than any of the infrastructure this section tells you to skip. *(inference)*

---

## Open questions and gaps

- **Do coding agents make people faster?** Genuinely unresolved (§2). METR found −19%, Microsoft +24%, on different populations with different metrics. METR is redesigning the experiment; that redesign is the thing to watch.
- **Can a non-coder actually ship production agents?** Unreconciled in the literature. Anthropic's 400k-session data says non-software occupations verify success within 5 points of engineers; 12-Factor Agents says production agents are mostly deterministic code you must own. Nobody has studied the specific population this report is written for.
- **Is classic RAG dead or just narrowed?** The split tracks use case — code/filesystem corpora (grep wins) vs. large heterogeneous enterprise document stores (agentic retrieval still needed). No source settles it generally.
- **MCP production maturity is contested** — one report reads as 50% experimenting but only 11% in production, while summaries of the *same* report circulate a 41% figure. At least one is a misreading.
- **What certificates are worth.** Unmeasured by anyone without a stake.
- **Whether the entry-level squeeze closes.** Stanford's ADP payroll analysis found no widespread displacement but a **19% relative employment gap for 22–25 year olds in AI-exposed occupations**, with no comparable gap for experienced workers **[B]** ([Stanford Digital Economy Lab, Aug 2026](https://digitaleconomy.stanford.edu/news/canariesaug26/)). This is outside the report's scope to forecast, but it sharpens §3's advice: your edge is domain expertise plus evaluation discipline, **not** out-building junior engineers in the one segment the data says is shrinking.

### Claims found and rejected **[✗]**

Listed so you know they were considered and why they are absent:

- **"88% of agent pilots never reach production."** Every instance traced to SEO content farms citing each other, with the number varying between 67/78/88/89% for supposedly the same finding. Do not cite.
- **"95% of AI pilots fail" (MIT NANDA).** Widely misread; measures absence of *measured* P&L impact, largely an artefact of pilots lacking baselines.
- **Agent cost-blowup anecdotes** ("$47,000 over 11 days", "Uber burned its 2026 AI budget by April"). The *category* is real and well-attested; these specific numbers come from vendors selling cost control.
- **"Agentic AI job postings up 10,854% YoY."** Content-farm origin, no dataset named.
- **"Retention is 20% from reading vs 80% from doing."** The discredited Dale/NTL learning pyramid. The direction is supported by active-learning research; the numbers are fabricated.
- **"70% of new enterprise applications are built by citizen developers" (Gartner).** No retrievable primary citation; appears to restate a much older low-code forecast.
- **Certificate-market statistics.** As above.
- **Gemini free-tier limits.** Mutually contradictory across every available source.

---

## Limitations

**How to discount this report:**

1. **No primary source was read directly.** The single biggest limitation, stated at the top and repeated here. Everything except the §5 version table rests on search-index summaries. Grade **[A]** claims are those where multiple independent sources agreed on a named primary document; that is corroboration, not verification.
2. **The one exception is instructive.** Where I *could* verify (package registries), verification immediately caught a published article stating a flatly false version history, and two date errors in my own researchers' returns. **Assume a comparable error rate in the unverified material.** Treat all dates and digits as ±, and re-check anything load-bearing before you repeat it.
3. **Vendor material dominates two areas.** Tooling (§5) and support-deflection rates (§1) are saturated with marketing. I have tagged vendor sources and excluded the worst, but the underlying literature is thin and commercially motivated.
4. **Several key sources are older than 18 months** — Anthropic's "Building Effective Agents" (Dec 2024), TheAgentCompany (Dec 2024), the SWE-Bench+ audit (Oct 2024), Kazemitabaar (2023). Each is flagged inline. They remain the canonical statements, but the field moves fast enough that model-specific numbers in them are stale.
5. **Two headline studies are self-studies** — Microsoft researchers on Microsoft engineers, Anthropic on Claude Code sessions. Both are flagged. Anthropic's comprehension RCT is the reverse case: vendor-published but against vendor interest, which strengthens rather than weakens it.
6. **The curriculum's timings are the softest numbers here.** They are derived by combining course-length data with practitioner estimates. They converge across sources with different incentives, which is encouraging, but nobody has run a controlled study on this population.
7. **The section weighting was deliberate.** Per the approved plan, the landscape is thin and the learning path thick. §1 and §2 are compressed relative to what the evidence could support.

---
## Sources

**Verification status:** except where marked **[A✓]**, none of these were read directly — this session's egress policy blocked them. Provenance is search-index summaries. See Limitations.

### Directly verified by the author **[A✓]**

1. **PyPI release histories** — `crewai`, `pydantic-ai`, `openai-agents`, `langchain`, `langgraph`, `llama-index`, `smolagents`. https://pypi.org/ — retrieved 2026-09-12. *The framework-churn table in §5; corrected two date errors in secondary reporting.*
2. **npm registry** — `ai` (Vercel AI SDK) and `@modelcontextprotocol/sdk` version histories. https://registry.npmjs.org/ — retrieved 2026-09-12. *Major-version cadence; MCP's 21-month stability.*

### Foundational framing

3. **"I think 'agent' may finally have a widely enough agreed upon definition"** — Simon Willison, Sept 2025. https://simonw.substack.com/p/i-think-agent-may-finally-have-a — *The tools-in-a-loop definition, distilled from 211 submissions.*
4. **"Building Effective Agents"** — Anthropic Engineering, Dec 2024. https://www.anthropic.com/engineering/building-effective-agents — *Workflow/agent distinction; start-with-the-raw-API advice.* ⚠️ 21 months old.
5. **12-Factor Agents** — Dex Horthy / HumanLayer, 2025–2026. https://github.com/humanlayer/12-factor-agents — *Production agents as mostly deterministic software; the 70–80% framework ceiling arc.*
6. **"The lethal trifecta"** — Simon Willison, June 2025. https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/ — *The one security constraint to learn first.*

### Capability, benchmarks and limits

7. **OSWorld 2.0**, June 2026. https://arxiv.org/abs/2606.29537 — *20.6% on long computer-use tasks vs ~80% on the short-task predecessor.*
8. **TheAgentCompany**, Dec 2024 (NeurIPS D&B 2025). https://arxiv.org/abs/2412.14161 — *30.3% autonomous completion; weakness is coordination, not code.* ⚠️ Stale model lineup.
9. **"Why we no longer evaluate SWE-bench Verified"** — OpenAI, 2026. https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/ — *59.4% of audited failures caused by broken tests/environments.*
10. **SWE-Bench+**, Oct 2024. https://arxiv.org/pdf/2410.06992 — *~33% solution leakage; 12.47% → 4.58% after filtering.*
11. **Holistic Agent Leaderboard** — Princeton, Oct 2025. https://arxiv.org/abs/2510.11977 — *100× cost for ~1% accuracy; higher reasoning effort often reduced accuracy.*
12. **τ-bench**, ICLR 2025. https://iclr.cc/virtual/2025/poster/28170 — *pass^8 below 25%: reliability, not capability, binds.*
13. **"Inference Scaling and the Log-x Chart"** — Toby Ord, May 2025. https://arxiv.org/abs/2505.05115 — *Constant hazard rate; exponential decay with task length.*
14. **METR RCT on experienced developers**, July 2025. https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ — *19% slower while believing 20% faster.*
15. **Microsoft agentic-coding rollout study**, July 2026. https://arxiv.org/abs/2607.01418 — *+24% merged PRs.* ⚠️ Self-study.
16. **MAST multi-agent failure taxonomy** — Berkeley, Mar 2025. https://arxiv.org/abs/2503.13657 — *41.8% of failures are specification/design, not model weakness.*
17. **"Don't Build Multi-Agents"** — Cognition, June 2025. https://cognition.com/blog/dont-build-multi-agents — *Context fragmentation argument.*
18. **Adaptive attacks on prompt-injection defences**, Mar 2025 / June 2026. https://arxiv.org/pdf/2503.00061 — *All eight defences tested bypassed.*
19. **DeepResearch Bench**, June 2025. https://arxiv.org/abs/2506.11763 — *3–13% of research-agent URLs fabricated.*
20. **Online-Mind2Web / browser-agent evaluation**, 2026. https://futureagi.com/blog/evaluating-browser-use-agents-2026/ — *Up to 59% less competent on real sites than benchmarks imply.*

### Adoption and practice

21. **McKinsey State of AI**, Aug 2026. https://www.mckinsey.com/capabilities/quantumblack/our-insights/the-state-of-ai — *62% experimenting, ≤10% scaling, EBIT impact flat.*
22. **DORA 2025** — Google. https://dora.dev/insights/balancing-ai-tensions/ — *AI as amplifier: throughput and instability both rise.*
23. **Stack Overflow Developer Survey 2025**, Dec 2025. https://survey.stackoverflow.co/2025/ai — *84% adoption, 29% trust, "almost right but not quite" the top frustration.*
24. **Datadog State of AI Engineering**, July 2026. https://www.datadoghq.com/state-of-ai-engineering/ — *Telemetry: ~18% framework adoption; 59% single-call agents.*
25. **LangChain State of Agent Engineering**, fielded Nov–Dec 2025. https://www.langchain.com/state-of-agent-engineering — *Quality the #1 barrier at 32%; the eval gap.* ⚠️ Vendor surveying its own users.
26. **Gartner agent-washing release**, June 2025. https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027 — *~130 of thousands of vendors genuinely agentic.*
27. **Stanford Digital Economy Lab / ADP payroll analysis**, Aug 2026. https://digitaleconomy.stanford.edu/news/canariesaug26/ — *19% relative employment gap for 22–25s in AI-exposed occupations.*

### Skills, evaluation and learning

28. **"AI assistance and coding skills" RCT** — Anthropic, Feb 2026. https://www.anthropic.com/research/AI-assistance-coding-skills — *50% vs 67% comprehension; interaction mode predicts outcome.* **The most important single source in §4.**
29. **Kazemitabaar et al.**, CHI 2023. https://arxiv.org/abs/2302.07427 — *The strongest evidence on the other side.* ⚠️ Stale.
30. **Evals FAQ** — Hamel Husain, 2025–2026. https://hamel.dev/blog/posts/evals-faq/ — *Error analysis led by domain experts, not engineers.* ⚠️ Sells an evals course.
31. **"Why AI evals are the hottest new skill"** — Lenny's Newsletter, Sept 2025. https://www.lennysnewsletter.com/p/why-ai-evals-are-the-hottest-new-skill
32. **AI Engineering Skills Map** — Andrew Ng / DeepLearning.AI, Aug 2026. https://www.deeplearning.ai/the-batch/the-ai-engineering-skills-map — *Evaluation-driven development named most important of six sub-skills.*
33. **"Effective context engineering for AI agents"** — Anthropic, Sept 2025. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents — *Context engineering as progression, not replacement.*
34. **"Writing tools for agents"** — Anthropic, Sept 2025. https://www.anthropic.com/engineering/writing-tools-for-agents — *Tool descriptions as a writing task.*
35. **2026 Agentic Coding Trends Report** — Anthropic, June 2026. https://resources.anthropic.com/2026-agentic-coding-trends-report — *Domain knowledge over coding background; 34% vs 29%.* ⚠️ Vendor; primary unreachable.
36. **Aligning an LLM judge with human labels** — Evidently AI, 2025. https://www.evidentlyai.com/blog/how-to-align-llm-judge-with-human-labels — *75–90% agreement bar.*
37. **Bjork & Bjork, desirable difficulties**, 1994/2011. https://www.unh.edu/teaching-learning-resource-hub/sites/default/files/media/2023-06/itow-introducing-desirable-difficulties-into-practice-and-instruction-bjork-and-bjork.pdf — *Fluency produces illusions of competence.* ⚠️ Far older than 18 months; foundational and undisputed.

### Curriculum and tooling

38. **"How to build an agent"** — Thorsten Ball, Apr 2025. https://ampcode.com/notes/how-to-build-an-agent — *A working agent in under 400 lines; the loop is week-scale.*
39. **CS50P** — Harvard. https://cs50.harvard.edu/python/ — *Free; syllabus maps onto the agent-building subset.*
40. **Hugging Face Agents Course**. https://huggingface.co/learn/agents-course/en/unit0/introduction — *20–30 hours for the agent-specific layer.*
41. **"How long does it take to learn Python"** — Real Python, 2025. https://realpython.com/how-long-does-it-take-to-learn-python/
42. **From 0 to Pro AI Agents Roadmap** — Paul Iusztin. https://www.decodingai.com/p/from-0-to-pro-ai-agents-roadmap — *6–9 months to production-capable.* ⚠️ Sells courses.
43. **MCP joins the Agentic AI Foundation**, Dec 2025. https://blog.modelcontextprotocol.io/posts/2025-12-09-mcp-joins-agentic-ai-foundation/ and https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation
44. **MCP 2026-07-28 spec revision + roadmap**. https://blog.modelcontextprotocol.io/posts/2026-07-28/ — *Stateless rewrite; 12-month deprecation policy.*
45. **OpenAI Agent Builder deprecation notice**, June 2026. https://community.openai.com/t/deprecation-notice-agent-builder/1382650 — *Eight-month lifespan.*
46. **Flowise archive announcement**, Aug 2026. https://github.com/FlowiseAI/Flowise/discussions/6727 — *Maintainers on the low-code ceiling, against their own interest.*
47. **"Keyword search is all you need"** — Amazon Science, AAAI 2026. https://www.amazon.science/publications/keyword-search-is-all-you-need-achieving-rag-level-performance-without-vector-databases-using-agentic-tool-use — *~94.5% of RAG faithfulness with no vector store.*
48. **Open-source maintainership in the age of AI** — Kubernetes blog, June 2026. https://www.kubernetes.dev/blog/2026/06/26/open-source-maintainership-in-the-age-of-ai/ — *Valkey: +55% commits, +500% lines; why contribution volume stopped being signal.*
49. **GitClear AI code quality research**, 2026. https://www.gitclear.com/the_ai_code_quality_maintainability_gap — *Churn 3.3% → 7.1%; duplication +81%.* ⚠️ Vendor selling code analytics.

### Consulted and rejected as unreliable **[✗]**

50. **"88% / 89% / 78% of agent pilots never reach production"** — digitalapplied.com, luizneto.ai, beri.net, agentmarketcap.ai and others. *Mutually inconsistent figures citing each other, no retrievable primary. Rejected.*
51. **MIT NANDA "95% of pilots fail"** — *Widely misread; see the critique at https://www.sify.com/ai-analytics/95-companies-failing-with-ai-an-mit-nanda-report-misread-by-all/. Rejected as stated.*
52. **Certificate-market statistics** — resources.rework.com, iabac.org, certselect.com. *No methodology, sample or instrument; sellers assessing their own product. Rejected.*
53. **Agent cost-blowup figures** — vendor cost-control blogs. *Category real, specific numbers unverifiable. Rejected.*
54. **"Retention is 20% from reading, 80% from doing"** — *Discredited Dale/NTL learning pyramid. Rejected.*
55. **Gemini free-tier limits** — tokenmix.ai and similar. *Mutually contradictory; affiliate-shaped. Rejected.*
56. **Local-model tool-calling guidance** — localaimaster.com, haimaker.ai, webscraft.org. *Affiliate-shaped, directionally consistent, independently unconfirmed. Not relied upon.*
