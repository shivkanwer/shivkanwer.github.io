---
title: "Proximity to Change"
date: 2026-04-04
layout: wide-no-author
featured: true
excerpt: "AI is changing how software gets built, but most advice about keeping up stops at 'stay informed'. That's not enough. This article explores Proximity to Change, three distinct ways software developers can adapt to AI: knowing what's happening, doing real work with it, and knowing when AI is wrong. It covers the traps that catch even talented developers off guard, and what to actually do about it."
header:
  teaser: "https://raw.githubusercontent.com/shivkanwer/shivkanwer.github.io/main/assets/images/proximity-to-change/proximity-to-change.png"
categories:
  - AI
tags:
  - ai
  - ai-coding-agents
---

AI is changing how software gets built. That much is obvious. What's less obvious is why some developers adapt naturally while others, equally talented, find themselves caught off guard.

This shift is structurally different from the ones that came before. Every previous wave (cloud, containers, mobile, microservices) had its "adapt or die" rhetoric, and every time most developers adapted just fine through normal work. But those shifts changed where your code ran, how it was packaged, or what platform it targeted. None of them changed the core act of writing software. AI does. And I don't know with certainty where this leads. Neither does anyone else, regardless of what they tell you.

What I do know is that the developers who navigate this best will be the ones who understand and act on both of these facts: that the profession will endure, and that the work inside it is genuinely changing.

I've been thinking about this through a lens I call **Proximity to Change**. The idea is simple: the closer you stay to what's changing, the easier it is to adjust. The further away you drift, the harder the catch-up becomes. But "staying close" is not as straightforward as it sounds, and not all forms of closeness are created equal.

## The Three Kinds of Proximity

Most advice about keeping up with AI falls into one bucket: stay informed. Read the newsletters, watch the demos, follow the right people on social media. Awareness matters, but it's the weakest form of proximity, and on its own, it can actually make things worse. I think there are three distinct ways you can be "close" to change, and they're not interchangeable.

### 1. Informational Proximity — Knowing What's Happening

This is the easiest level to reach. You read about new models, you follow AI Twitter, you watch conference talks. You know what GPT, Claude and Gemini can do. You've seen the demos.

This is necessary. You can't engage with something you don't know exists. But informational proximity alone creates a trap I call **confident obsolescence** where you feel prepared because you're well-informed, while your actual hands-on skills quietly fall behind. You're reading the menu, not eating the food.

Psychologists have a name for the underlying mechanism: the illusion of explanatory depth. We consistently overestimate how well we understand things we've only read about. Knowing what a coding agent can do is not the same as knowing what it actually does when you point it at your codebase.

Most "keep up with AI" advice stops at informational proximity, and that's a problem.

### 2. Practical Proximity — Doing Real Work With It

This is where adaptation actually happens. Not just reading about a coding agent but using one on a real task and seeing where it helps and where it hallucinates.

Hands-on practice creates a reinforcing loop: you try a tool, you build some skill, that skill gives you confidence, and that confidence makes you try more. It compounds. But the same loop runs in reverse — avoid the tools, fall behind, feel less confident, avoid them more.

The good news is that the entry point is low. You don't need to overhaul your workflow. Pick one real task this week and try it with an AI tool you haven't used before. Something where you'll actually notice whether the AI helped or hurt. A refactoring task or a migration task works well for this because the code already works, giving you a clear baseline to evaluate the AI's output against. A developer who has struggled with one real AI tool knows more than someone who has read about twenty.

### 3. Evaluative Proximity — Knowing When AI Is Wrong

This is the one most people miss, and it might be the most important.

When you use AI tools regularly, you get faster. The output looks good. You feel productive. But here's the hidden risk: **fluency is not the same as competence**. AI doesn't reason about your code. It predicts what correct code probably looks like, based on patterns in its training data, and most of the time that prediction is right. But when your problem diverges from what the model has seen, the output doesn't break visibly. It fails silently, in ways that look exactly like correct code.

Over time, the ease of the output can quietly erode your ability to catch these cases. I call this **fluent incompetence**, you're using the tools effectively, shipping faster, feeling good about it, but slowly losing the judgment muscle that catches subtle bugs, bad architectural choices, or AI outputs that look correct but aren't.

This matters because the cases where AI fails are not random. They cluster in specific, predictable places:

- **Edge cases and boundary conditions** — AI handles the happy path well; it's the corner cases where it falls apart
- **Domain-specific constraints** — regulatory requirements, security, timing constraints in embedded systems
- **Novel combinations** — using multiple APIs together in ways that don't appear often in training data
- **Recently changed APIs** — libraries updated after the model's training cutoff
- **Concurrency and state management** — code that requires reasoning about execution order across threads or services

