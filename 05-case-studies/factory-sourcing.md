---
title: "Case Study: Factory Sourcing — PO to Booked Factory"
---

# Case Study: Factory Sourcing — PO to Booked Factory

*End-to-end supply chain operation, autonomous. The one that makes sourcing people's jaws drop.*

---

## The Problem

Superculture needed custom woven labels for a new product line. If you've done DTC production, you know the dance: find suppliers on Alibaba, send specs with correct garment terminology, wait for quotes, compare across multiple dimensions, negotiate, and book.

This process typically takes a sourcing agent or freelance production coordinator 1-2 weeks of back-and-forth. The sourcing agent costs $500-2,000 for the project. And when they're done, the knowledge walks out the door.

Cordell gave the project to Alice.

## The Context

Superculture Studios is a creative agency and DTC brand. Production runs are small-to-medium (hundreds to low thousands of units), which means factory relationships matter — you can't throw volume at pricing problems. The spec had to be precise: woven (not printed), damask weave, centerfold, specific dimensions in centimeters. Getting any of those wrong means a sample round that doesn't match, adding 1-2 weeks to the timeline.

The sourcing platform was Accio — Alibaba's newer B2B sourcing tool, designed for smaller buyers. No public API. No Zapier integration. Just a web interface that expects a human clicking through conversation threads.

## The Approach

### Step 1: Spec Extraction
Alice pulled artwork specifications from Superculture's design files. This wasn't just reading dimensions — it required translating design intent into garment industry terminology:

- **Woven, not printed** — completely different manufacturing process, different price structure, different MOQs
- **Damask weave** — smoother and more detailed than satin weave, critical for the artwork's fine lines
- **Centerfold** — the label folds in half along the center (vs. end-fold or Manhattan fold), which affects how it's sewn into the garment
- **Dimensions in centimeters** — suppliers are in China; inches create confusion and misquotes

Alice had never sourced woven labels before this project. She learned the terminology by researching garment industry standards and cross-referencing supplier listings to understand the vocabulary.

### Step 2: Supplier Outreach via Browser Automation
Alice drafted outreach messages to 5+ suppliers on Accio using browser automation. This wasn't a simple "send message" API call — it required:

1. Navigating to Accio's supplier search
2. Filtering for woven label manufacturers
3. Opening each supplier's page
4. Composing messages with the correct specifications
5. Including quantity tiers for pricing (different volumes, different unit costs)
6. Sending through Accio's messaging interface

Each message included correct garment industry terminology, specific technical requirements, and explicit quantity breakpoints. The messages read like they came from a production coordinator who'd been sourcing labels for years.

### Step 3: Automated Monitoring — Every 30 Minutes
Alice set up a monitoring cron job that ran every 30 minutes, scanning for vendor replies. When replies came in:

1. Navigate to Accio via browser automation
2. Check each conversation thread for new messages
3. Extract pricing, MOQ information, lead times, and specifications
4. Parse the supplier's response against the original spec (did they quote for damask or satin? centerfold or end-fold?)
5. Update the comparison matrix
6. Flag any discrepancies or missing information

This ran autonomously for two weeks. While Cordell was in meetings, on calls, working on other projects — Alice was monitoring supplier responses in the background.

### Step 4: Comparison Matrix
Alice built a structured comparison across seven dimensions:

| Dimension | Why It Matters |
|-----------|---------------|
| Unit price at different quantity tiers | Volume discounts vary dramatically between suppliers |
| Weave type (damask vs satin) | Affects visual quality and detail reproduction |
| Fold type (centerfold vs end-fold vs Manhattan fold) | Must match garment construction method |
| Sample cost and timeline | Some suppliers charge $50 for samples, others $200 |
| Shipping terms (FOB, CIF, DDP) | Affects landed cost significantly |
| Estimated lead time | Must fit the production calendar |
| Supplier communication quality | Response time, helpfulness, English fluency — predicts the working relationship |

The matrix wasn't just data collection — it was analysis. Alice noted that one supplier's pricing looked competitive but didn't include the centerfold specification. Without catching that detail, the sample would arrive with the wrong fold type, adding 2+ weeks to the timeline.

