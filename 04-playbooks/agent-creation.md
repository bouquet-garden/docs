---
title: "Agent Creation Playbook"
---

# Agent Creation Playbook

*How to add a new specialized agent to the Alice ecosystem — and when not to.*

---

## When to Create vs. When Not To

### Create a new agent when:
- Work requires **distinct specialization** that doesn't fit existing agents
- **Cost optimization** — move expensive recurring work off Opus to a cheaper model
- **Isolation needed** — risky or experimental operations that shouldn't pollute main workspace
- **Scale** — need multiple parallel sessions of the same work type

### Don't create a new agent when:
- An existing agent can handle the work adequately
- Task is one-time only → use a sub-agent (temporary spawn) instead
- The cost difference doesn't justify the maintenance overhead
- Specialization isn't meaningful (a "general helper" agent is just Alice again)

### The Real Test
Ask: "Will this agent run frequently enough to justify its configuration and maintenance?" If the answer is "maybe once a month," use a sub-agent spawn from Alice instead.

---

## Design Phase

### 1. Define the Single Responsibility

Each agent has ONE primary domain. State it in one sentence:

| Agent | Responsibility |
|-------|---------------|
| Builder | Web development and API architecture |
| Mojo | Mobile and React Native development |
| Frank | Operations monitoring and budget tracking |
| CS | Customer support triage via Commslayer |

If you can't state it in one sentence, the scope is too broad. Split into two agents or narrow the scope.

### 2. Choose the Model Tier

Match model to task complexity:

| Complexity | Model | Monthly Cost Estimate |
|-----------|-------|----------------------|
| Strategic reasoning, judgment | Opus | $$$ |
| Analysis, documentation | Sonnet | $$ |
| Mechanical operations, classification | Haiku | $ |
| Code generation, experimental | Kimi K2.5 | ¢ |

**Rule of thumb:** Use the cheapest model that produces acceptable output quality. Test with a canary task (see below) before committing.

### 3. Plan the Fallback Chain

Every agent needs a fallback chain for when the primary model fails:

```
Primary: kimi-k2.5-free
Fallback 1: claude-sonnet-4
Fallback 2: claude-opus-4-6 (optional, for critical agents)
```

Exceptions: Alice (Opus, no fallback — quality non-negotiable) and Frank (Haiku, simple enough that any model works).

### 4. Define Tool Access

Only provide tools the agent actually needs. Tool allowlists prevent accidents and reduce context size:

| Agent Type | Tools |
|-----------|-------|
| Developer | exec, read, write, edit, browser, web_search |
| Comms | exec, message, read, write |
| Monitor | exec, read, message |
| Research | exec, read, web_search, browser |

**Key exclusion:** Builder should NOT have `message` tool access. An overnight builder that can send Telegram messages will eventually send something it shouldn't.

### 5. Memory Search Decision

Enable or disable based on role:

- **Enable** for roles that benefit from historical context (research, analysis, strategy)
- **Disable** for roles that work from explicit specifications (coding, simple operations)

Cost impact: memory search adds embedding generation + vector queries per interaction.

---

## Implementation Phase

### 1. Create the Workspace

```bash
mkdir -p ~/.openclaw/workspace-{agentname}/
```

### 2. Write SOUL.md

The agent's identity, expertise, and behavioral constraints:

```markdown
# {Agent Name}

## Role
{One sentence responsibility}

## Expertise
{Specific domains and skills}

## Communication Style
{How this agent communicates — formal? concise? detailed?}

## Boundaries
{What this agent does NOT do}
```

**Why personality matters:** We tested agents with and without role definitions. Clear role definitions measurably improve output quality for specialized tasks. The personality constrains the model's approach to match the specialization.

### 3. Write BOOTSTRAP.md

First-run instructions that the agent follows on initialization:

```markdown
# Bootstrap

1. Read SOUL.md
2. Set up workspace structure: {directories, files}
3. Verify tool access: {test commands}
4. Confirm ready for work

After completing bootstrap, delete this file.
```

### 4. Configure in Gateway

Add the agent to `openclaw.json` (or via gateway config):

```json
{
  "name": "{agentname}",
  "model": {
    "primary": "kimi-k2.5-free",
    "fallbacks": ["claude-sonnet-4"]
  },
  "tools": ["exec", "read", "write", "edit"],
  "memorySearch": { "enabled": false },
  "workspace": "~/.openclaw/workspace-{agentname}/"
}
```

**⚠️ Critical:** When patching `agents.list`, you MUST include ALL agents — not just the ones you're changing. The patch replaces the entire array. Omitting agents deletes them. (Learned February 11: accidentally deleted 5 agents by patching only 2.)

### 5. Canary Task

Before real work, run a small throwaway task (30 min max):
1. Spawn the agent with a simple task in its domain
2. Review output quality
3. If quality is poor → escalate model before starting real work

Don't discover model limitations on milestone-critical work.

---

## Integration Phase

### 1. Update Routing Rules

Add to AGENTS.md: when should Alice delegate to this agent?

```markdown
**Route to {Agent} when:**
- {Condition 1}
- {Condition 2}

**Don't route to {Agent} when:**
- {Exception}
```

### 2. Set Up Monitoring (If Cron-Based)

If the agent handles scheduled work, add monitoring:
- Health check cron (verify agent can be spawned)
- Progress tracking (for long-running tasks)
- Stall detection (for overnight work)

### 3. Document

Update agents-guide.md with:
- Purpose and capabilities
- Model assignment and rationale
- Cost estimate
- Routing rules

### 4. First Week Monitoring

Watch the new agent closely for the first week:
- Is it producing quality output?
- Is the model appropriate for the task complexity?
- Are tool access restrictions correct?
- Is memory search needed/unnecessary?
- What's the actual cost?

---

## Maintenance

### The 3-Strike Rule

If an agent fails the same task 3 times:
1. STOP spawning
2. Write up what's failing (model? prompt? missing context?)
3. Escalate to Cordell
4. Do NOT keep burning tokens hoping attempt #4 works

### Retirement Criteria

Remove an agent when:
- It hasn't been spawned in 30+ days
- Its tasks can be absorbed by another agent
- The maintenance cost exceeds the value

### Config Patch Safety

When modifying agent configuration:
1. Back up current config
2. Include ALL agents in the patch (replacement, not merge)
3. Validate at least one agent works after the change
4. Check that no agents were accidentally deleted

---

*Current roster: 9 agents. Each was added to solve a specific problem and justified by clear cost optimization or specialization needs. The roster was consolidated from 8 to 4+3 during v2.0 "Mission Control" — fewer agents with clearer roles outperformed more agents with overlapping responsibilities.*

*See also: [Agent System](../tier1/agents.md) for the current roster, [Multi-Agent Orchestration](../tier2/multi-agent.md) for coordination patterns.*

---

*← Back to [README](../README.md) | Section: [Playbooks](./)*
