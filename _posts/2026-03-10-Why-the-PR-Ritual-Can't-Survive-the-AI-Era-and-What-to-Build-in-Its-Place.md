---
title: "Why the PR Ritual Can't Survive the AI Era and What to Build in Its Place"
date: 2026-03-10
layout: wide-no-author
featured: true
excerpt: "AI coding tools have accelerated output, but PR review time has increased 91%. The fix isn't faster review. It's decomposing the various functions that the PR serves and rethinking how each can be better served in the new way of working. Human effort needs to shift from inspecting code after it's written to defining intent before code exists."
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/pr-evolution/pr-evolution-flow.png"
categories:
  - AI
tags:
  - ai
  - ai-coding-agents
---
You're an engineer. You're using AI coding assistants (Claude Code, Cursor, Copilot, whatever your weapon of choice is) and you're writing code faster than you ever have. Your workflow is evolving toward orchestrating several agents at once. I'm somewhere between Stage 6 and Stage 7 of Steve Yegge's [AI-assisted coding journey](https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04), and if you're reading this, you're probably somewhere nearby.

You'd expect all that speed to show up somewhere. More features shipped, faster releases, shorter cycles.

[Faros AI](https://www.faros.ai/blog/ai-software-engineering) research found that developers on teams with high AI adoption complete 21% more tasks and merge 98% more pull requests, but PR review time has increased by 91%, revealing a critical bottleneck: human approval.

The [2025 DORA report](https://dora.dev/research/2025/dora-report/) lands on the same diagnosis from a different angle: AI boosts individual throughput but hurts delivery stability. One line from the report that stuck with me:

> AI's primary role in software development is that of an amplifier. It magnifies the strengths of high performing organizations and the dysfunctions of struggling ones.

We've made the input faster but we haven't made the filter wider. And the gap is growing.

## The Obvious Fix (And Why It's Not Enough)

The industry's response? Throw AI at the review problem too.

And it's happening. [PullFlow's 2025 report](https://pullflow.com/state-of-ai-code-review-2025) shows AI code review adoption is real: CodeRabbit has reviewed over 632k PRs, GitHub Copilot Code Review crossed 561k, Google Gemini hit 174k.

These tools help. They catch typos, flag vulnerability patterns, spot style issues, check test coverage, surface performance issues and catch other common error patterns.

But they have a ceiling. Most AI review tools see the changed lines and a narrow window of surrounding code. They can't reliably reason about system architecture, cross-repo dependencies, or the design decisions behind a change.

Use them. Absolutely use them. But deploying AI review inside the existing PR model is like adding a faster inspector at the end of a factory line. It helps. It doesn't address the design of the line itself.

## What Is a PR Actually Doing?

Before we redesign the process, we need to understand what it's doing. We talk about "code review" like it's one thing. It's not. When you look at what actually happens during a review, nine distinct functions emerge:

1. **Style and Consistency**: Enforcing formatting, naming conventions, code structure and adherence to team patterns.
2. **Defect Detection**: Finding bugs, logic errors, functional issues.
3. **Security**: Vulnerability scanning, secret detection, dependency CVE checks, auth pattern validation.
4. **Architecture Validation**: Does this change fit the broader system design? Is the right service owning this logic? Will this coupling create problems for us six months from now?
5. **Knowledge Transfer**: Learning about the codebase, APIs, and team conventions through the act of reviewing and being reviewed.
6. **Accountability**: The "someone will read this" effect. The [Hawthorne Effect](https://www.statsig.com/perspectives/hawthorne-effect-observation-behavior) applied to software engineering. It makes you write cleaner code before you submit.
7. **Shared Ownership**: More than one person understands each change. Bus factor of 1 means one resignation away from chaos.
8. **Compliance and Audit Trail**: Documented proof that changes were reviewed, satisfying regulatory requirements.
9. **Design Improvement**: The "have you considered...?" feedback where reviewers propose better approaches, spot performance implications and suggest alternative solutions.

Nine functions. One ritual. And when you look closely, three things jump out.

**The functions that consume the most reviewer time are the most automatable.** Style, basic defects, known security patterns. Linters, SAST tools, and AI review tools handle these deterministically or nearly so. If humans on your team are still commenting on formatting in PRs, that's not a review problem. It's a tooling gap.

**The most valuable functions are fundamentally human but the PR serves them poorly.** Knowledge transfer, accountability, shared ownership. These matter deeply. But a reviewer clicking "Approve" doesn't mean they absorbed the design reasoning or learned anything about the system. Pair programming, walkthroughs, and design discussions do this better than async PR comments ever have.

**The most impactful function happens at the wrong time.** Architecture validation (the one that catches the most expensive mistakes) only happens after code is written. If the architecture is wrong, the feedback is too late and the rework is costly.

Now consider what happens as agents generate more of the code. Michael Truell, CEO of Cursor, recently [shared](https://cursor.com/blog/third-era) that 35% of the PRs they merge internally are created by autonomous agents in cloud VMs. He expects the vast majority of dev work to be done by agents within a year.

If that trajectory holds and I think it will, the human-dependent functions of the PR don't just stay underserved, they become impossible to fulfill through the current ritual.

## The Inversion

The code review model we inherited was built for a world where writing code was the hard, expensive part. Code was the scarce artifact, so you inspected it carefully. That made sense.

AI has flipped what's scarce and what's abundant.

Code is cheap now. Hundreds of lines in minutes. What's still expensive is **intent**: knowing what the code should do, why, and how it fits the bigger picture. Design decisions, architectural trade-offs, business logic. That's still entirely human work, and it's where the real quality risk lives.

So review needs to flip too. Instead of concentrating human effort at the end, reading diffs, concentrate it at the beginning: defining and validating intent.

| | Old Model | Inverted Model |
|---|-----------|---------------|
| **Intent** | Defined loosely (Jira ticket, conversation) | Defined carefully (specs, ADRs, quality standards) |
| **Code** | Written carefully by humans | Generated quickly by AI |
| **Verification** | Human reads the diff | Machines verify against specs |
| **Human effort** | Heaviest at the end | Heaviest at the beginning |

Review doesn't disappear. It shifts upstream.

[Ed Wentworth](https://ed-wentworth.medium.com/rethinking-code-review-in-the-age-of-agentic-ai-a26e1e30d9c2) puts it well, "The future of software assurance lies not in inspecting the code that AI agents produce, but in governing the process that produces it."

But you can't flip this overnight. You have to take apart the PR monolith piece by piece, moving each of those nine functions to the mechanism that actually serves it best. I think of this as "Unbundling Code Review."

## Unbundling the PR

I think about this in three stages.

### Stage 1: Optimize the Existing Process

Start here. This is where the quickest wins live.

**Close the automation gaps.** If your reviewers are commenting on formatting, naming conventions, or missing null checks, that's not review. It's a tooling gap. Enforce linters and formatters as blocking CI gates, not suggestions. Add static security analysis, dependency scanning, test coverage thresholds. Make them authoritative: if the check fails, the PR cannot merge. Period. If you're running a mature DevOps practice, you've probably got some of this in place already.

**Tier the review by risk.** A typo fix and an auth rewrite shouldn't go through the same process. But on most teams, they do.

[Ship/Show/Ask](https://martinfowler.com/articles/ship-show-ask.html) breaks this habit:

- **Ship**: merge directly for trivial, low-risk changes backed by automated checks.
- **Show**: merge immediately, open a PR for team awareness.
- **Ask**: traditional review before merge. New architectural patterns, security-sensitive code, public APIs, database migrations.

The power isn't in the tiers. It's in the conversation they force: *what actually needs review here?* Most teams have never had that discussion.

This tiering doesn't have to stay manual either. [Meta built Diff Risk Score](https://engineering.fb.com/2025/08/06/developer-tools/diff-risk-score-drs-aware-software-development-meta/), an ML model that scores the risk of each code change, gating the riskiest diffs and letting low-risk ones flow. An [open-source replication (DRS-OSS)](https://arxiv.org/html/2511.21964v2) found that gating just the riskiest 30% of commits prevented 86.4% of defect-inducing changes. The tooling in this space is early, but the direction is clear.

**Make changes smaller.** Large PRs are exponentially harder to review. The [SmartBear/Cisco study](https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/) found effectiveness drops sharply past 200-400 lines. AI tools make this worse by generating big changes in one shot.

Stacked PRs are one structural fix: small, dependent changes, each 50-200 lines, each reviewable on its own. [Google's engineering practices](https://google.github.io/eng-practices/review/developer/small-cls.html) lay out several decomposition strategies worth studying. Tools like [Graphite](https://graphite.com/blog/stacked-prs) make this easier in practice. And it amplifies your AI review tools. They work dramatically better on a focused 200-line diff than on a 2,000-line monster.

**What Stage 1 relocates:** Style, Defect Detection, and Security move to automated CI gates. Architecture Validation and Design Improvement stay in the PR but benefit from smaller diffs. Knowledge Transfer, Shared Ownership, and Accountability? Still untouched. Still riding the PR ritual.

### Stage 2: Encode Quality at the Source

Stage 1 improves the process. Stage 2 starts to change it.

The core idea: instead of waiting until PR time to enforce quality standards, encode them as instructions that run *during development*, while the developer still has full context.

Teams had a version of this before AI assistants. EditorConfig, custom linter rules, pre-commit hooks, IDE templates. They worked for what they could express, but they were limited to formal patterns. "Use consistent indentation" is easy to encode. "This API endpoint should follow our rate-limiting pattern, which we adopted after the incident in Q3" is not.

That's what's changed. Every major AI coding platform now supports natural-language project instructions:
- [Agents.md](https://agents.md/)
- [Claude Code Skills](https://code.claude.com/docs/en/skills)
- [Cursor Rules](https://cursor.com/docs/context/rules)
- [GitHub Copilot Custom Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions)

The pattern is the same across all of them: **team-shareable, version-controlled, natural-language instructions that encode institutional knowledge at the point of code creation.**

The difference from a linter is qualitative. A linter catches "unused variable." A dev-time quality gate catches "this endpoint doesn't follow our rate-limiting pattern — see ADR-042." Context, conventions, nuance. Stuff that formal rules can't touch.

And here's where it gets interesting. You can build a self-learning loop where review comments feed back into the rules automatically. Pattern keeps surfacing in reviews? Encode it. Issue slips through to production? Write a rule that prevents it next time. The flywheel compounds: you accumulate more standards, and the agents that interpret them keep improving with every model generation.

**What Stage 2 relocates:** Defect Detection and Design Improvement shift further upstream, from PR-time review to creation-time prevention. But the deeper change is to Knowledge Transfer and Shared Ownership. When a senior engineer's standards are encoded as version-controlled rules, those rules become passive knowledge transfer. Every developer absorbs conventions by working alongside an AI that already knows the team's patterns. Shared rules mean shared understanding of *why*, not just *what*.

Accountability transforms too. "Someone will read this" doesn't disappear. It becomes "the system will verify this." For routine changes, the latter is more consistent and more honest.

This changes the role of the senior engineer. In the old model, they're a serial quality gatekeeper whose expertise reaches one PR at a time. When they encode their standards as reusable rules, those standards run in parallel across every developer, every line of code, in real time. The senior engineer stops catching fish and starts building nets.

And when they leave the team? Their standards don't leave with them. Encoded as version-controlled artifacts, institutional knowledge persists, teaching and enforcing even after its author has moved on.

### Stage 3: The Full Inversion

Stage 2 encodes *how* code should be written. The natural next step is encoding *what* should be written: the intent itself. Human effort goes to the front: specifications, design decisions, architectural intent. Systems handle verification downstream.

**Spec-driven development** is gaining traction with tools like [AWS Kiro](https://kiro.dev/blog/general-availability/) and [GitHub Spec Kit](https://github.com/github/spec-kit). Write specifications first, let AI generate the implementation, run automated checks to verify the code against the spec. The criticism is that this is waterfall reborn. The distinction however is between heavyweight up-front specs and lightweight, iterative ones. We're talking ADRs, behavioral acceptance criteria, Given/When/Then scenarios. Not 50-page documents. They evolve with the implementation. Closer to TDD than waterfall in my opinion.

**What Stage 3 relocates:** Architecture Validation and Design Improvement finally leave the PR. They move upstream to the spec review, where they should have been all along. By the time code exists, architectural decisions have already been debated and approved. The PR becomes a verification event, not a design discussion.

Knowledge Transfer and Shared Ownership complete their transformation here. In this model, code is generated, potentially disposable, certainly regenerable. What matters is understanding the *intent*: why this design decision, what constraints the spec captures, what trade-offs we accepted. Shared ownership stops meaning "two people read this diff" and starts meaning "two people understand why we chose this architecture." The bus factor no longer measures who can read the code. It measures who understands the decisions that produced it.

Accountability completes its shift. The spec author owns the intent. The automated pipeline owns correctness. The evidence trail — test results, security scans, coverage, contract verification — forms an **evidence pack** documenting what was verified and how. Together, these signals provide stronger assurance than any single human approval.

**Post-merge monitoring** completes the picture. Feature flags, canary deployments, observability, shifting quality assurance from pre-merge review to production verification. Charity Majors nails it: ["You don't know if code works until it's running with enough instrumentation to see what it's actually doing."](https://charity.wtf/2023/09/20/llms-demand-observability-driven-development/)

**The North Star.** Where does this end? Martin Kleppmann [offers a provocative answer](https://martin.kleppmann.com/2025/12/08/ai-formal-verification.html): if AI can generate both code *and* a formal proof of correctness, reviewing AI-generated code becomes as unnecessary as reviewing compiler output. We don't read assembly because we trust the compiler. The day we trust the spec-to-code pipeline the same way — specifications in, verified implementation out — the PR as we know it is done.

We're not there yet. But the direction is unmistakable.

### Putting It Together

Here's how each of the nine functions migrates and accumulates across the three stages:
<p class="image-with-caption">
  <img alt="A matrix showing how nine PR functions — Style & Consistency, Defect Detection, Security, Architecture Validation, Design Improvement, Knowledge Transfer, Accountability, Shared Ownership, and Compliance & Audit — evolve across three stages: Optimize, Encode, and Invert. Each cell describes how the function shifts from manual PR review toward automated CI gates, encoded dev-time rules, and spec-driven verification." src="/assets/images/pr-evolution/pr-evolution-flow.svg" style="max-width: 950px;">
  <figcaption style="text-align: center;">Unbundling the PR: How nine code review functions migrate across three evolution stages</figcaption>
</p>

## Where to Start

The PR isn't going away tomorrow. But the nine functions it bundles are already migrating to better homes, and the teams that recognize this early will stop drowning in review queues and start spending their time on the work that actually matters.

If you do one thing this week, **measure.** What percentage of your cycle time is review? How many review comments are about things a machine could catch? Most teams don't know. The answers will tell you which stage to start with.

If the answer is "most of our comments are about formatting and style," you've got Stage 1 work to do. Close the automation gaps. Make linters and SAST tools authoritative, not advisory.

If you've already automated the obvious, start encoding. Take your three most common review comments and turn them into a Skill, Rule, or Custom Instruction. Run it for a sprint. See if those comments stop appearing. That's Stage 2's flywheel starting to turn.

If you're feeling ambitious, try classifying your next sprint's PRs as Ship, Show, or Ask. The conversation it forces — *what actually needs review here?* — will teach you more about your process than any metric.

None of these require a process overhaul. All of them are steps on the trajectory this post describes: moving human effort from inspecting code after it's written to defining intent before code exists.