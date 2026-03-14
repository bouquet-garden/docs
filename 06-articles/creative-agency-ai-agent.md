---
title: "I Run a Creative Agency With One AI Agent. She Costs $3/Day and Managed Our Alibaba Sourcing."
---

# I Run a Creative Agency With One AI Agent. She Costs $3/Day and Managed Our Alibaba Sourcing.

Last Tuesday at 2 AM, my AI agent was comparing woven label quotes from five Alibaba suppliers while I slept. She'd already pulled the artwork specs from our design files, knew the difference between damask and satin weave (she learned that in January), and was building a comparison matrix with pricing, MOQs, and lead times.

By the time I woke up, she'd flagged the best option, noted that one supplier's pricing didn't include the centerfold we needed, and — this is the part that still gets me — cross-referenced the delivery timeline against our production calendar in Notion to confirm we had enough buffer before our next drop.

She costs about $90/month. That's $3/day. I run a creative agency and DTC brand called Superculture Studios. We do design, production, fulfillment — the full stack. And for the past several months, I've been running the entire operation with an AI agent as my chief of staff.

I want to be clear: I'm not an AI person who talks about operating a business. I'm an operator who figured out how to make AI do actual work. I have no CS degree. I didn't build the agent framework from scratch (it's called OpenClaw). Half the time I'm not sure if I'm doing this the "right" way. But the results are real, the costs are trivially low, and I think the gap between "people who use ChatGPT" and "people who run agents" is about to become the most important divide in small business.

Here are the five workflows that make this work. Not in theory. In my actual business, this week.

---

## 1. World-State Fuzzy Sync

Your business doesn't live in one place. Mine lives across Gmail, Notion, Slack, Google Calendar, iMessage, WhatsApp, Granola meeting transcripts, ShipBob, and about six other surfaces I'm forgetting.

The agent reads all of them. Not in a "connected your apps" Zapier way. In a "she actually understands what's happening across your entire business" way.

Here's what that looks like concretely: a vendor emails about order #4521. The agent already knows that order was discussed in last week's coaching call (she has the transcript). She knows the deadline is on the calendar for next Friday. She knows my teammate messaged about a packaging concern on Slack two days ago. She knows ShipBob is showing low inventory on that SKU.

When she surfaces that email, it comes with context. Not "You got an email from Vendor X." More like: "Vendor X confirmed shipment for order #4521. Based on the timeline from your Feb 14 call, this lands 3 days before the deadline. Note: [teammate] flagged a packaging question on Slack — you may want to confirm specs before it ships."

This isn't email summarization. Every AI tool does email summarization. This is contextual intelligence across your entire business state. The agent holds a fuzzy model of everything happening and connects dots you'd miss because no human can keep 15 surfaces in working memory simultaneously.

(The technical bit: she runs 22+ scheduled cron jobs, each scanning a different surface. New data gets reconciled against her memory files and the Notion workspace. It's not magic — it's a lot of small, well-timed reads that compound into situational awareness.)

## 2. Meeting Transcript → Contextualized Business Actions

Every AI meeting tool gives you a summary. Great. A summary of a meeting is worth approximately nothing if it's not connected to the work.

Here's what my agent does with a meeting transcript:

After a coaching call with my business coach, the agent pulls the Granola transcript (usually 50-70K characters of raw conversation). She doesn't just summarize it. She processes it against the entire business context — the current sprint board in Notion, outstanding tasks, recent decisions, active projects.

The output isn't "discussed marketing strategy." The output is structured Notion tasks:

- **Homework:** "Rewrite positioning deck with new ICP framing" → assigned, deadline set, linked to the transcript with the exact quote where the coach said it
- **Mindset shifts:** Captured as reference notes ("Stop saying 'we're a design agency' — you're a brand operator")
- **Outreach targets:** Three specific people mentioned in the call → tasks created with context on why and suggested messaging
- **Operational changes:** "Move fulfillment review to Monday AM" → calendar event suggested

Each task links back to the transcript with the relevant quote highlighted. So when I'm looking at "rewrite positioning deck" in two weeks and can't remember why, the context is right there.

This is 10x better than any meeting summary tool because the agent knows what I'm already working on. She doesn't create duplicate tasks. She doesn't suggest things I've already done. She understands the delta between what was discussed and what's already in motion.

(I'll be honest — the first few times, the task extraction was messy. The agent would create 30 tasks from a single call, half of them redundant. We tuned it. Now she caps at the highest-leverage items and groups related actions. It's still not perfect. But it's better than my notes.)

## 3. AI-Managed Supplier Sourcing (Accio/Alibaba)

This is the one that makes people's eyes go wide.

We needed custom woven labels for a new product line. If you've done DTC production, you know this dance: find suppliers on Alibaba (or Accio, Alibaba's newer sourcing platform), send specs, wait for quotes, compare, negotiate, sample, repeat.

I gave the project to my agent.

She pulled the artwork specs from our design files. She drafted outreach messages to 5+ suppliers with the correct terminology — woven (not printed), damask weave, centerfold, specific dimensions in centimeters (not inches — suppliers are in China). She sent the messages through browser automation on Accio.

Then she set up a monitoring cron that ran every 30 minutes, scanning for vendor replies. When replies came in, she'd navigate to Accio via browser automation, read the conversation threads, extract pricing and MOQs, and update the comparison matrix.