### Step 5: Cross-Reference with Production Calendar
Before making a recommendation, Alice cross-referenced the delivery timeline against the production calendar in Notion. The question wasn't just "which supplier is cheapest?" — it was "which supplier can deliver with enough buffer before the next drop?"

This required reading the Notion production schedule, understanding the dependency chain (labels must arrive before garment assembly, garment assembly before fulfillment prep, fulfillment before the drop date), and calculating buffer days.

### Step 6: Recommendation with Reasoning
Alice flagged the best option with full reasoning — not just "Supplier C is cheapest" but a multi-factor analysis weighing price, quality signals, communication quality, timeline fit, and risk factors.

## The Failure That Became a Rule

When Alice selected the primary supplier, she proactively declined all backup vendors. Seemed logical — the decision was made, why waste their time?

Then the primary supplier had a production delay. Re-engaging backup vendors cold — after politely declining them — cost a week. The vendors who'd been told "thanks but no thanks" were slower to respond, less motivated to prioritize the order, and one had already allocated capacity elsewhere.

This became a permanent operating rule: **"Never close out optionality without explicit approval."** Keep backup vendors warm until delivery is confirmed. The rule doesn't just apply to sourcing — it applies to every future decision where Alice might prematurely close down alternatives.

This is how the compound learning engine works: a real failure in production becomes a permanent rule that prevents an entire class of errors forever.

## The Knowledge That Compounds

Through this project, Alice learned garment industry terminology that became part of her permanent knowledge base:

- Damask weave is smoother and more detailed than satin weave — better for fine artwork
- Centerfold vs end-fold vs Manhattan fold — different fold types for different label applications
- "Woven" and "printed" labels have completely different price structures and MOQs
- Chinese suppliers quote in centimeters, not inches — sending specs in inches causes confusion
- MOQs vary dramatically by weave type — satin has lower minimums, damask requires higher volume
- FOB vs CIF vs DDP shipping terms — each shifts cost and risk differently between buyer and seller
- Sample lead times are a proxy for production reliability — suppliers who can't turn samples quickly won't hit production deadlines

**This vocabulary didn't disappear after the project.** It's stored in solution files and the memory system. The next time Superculture sources any garment component — labels, hang tags, woven patches, care labels — Alice starts from that knowledge base, not from zero.

Freelancers turn over. Contractors forget context between engagements. Alice's garment sourcing knowledge is permanent and grows with every project.

## The Numbers

| Metric | Value |
|--------|-------|
| Suppliers contacted | 5+ |
| Monitoring frequency | Every 30 minutes for 2 weeks |
| Total monitoring checks | ~670 |
| Compute cost | ~$15 over 2 weeks |
| Time saved | 1-2 weeks of manual sourcing |
| Equivalent freelancer cost | $500-2,000 |
| Knowledge retained | Permanent (garment terminology, supplier evaluation criteria, negotiation patterns) |
| Rules created | 1 permanent rule ("never close optionality without approval") |

## Why This Matters

**Autonomous execution.** This wasn't Alice responding to prompts — it was Alice running an entire workflow autonomously over two weeks, with monitoring crons, browser automation, structured analysis, and proactive recommendations.

**Domain learning.** Alice didn't know garment industry terminology before this project. She learned it through execution, and the knowledge persisted. The system gets smarter through use.

**Cost asymmetry.** $15 in compute over two weeks vs. $500-2,000 for a freelance sourcing agent. And the freelancer's knowledge leaves when the project ends.

**The compound effect.** The next sourcing project — for any garment component, not just labels — starts from the knowledge base built here. Solution files, terminology, supplier evaluation frameworks, and the "keep backups warm" rule all carry forward. Every project makes the next one faster, cheaper, and better.

> A sourcing agent would've cost $500-2,000 for the same project. Alice did it for $15 in compute over two weeks, running while Cordell worked on other things. And she kept the knowledge for next time. That's not just cheaper — it's a fundamentally different model for how small operations handle supply chain.

---

*← Back to [README](../README) | Section: [Case Studies](./) | Related: [Compound Learning](../02-deep-technical/compound-learning)*
