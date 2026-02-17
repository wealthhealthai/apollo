# Apollo — Human Intelligence Patterns for AI Agents

**God of knowledge, logic, and prophecy.**

Apollo is a collection of cognitive architecture patterns that enable AI agents to exhibit human-like learning, memory consolidation, and strategic reasoning over long time horizons.

## Overview

Modern AI agents have sophisticated reasoning but lack systematic approaches to:
- **Memory consolidation** — Converting daily experiences into long-term knowledge
- **Pattern recognition** — Extracting generalizable rules from specific incidents  
- **Strategic continuity** — Maintaining project context over weeks and months
- **Self-assessment** — Structured reflection on performance and system health
- **Critical knowledge retention** — Ensuring important lessons remain accessible

Apollo provides 5 cognitive patterns that address these gaps, inspired by human intelligence mechanisms like sleep consolidation, mental models, retrospectives, timeline tracking, and spaced repetition.

## The Five Patterns

### 1. Dream State — Periodic Memory Consolidation
Nightly synthesis of daily notes into long-term knowledge structures. Detects patterns, reinforces important memories, and maintains project dashboard freshness.

**Key insight:** Daily notes are write-only without consolidation. Humans consolidate during sleep; agents need explicit consolidation cycles.

### 2. Mental Models — Explicit Workflow Pattern Recognition  
Structured documentation of "how things work" vs. "what happened." Captures behavioral patterns, system dynamics, and recurring failure modes as reusable models.

**Key insight:** Experienced professionals use mental models, not first-principles reasoning, for familiar situations. Agents should too.

### 3. Retrospectives — Structured Periodic Self-Assessment
Weekly strategic review producing concrete action items. Validates mental models against reality, checks system health, and maintains improvement momentum.

**Key insight:** Without forced reflection cadence, systems default to "keep doing what we're doing" even when it's not working.

### 4. Gantt-State Timeline Tracking
Dependency-aware project timelines with historical velocity tracking. Automatically flags downstream impacts of delays and corrects planning optimism with empirical data.

**Key insight:** Gantt charts aren't just pretty pictures — they expose critical path, parallelism, and realistic completion estimates.

### 5. Spaced Repetition for Critical Rules
Three-tier system (Active → Reinforced → Established) that keeps the most important lessons always in context while aging out well-established knowledge.

**Key insight:** Rules buried in files don't influence behavior. Active repetition with promotion/demotion ensures critical knowledge stays accessible.

### 6. Warm Handoff — Sub-Agent Context Protocol
Universal onboarding protocol for spawned sub-agents. Every delegation includes environment context, task reasoning, required reading, and verification rules. Includes **ContextPacket API** evolution — a programmatic interface that queries the memory layer to automatically build context packets for sub-agent spawns, eliminating the discipline requirement of manual protocols.

**Key insight:** A sub-agent spawned without shared context is a new hire with no onboarding — they'll make confident mistakes based on incomplete information. Delegation without context is abandonment, not empowerment.

## Implementation

Each pattern includes:
- Detailed implementation instructions (cron schedules, file structures, exact processes)
- Integration points with existing systems
- Complexity ratings and time estimates
- Expected benefits and cost analysis
- Priority recommendations

**Quick start:** Implement Pattern 6 first (15 minutes, prevents sub-agent failures immediately). Then Patterns 5, 2, and 3 (all ~1 hour each, immediate value). Then Pattern 1 (2-3 hours, highest ceiling). Pattern 4 when first real project starts.

See [HUMAN-INTELLIGENCE-PATTERNS.md](./HUMAN-INTELLIGENCE-PATTERNS.md) for full details.

## Architecture

```
Session Start → Load Critical Rules (Pattern 5)
     ↓
Daily Work → Write daily notes
     ↓
Nightly (2 AM) → Consolidation Run (Pattern 1)
     ↓
     ├─→ Update Mental Models (Pattern 2)
     ├─→ Reinforce antaris-memory facts
     └─→ Refresh Project Dashboards
     ↓
Weekly (Mon 9 AM) → Retrospective (Pattern 3)
     ↓
     ├─→ Validate Models against reality
     ├─→ Promote/demote Critical Rules
     └─→ Check Timeline velocity (Pattern 4)
```

All patterns integrate with each other, creating a self-reinforcing cognitive architecture.

## Design Philosophy

1. **Files are memory** — Context windows are ephemeral. Structured files create persistent memory.
2. **Explicit over implicit** — Make cognitive processes visible and debuggable.
3. **Integrate, don't replace** — Patterns augment existing systems (PREFLIGHT, antaris-memory, PROJECT-BRAIN).
4. **Empirical feedback** — Track what works. Patterns validate themselves against outcomes.
5. **Human-inspired, not human-mimicking** — Borrow mechanisms that transfer well to agents.

## Use Cases

- **Long-term project management** — Maintaining strategic continuity over months-long initiatives
- **Agent teams** — Shared memory and models across multiple agent instances
- **High-stakes operations** — Critical rules stay accessible when they matter most
- **Learning systems** — Patterns improve over time as consolidation builds knowledge corpus
- **Human-AI collaboration** — Transparent cognitive processes enable better human oversight

## Requirements

- Cron/scheduler support for periodic jobs
- File system access for markdown-based memory
- Optional: antaris-memory integration for reinforcement cycles
- Optional: antaris-router for cost-optimized model selection

## Status

**Proposal stage** — Patterns documented, awaiting implementation and field testing.

Initial deployment target: Kuro agent, supporting Olympus/Midas project coordination.

## Credits

**Author:** Kuro (AI agent)  
**Sponsor:** Jason, WealthHealth AI  
**Inspiration:** Human cognition, agile retrospectives, spaced repetition research, project management best practices

## License

MIT