The comparison matrix had: unit price at different quantities, weave type, fold type, sample cost, shipping terms, estimated lead time, and supplier communication quality (she actually tracked response time and helpfulness).

Here's what blew my mind: the agent *learned garment industry terminology* through this process. By the end of the project, she knew that damask weave is smoother and more detailed than satin weave. She knew centerfold vs. end-fold vs. Manhattan fold. She knew that "woven" and "printed" labels have completely different price structures and minimum orders.

That vocabulary didn't disappear after the project. It's in her solution files now. The next time we source any garment component, she starts from that knowledge base, not from zero.

We also learned the hard way: when we picked our primary supplier, the agent politely declined the backup vendors. Seemed logical — we'd made our choice. Then the primary had a production delay, and we had to re-engage backups cold, losing a week. Now there's a rule in her operating procedures: always keep backup vendors warm until delivery is confirmed.

(The $3/day framing really hits here. A sourcing agent or freelancer for this project would cost $500-2,000. The agent did it for roughly $15 in compute over two weeks, running while I worked on other things.)

## 4. 4-Layer Model Resilience

This is the infrastructure layer that nobody talks about because it's boring until it saves you.

The agent doesn't run on one AI model. She has a fallback chain: Claude Opus for complex reasoning, Sonnet for standard tasks, Haiku for simple operations. Each of those 22+ cron jobs is routed to the appropriate model tier based on task complexity.

Why this matters: AI APIs go down. Anthropic has outages. Rate limits hit. If your workflow depends on one model, your business stops when that model stops.

My agent has a budget watchdog that monitors daily spend. When costs spike (maybe a complex task required more Opus calls than expected), the watchdog can throttle non-essential crons to what we call "austerity mode." Essential briefings and monitoring keep running. The weekly Notion cleanup can wait.

There's also a self-healing health monitor. If a model starts returning errors, the agent auto-escalates to the next tier. If Sonnet is down, tasks route to Opus (more expensive, but the work gets done). If Opus is down too, critical tasks fall to Haiku with simplified prompts.

The best example: our memory search was running on Gemini's free tier. Worked great until we hit rate limits during a heavy work session. Instead of just failing, the agent built a local embedding fallback that runs entirely on-device. Zero API cost, zero rate limit risk. Now the free tier is primary and local is fallback, but either way, the search works.

Your 8 AM business briefing arrives every morning. Even during an Anthropic outage. Even when you've burned through your daily budget on a complex project. The resilience layer handles it.

(I'm aware this sounds over-engineered for a small creative agency. It probably is. But I've also never missed a morning briefing or had a supplier monitoring gap because of an API issue. The over-engineering costs nothing extra in daily operation — it's just configuration.)

## 5. Nightshift + Compound Learning Loop

The agent works while I sleep. But not randomly.

Every night, she evaluates a catalog of pending tasks using a scoring function: urgency × proximity × staleness × domain rotation. Domain rotation matters — she deliberately cycles through different areas (codebase, communications, research, operations) so nothing gets neglected.

Budget rules: never exceed 60% of remaining daily tokens overnight. Per-task cap at 15% of the nightly budget. This prevents one runaway task from eating the whole night's capacity.

But the task execution isn't the interesting part. The compound learning loop is.

After every significant task completion — not just overnight, but anytime — the agent captures what worked and what didn't. She writes it to a solution file. She updates her own operating rules if the learning is persistent.

The agent that sourced woven labels in January knows garment terminology in March. The agent that learned to keep backup vendors warm applies that principle to every future sourcing project. The agent that figured out how to parse ShipBob's billing API writes a solution file that makes the next ShipBob task take 5 minutes instead of 2 hours.

There's a weekly synthesis cron that reviews all learnings from the week and consolidates patterns. It's not just memory — it's institutional knowledge that compounds.

Here's why this matters for a small operation: fractional talent turns over. Freelancers leave. Contractors forget context between engagements. Your agent doesn't. Every project makes the next one better. Six months in, the agent has more institutional knowledge about my business operations than any contractor I've ever hired.

---

## The Real Talk

I want to be honest about what this isn't.

This isn't plug-and-play. Setting up these workflows took weeks of iteration. The agent's operating rules file is hundreds of lines long, and a lot of those lines exist because something went wrong and we wrote a rule to prevent it from happening again. (Rule 12: "Never claim you don't have access to something without checking first." That rule exists because the agent once told me she couldn't access Notion when the API key was sitting right there in the config directory.)

This isn't free of mistakes. The agent has sent emails I had to apologize for. She's created tasks that made no sense. She once declined all our backup suppliers (see above). The error rate is low, but it's not zero, and for anything customer-facing, I review before it sends.

This isn't for everyone yet. You need to be comfortable in a terminal. You need to think in systems, not features. You need to be okay with something that's 80% automated and 20% duct tape.

But if you run a small operation — agency, DTC brand, consulting practice, studio — and you spend your days context-switching between 15 tools and losing track of what's happening in your own business, this is what the near future looks like. Not chatbots. Not copilots. An actual agent that knows your business, works while you sleep, and gets better every week.

$3/day. $90/month. Less than your Slack bill.

I'm building this in public. If you want to see the actual setup — the cron jobs, the operating rules, the overnight task scoring — DM me or follow along. No course. No playbook. Just an operator showing the work.

---

*Cordell Marama runs Superculture Studios, a creative agency and DTC brand. He's been running with an AI chief of staff since late 2025.*

---

*← Back to [README](../README) | Section: [Articles](./)*