The fix isn't to avoid AI tools. It's to know where to focus your attention. Not every line of AI-generated code needs the same level of scrutiny. But anything touching authentication, data validation, state management, or external contracts deserves careful, skeptical review. 

Those are the three levels. Most developers can tell which one they're weakest on. What's harder to see are the patterns that quietly keep you there.

## The Traps

### The Complacent Middle

Here's a distinction that took me a while to see clearly. There are two ways to be in the middle of AI adoption, and they look similar from the outside but carry very different risk profiles.

The **deliberate middle** is a developer who uses AI tools regularly, maintains solid general skills, stays moderately current, and applies good judgment. They're not at the bleeding edge, and they're not running wild experiments every week. But they're engaged, they're thoughtful, and they're paying attention. This is probably the most rational position for a lot of working engineers, especially those under delivery pressure or working in domains where current AI tools perform unevenly.

The **complacent middle** is different. This is the developer who uses Copilot casually, tried ChatGPT a few times, and feels generally "up to speed." They're not panicking, and they're not deeply engaged. They're coasting. They've built just enough familiarity to stop feeling urgency, but not enough depth to notice what they're missing.

The risk of the complacent middle isn't that you'll wake up obsolete one morning. It's subtler than that. AI capabilities are advancing in bursts, not smooth curves. GPT-3.5 to GPT-5.4, autocomplete to coding agents, 2% to 80% on real-world coding benchmarks, in under three years. When the next burst happens, the complacent middle is the group least prepared to absorb it. Not because they lack information, but because they lack the muscle memory of engaging deeply with new capabilities.

To be clear, many developers are in the middle not by choice but by constraint like sprint pressure, legacy codebases, domains where AI tools underperform, or organizations where AI tools are restricted due to security or compliance reasons. That's not complacency. That's the reality of the job. The question is whether you can find even small pockets of deliberate engagement within those constraints, even if that means experimenting with free-tier tools on a side project outside work.

### The Identity Trap

There's a deeper risk than falling behind on tools. When AI starts displacing tasks that developers see as central to who they are — writing elegant code, debugging complex systems, architecting solutions from scratch. That displacement can erode the willingness to engage at all.

This creates a vicious cycle: AI threatens your identity, so you avoid it, which widens the skill gap, which makes your identity feel more threatened.

By the time the market sends a clear signal, the recovery investment is steep. Your skills feel outdated, the job market has shifted. Not impossible to catch up, but steep.

The way through this isn't to ignore the identity question. It's to reframe what it means to be a developer. The shift isn't from "I write code" to "I'm obsolete." It's from "I write code" to "I direct systems, make judgment calls, and solve problems using whatever tools exist." That identity survives tool transitions because it's anchored to what you think and decide, not to the specific medium you work in.

But here's the thing, that reframe only sticks if you have real experience to anchor it to. You can't genuinely adopt the identity of "I direct systems" if you've never actually directed an AI system through a real task. The identity shift follows the hands-on experience. It doesn't precede it. Which is another reason practical proximity matters.

### Shifting the Burden

There's a structural trap worth naming that sits underneath the others. When you use AI tools primarily for speed, to ship faster, to clear the backlog, to hit sprint targets, you're solving a real problem. But you're solving it in a way that can quietly undermine the deeper solution.

The deeper solution is building the kind of judgment that makes you genuinely good at working with AI — knowing when to trust it, when to override it, when to throw its output away. That judgment comes from effortful practice, from struggling with hard problems, from the slow accumulation of experience that tells you something is wrong before you can articulate why.

Every hour spent accepting AI output without evaluation is an hour not spent building that judgment. And unlike AI fluency, which can be built in weeks, judgment is a slow-accumulating asset that takes years to build and is slow to recover once it erodes.

The fix isn't to refuse the speed. It's to invest in judgment *alongside* the speed, deliberately and consistently, even when it feels slower. Treat the judgment-building as non-negotiable, not as a nice-to-have that gets squeezed out when deadlines press.

## What to Actually Do on Monday Morning

### If You Haven't Started Yet

Start small. Pick one real task you'd normally do yourself and try it with an AI tool. See what works and what doesn't.

But pick the task intentionally. The best starting tasks have a clear "right answer" you can check against like refactoring existing code, writing tests for a module you already understand, or migrating a function from one pattern to another. These tasks let you evaluate the AI's output against your own knowledge, which builds evaluative skill from day one. Avoid starting with greenfield tasks where you have no baseline to judge against.

Pay attention to three things: Where did the AI save you real time? Where did you have to correct it? And was there anything in the output you almost accepted but caught on closer inspection?

One task a week is enough to start. The goal isn't to transform your workflow overnight. It's to build the habit of being a beginner at something, because that habit is the actual protective factor.

### If You're Already Using AI Tools

