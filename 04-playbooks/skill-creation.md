---
title: "Skill Creation Playbook"
---

# Skill Creation Playbook

*Turn repeatable workflows into reusable capabilities.*

---

## When to Create a Skill

### Create a skill when:
- **Multi-step process** that agents repeat frequently (3+ times)
- **Complex integration** requiring specific setup or credentials
- **Error-prone operations** needing robust error handling
- **Cross-agent capability** — used by multiple agents
- **Domain expertise** requiring specialized prompts or logic

### Don't create a skill when:
- Simple one-liner commands → use `exec` directly
- One-off operations → handle inline
- Agent-specific logic → put in that agent's SOUL.md
- Existing skill covers the use case (check the 67-skill inventory first)

### The Real Test
"Have I done this exact thing 3+ times, and did I make mistakes at least once?" If yes, it should be a skill. The SKILL.md captures the correct approach so mistakes don't recur.

---

## Skill Design Principles

1. **Single Responsibility.** One skill, one clear purpose. `email-triage` triages email. It doesn't also manage Notion tasks.

2. **Consistent Interface.** Predictable inputs and outputs. Every skill should have clear: what goes in, what comes out, what errors are possible.

3. **Error Resilience.** Handle failures gracefully with clear messages. Don't let a failed API call crash the entire workflow.

4. **Documentation First.** Write SKILL.md before the implementation. If you can't explain the skill clearly, the design isn't ready.

5. **Cost Awareness.** Skills that run frequently (via crons) should be optimized for token efficiency.

---

## Step-by-Step Guide

### Phase 1: Planning

**1. Define purpose in one sentence:**
> "This skill triages incoming email by priority and prepares draft responses."

**2. Identify target agents:**
Which agents will use this skill? This determines tool requirements and model expectations.

**3. Design interface:**
```
Input:  {what the skill needs to run}
Output: {what the skill produces}
Errors: {what can go wrong and how to handle it}
```

**4. Plan dependencies:**
External services, credentials, tools, other skills.

**5. Estimate frequency:**
How often will this run? Daily cron = optimize for cost. Ad-hoc = optimize for quality.

### Phase 2: Implementation

**1. Create skill directory:**
```bash
mkdir -p ~/clawdbot/skills/{skill-name}/
```

**2. Write SKILL.md first:**

```markdown
# {Skill Name}

## Purpose
{One sentence}

## Usage
{How an agent invokes this skill}

## Examples
{Concrete usage examples with real-world scenarios}

## Inputs
{What the skill needs}

## Outputs
{What the skill produces}

## Error Handling
{Common failures and recovery steps}

## Prerequisites
{Setup requirements, credentials, dependencies}

## Cost
{Estimated token cost per invocation}
```

**3. Implement core functionality:**

For script-based skills:
```javascript
// skills/{name}/index.js
async function execute(params) {
  try {
    // Core logic
    return { success: true, result: ... };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

For prompt-based skills (structured instructions that agents follow):
```markdown
## Execution Steps
1. {Step 1 with specific commands}
2. {Step 2 with expected output}
3. {Step 3 with error recovery}
```

**4. Add error handling:**

Every external call should have:
- Timeout (don't hang forever)
- Retry logic (transient failures are common)
- Graceful failure message (what went wrong, what the agent should do next)
- Fallback path (alternative approach if primary fails)

### Phase 3: Integration

**1. Test with target agents:**
Spawn the target agent with a realistic task that uses the skill. Verify:
- Does it find the skill?
- Does it use it correctly?
- Does error handling work?
- Is the output useful?

**2. Add to skill registry:**
Update `skills-inventory.md` with the new skill's purpose, usage, and cost.

**3. Update agent configurations:**
If the skill requires specific tools, ensure target agents have access.

**4. Monitor first week:**
Watch for:
- Invocation frequency (is it being used?)
- Error rate (is it reliable?)
- Token cost (is it efficient?)
- Output quality (is it producing useful results?)

---

## Patterns from Our Skills

### The Email Triage Pattern

Multi-step workflow that combines reading, classification, and drafting:

```
1. Pull unread (himalaya envelope list)
2. For each: read content → classify priority → determine thread ownership
3. For urgent: draft response with context → save to drafts
4. Summarize: notification with counts and highlights
```

Key decisions:
- Classification is rule-based for common patterns, LLM-based for ambiguous cases
- Thread ownership uses keyword matching ("looking forward to your response" = ball with them)
- Drafts include Alice notes (red HTML blocks) that get stripped before sending

### The Reverse-Engineering Pattern

For integrations without public APIs:

```
1. Open target app in browser with dev tools → Network tab
2. Perform the action you want to automate
3. Copy the curl request
4. Extract: auth tokens, endpoint URLs, request format
5. Build script that replicates the request
6. Handle auth refresh (tokens expire)
```

Used for Commslayer (Chatwoot). Applicable to any web app.

### The Overnight Builder Pattern

Long-running autonomous task with monitoring:

```
1. Define task specification
2. Spawn agent with spec
3. Create progress file mechanism
4. Create paired monitoring cron
5. Monitor: stall detection (20 min no progress → kill)
6. On completion: immediate delivery + structured handoff
```

### The Cross-Reference Pattern

Connecting data across surfaces:

```
1. Read data from surface A (email)
2. Read data from surface B (Notion)
3. Extract entities from both
4. Match entities across surfaces
5. Surface connections in output
```

---

## Common Mistakes

### Mistake: Skill Does Too Much
A skill that "triages email, creates Notion tasks, updates Slack, and sends a summary" should be 3-4 separate skills composed by the agent. Single responsibility.

### Mistake: No Error Handling
External APIs fail. Network requests timeout. Credentials expire. If your skill doesn't handle these, it will crash and the agent won't know why.

### Mistake: Documentation After Implementation
Writing SKILL.md last means the implementation drives the design. Writing it first means the design drives the implementation. The second approach produces cleaner skills.

### Mistake: Not Checking Existing Skills
Before creating a new skill, check the 67-skill inventory. There might be a stock OpenClaw skill that does 80% of what you need. Extend, don't reinvent.

---

## Skill Lifecycle

```
Identify pattern (3+ repetitions)
    ↓
Design interface (SKILL.md first)
    ↓
Implement (minimal viable skill)
    ↓
Test (with target agent, realistic scenario)
    ↓
Deploy (add to registry, update configs)
    ↓
Monitor (first week: errors, usage, cost)
    ↓
Iterate (fix issues, add error handling)
    ↓
Mature (stable, documented, cross-agent)
```

---

*67 skills currently installed. Mix of stock OpenClaw skills from ClawdHub and custom skills built for Alice's specific workflows. Each custom skill represents a workflow that was repeated enough times to justify formalization.*

*See also: [Skills & Integrations](../tier1/skills-integrations) for the full inventory, [Agent Creation](agent-creation) for creating agents that use skills.*

---

*← Back to [README](../README) | Section: [Playbooks](./)*