Good. Now pressure-test your judgment.

Start keeping a lightweight log of the cases where AI output looked right but wasn't. Over time, you'll start seeing patterns. Those patterns are your personal failure-mode taxonomy, and they're worth more than any general advice because they're calibrated to your specific codebase, language, and domain.

Some failure modes to watch for specifically:

- **Plausible but outdated**: the AI uses a deprecated API or an old version of a library's interface
- **Correct syntax, wrong semantics**: the code runs but doesn't actually do what you intended, especially around state management or async operations
- **Missing constraints**: the happy path works, but validation, error handling, or security properties you'd normally include are absent or superficial
- **Confident nonsense**: the AI invents a function, flag, or configuration option that doesn't exist but sounds like it should

### If You Lead a Team

This one matters most, because individual developers can only do so much if their organization doesn't create space for it.

The structural barriers are real - sprint pressure, legacy codebases, no protected time for experimentation. So here's what I'd actually recommend:

**Start with the business case.** If you need to justify experimentation time to your manager, frame it as risk reduction: "We're going to be using these tools whether we invest in learning them well or not. The question is whether we learn through deliberate practice or through production incidents." That framing usually lands.

**Protect 10-20% of sprint capacity for AI experimentation in the first cycle.** Not as a permanent allocation, but as a time-boxed experiment with a clear evaluation point. "For the next two sprints, we're dedicating one day per sprint to AI-assisted development on real backlog items. At the end, we'll measure what we learned and decide whether to continue." That's a concrete proposal a skip-level can say yes to.

**Build evaluation into the workflow, not alongside it.** Add AI-output scrutiny as an explicit quality gate in code review. When a PR includes AI-generated code, flag it. Not to penalize it, but to create the organizational habit of evaluating it with the same rigor you'd apply to any code you didn't write yourself. Over time, this builds team-level evaluative skill.

**Make judgment visible.** When someone catches an AI error in review, celebrate it. When a post-mortem reveals that AI over-reliance was a contributing factor, name it without blame. The goal is to make the judgment stock (the team's collective ability to catch AI mistakes) a thing that's measured and valued, not just assumed to exist.

**Track learning rate, not just output velocity.** This is the hard one. Sprint velocity is easy to measure. Learning rate is not. But you can approximate it: How many new AI capabilities did the team try this quarter? How many AI-specific failure modes has the team documented? How quickly does a new team member get up to speed on the team's AI workflows? These are proxies for adaptive capacity, and they matter.

The organizations that will adapt fastest are not the ones where individual developers are the most motivated. They're the ones where the organizational structure makes adaptation the path of least resistance rather than something that competes with delivery for attention.

## The Obvious Objection

There's a fair counter-argument to everything I've written: *"Developers should learn their tools, think critically, and keep learning throughout their careers. That's been true forever. You don't need a new framework for that."*

That's right. The underlying advice — engage with new tools, maintain your judgment, don't get complacent — is not new. It's the same advice every thoughtful senior developer has been giving for decades. And if that's all you take from this article, you'll be fine.

What this article adds, I think, is a diagnostic. Most developers who are stuck don't know *where* they're stuck. "Stay current" is too vague to act on. "Stay close to the change" is too vague to act on. But "you have strong informational proximity and weak practical proximity" is specific enough to point at a concrete next step. And "you have strong practical proximity but weak evaluative proximity" points at a different next step entirely.

## The Habit That Actually Matters

The specific AI tools you learn today might not be the ones that matter in a year. AI capabilities can shift fast, and the landscape is genuinely unpredictable. That's okay.

What transfers across every tool generation is the **habit of adaptation**, the muscle memory of picking up unfamiliar things, struggling with them, and integrating what works. The developer who has gone through the cycle of confusion, experimentation, and competence a few times will handle the next cycle with less friction, regardless of which tool it involves.

It's not about mastering any particular AI tool. It's about staying comfortable with being uncomfortable. That's the skill that compounds.

## The Bottom Line

Proximity to change isn't about subscribing to more newsletters or attending more conferences. It's about three things, in order of importance:

1. **Protect your judgment** — the ability to tell when AI is wrong is more valuable than the ability to use AI when it's right. This is the skill that ages best.
2. **Do real work with it** — one task with an unfamiliar AI tool teaches more than twenty articles about AI. Start with tasks where you can check the output against your own knowledge.
3. **Know what's happening** — but don't mistake this for actually adapting. Information without practice is preparation theater.

The developers who thrive through this won't necessarily be the ones who adopted AI first. They'll be the ones who built the habit of engaging with the unfamiliar and kept their judgment sharp along the way.

Pick a task. Something real. Notice what the AI gets right, what it gets wrong, and what it gets wrong in a way that almost fooled you. That last category is where the learning lives.

Go from there.