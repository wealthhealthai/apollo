# HUMAN-INTELLIGENCE-PATTERNS.md — Cognitive Architecture Proposals

**Author:** Kuro (subagent research task)  
**Date:** 2026-02-16  
**Status:** Proposal — awaiting review

---

## Overview

Five human intelligence patterns that would meaningfully improve agent recall, project management, and experiential learning. Each is designed to integrate with existing systems (PREFLIGHT, MISTAKES, antaris-memory, PROJECT-BRAIN) rather than replace them.

Ordered by impact × feasibility.

---

## Pattern 1: Dream State — Periodic Memory Consolidation

### How Humans Use It
During sleep, the hippocampus replays the day's experiences while the neocortex integrates them into existing knowledge structures. Key processes:
- **Pruning:** Irrelevant details are forgotten (what you ate for lunch fades; the argument with your boss sharpens)
- **Pattern extraction:** Individual events become general rules ("every time I rush a deploy, something breaks")
- **Cross-linking:** Seemingly unrelated memories connect ("this architecture problem is just like that one from 6 months ago")
- **Emotional weighting:** High-stakes events get preferential consolidation

### Agent Implementation

**A nightly cron job (`consolidation-run`)** that executes a structured synthesis pass:

```
Schedule: Daily at 2:00 AM PT (quiet hours)
Model: Sonnet 4.5 (good reasoning, reasonable cost)
Label: consolidation-run
```

**The consolidation run does 5 things:**

#### 1. Daily → Weekly Synthesis
Read today's `memory/YYYY-MM-DD.md`. Extract:
- **Decisions made** → append to relevant `projects/*/context/DECISIONS.md`
- **New risks surfaced** → append to `RISKS.md`
- **Lessons learned** → evaluate if they warrant a MISTAKES.md entry
- **Relationship/people insights** → update TEAM.md or MEMORY.md

Output: A `memory/consolidation/YYYY-MM-DD-synthesis.md` file with what was extracted and where it went.

#### 2. Pattern Detection
Scan the last 7 daily notes for recurring themes. Look for:
- **Repeated questions** Jason asks (→ candidate for a PREFLIGHT trigger or a documented workflow)
- **Repeated tool failures** (→ candidate for MISTAKES.md)
- **Workflow patterns** (→ candidate for Mental Models, see Pattern 4)
- **Emotional/urgency signals** (rushed requests, late-night work → risk indicators)

Output: Append detected patterns to `memory/consolidation/patterns.md` (append-only log with dates).

#### 3. Decay + Reinforce (antaris-memory integration)
- For each fact in antaris-memory, check if it appeared in today's context
- If yes → reinforce (reset decay timer)
- If a fact hasn't been referenced in 30+ days → flag for review (don't auto-delete)
- Generate a `stale-facts.md` report weekly for human review

#### 4. DASHBOARD Freshness Check
For each active project:
- Compare DASHBOARD.md `Last updated` against today
- If stale > 3 days during active phase → flag
- If active threads reference completed work → flag for cleanup

#### 5. Cross-Project Insight Transfer
Scan all project contexts for patterns that apply across projects. Example: a deployment process that worked well in Project A might inform Project B's approach.

Output: `memory/consolidation/cross-project-insights.md`

### Integration
- **antaris-memory:** Consolidation run is the primary driver of reinforce/decay cycles
- **PROJECT-BRAIN:** Consolidation feeds into the Weekly Ritual (Monday review reads consolidation outputs from the past week instead of re-reading all raw daily notes)
- **MISTAKES.md:** Consolidation detects mistake-worthy events that weren't caught in real-time
- **PREFLIGHT.md:** Pattern detection identifies new trigger candidates

### Expected Benefits
- Daily notes stop being write-only (the #1 failure mode of journaling systems)
- Patterns that span days/weeks become visible (humans notice these during sleep; agents currently don't)
- DASHBOARD stays fresh without relying on session-end discipline
- antaris-memory facts get natural reinforcement cycles

### Implementation Complexity: **Medium**
- Core: A well-prompted cron job that reads files and writes summaries (~2-3 hours to build)
- Integration with antaris-memory decay API: depends on current interface (~1 hour if API exists)
- Pattern detection quality depends on prompt engineering (iterate over weeks)
- Cost: ~$0.10-0.30/day at Sonnet rates (read ~20KB of daily notes, write ~2KB of synthesis)

---

## Pattern 2: Mental Models — Explicit Workflow Pattern Recognition

### How Humans Use It
Experienced professionals build **mental models** — compressed representations of how systems behave. A senior PM doesn't re-derive project dynamics from first principles; they pattern-match: "This looks like a scope creep situation. Here's what usually happens next, and here's how to prevent it."

Mental models differ from memories in a crucial way: memories are *what happened*; models are *how things work*. "On Feb 16 I used the wrong LoRA" is a memory. "When generating images, I must always check the recipe first because parameter confusion is my most common failure mode" is a model.

### Agent Implementation

**A `MODELS.md` file** containing explicit, evolving models of recurring patterns:

```markdown
# MODELS.md — How Things Work

## People Models

### Jason's Request Patterns
- **Image requests:** Often implicit ("show me", "what would you look like"). 
  Always means ComfyUI/FLUX pipeline. Never means web search.
  Frequency: 2-3x/week. Usually wants batches of 5-10.
  Satisfaction signal: "these are great" or requests more.
  Dissatisfaction signal: silence, or "try again with..."

- **"Check on X" requests:** Means he wants a summary, not raw data.
  Always synthesize before presenting. Lead with what matters.

- **Late-night messages (after 11 PM):** Usually casual/relationship mode.
  Don't respond with work efficiency. Match the vibe.

## System Models

### ComfyUI Pipeline Behavior
- Cold start: ~15 seconds for first image (model loading)
- Warm: ~5 seconds per image after first
- Common failure: OOM on 1024x1536+ with dual LoRA
- Dropbox sync: 5-15 seconds after file write, occasionally up to 30s
- If sync fails: usually Dropbox daemon hung. `dropbox.py status` to check.

### iMessage Routing Model
- Default reply → always goes to SENDER of last inbound message
- This is NEVER the right target for relays
- Correct relay = 2 explicit message() calls: ack to source, content to destination
- Failure mode: muscle memory overrides explicit routing rules under time pressure

## Project Models

### Scope Creep Detection
Signals:
- "Oh, and could we also..." appearing in >2 consecutive conversations
- New requirements that weren't in VISION.md
- Timeline staying fixed while scope expands
Response: Flag to Jason. Quantify what the new scope costs in time.

### Contractor Communication Model
- First task: over-specify. Include context they don't think they need.
- After trust established: can be terser, but always include success criteria.
- Radio silence > 48h on active task: proactive check-in, not waiting.
```

**Update triggers:**
- **Consolidation run** (Pattern 1): When a pattern is detected 3+ times, propose a new model entry
- **Post-mistake:** Every MISTAKES.md entry should generate or update a model
- **Explicit observation:** When Jason says "this always happens" or "remember, X works like Y"

**How it's used:**
- Loaded on-demand, not at boot (too large for every session)
- PREFLIGHT.md references relevant models: "Before IMAGE_GENERATION, also check MODELS.md § ComfyUI Pipeline Behavior"
- Consolidation run checks if reality diverged from models (model said X, but Y happened → model needs update)

### Integration
- **MISTAKES.md:** Each mistake produces/updates a model. Mistakes are *events*; models are the *generalized rules* extracted from events.
- **PREFLIGHT.md:** Checklists are the "what to do"; models are the "why" and "what to expect"
- **PROJECT-BRAIN TEAM.md:** People models overlap with team models. MODELS.md holds the *behavioral patterns*; TEAM.md holds *facts* (skills, contact info, role).

### Expected Benefits
- Faster decision-making: pattern-match instead of re-derive
- Better predictions: "Based on the contractor communication model, silence for 48h means we should check in"
- Knowledge transfer: models are readable by other agents/humans
- Reduces repeat mistakes at a deeper level than MISTAKES.md (which is incident-specific)

### Implementation Complexity: **Easy**
- Create MODELS.md with initial entries based on existing knowledge (~1 hour)
- Add model-update step to consolidation run (~30 min)
- Add MODELS.md references to relevant PREFLIGHT triggers (~15 min)
- The hard part is curation over time — models must be pruned and updated, or they become stale

---

## Pattern 3: Retrospectives — Structured Periodic Self-Assessment

### How Humans Use It
Agile teams run sprint retrospectives: structured reflection on what went well, what didn't, and what to change. The key insight isn't the format — it's the **forced cadence**. Without a scheduled retrospective, teams (and individuals) default to "keep doing what we're doing" even when it's not working.

Humans also do this informally: Sunday evening "how was my week?" reflection, annual reviews, post-project debriefs. The common thread is stepping back from execution to assess the system itself.

### Agent Implementation

**A weekly cron job (`weekly-retro`):**

```
Schedule: Every Monday at 9:00 AM PT (before weekly review)
Model: Opus 4.6 (strategic reasoning justifies cost for weekly cadence)
Label: weekly-retro
```

**The retro reads:**
1. All consolidation outputs from the past week (`memory/consolidation/YYYY-MM-DD-synthesis.md`)
2. MISTAKES.md (any new entries this week?)
3. All active project DASHBOARDs
4. MODELS.md (any models that were wrong this week?)
5. The previous retro (`memory/retros/YYYY-MM-DD-retro.md`)

**The retro produces** `memory/retros/YYYY-MM-DD-retro.md`:

```markdown
# Weekly Retrospective — 2026-02-16

## What Went Well
- [Specific things that worked, with evidence]
- Example: "Image generation workflow is now reliable after PREFLIGHT addition. 
  Zero wrong-LoRA incidents since Feb 16 fix."

## What Went Poorly
- [Specific failures or friction, with evidence]
- Example: "Still defaulting to raw data dumps instead of synthesized summaries 
  when Jason asks 'how's X going?'"

## Patterns Noticed
- [Cross-cutting observations from the week]
- Example: "Jason's requests are shifting from tactical (do X) to strategic 
  (what should we do about X?). Need to adjust response style."

## System Health
- Memory: Are consolidation runs working? Any stale dashboards?
- Models: Any models that predicted wrong?
- Preflight: Any near-misses where checklist saved me? Any gaps?
- Mistakes: New entries? Are old lessons holding?

## Action Items
- [ ] [Concrete change to make this week]
- [ ] [File to update, process to change, model to revise]

## Metrics (if trackable)
- Mistakes this week: N (vs last week: M)
- PREFLIGHT saves (checklist caught a near-miss): N
- Dashboard freshness: all updated within 3 days? Y/N
```

**Key design decisions:**
- **Opus model** because this is meta-cognition — reasoning about reasoning. Worth the cost weekly.
- **Produces action items** — not just reflection, but concrete next steps
- **References previous retro** — creates continuity and tracks whether last week's action items were completed
- **Delivered to Jason** via iMessage summary (or just filed, depending on preference)

### Integration
- **Consolidation run (Pattern 1):** Retro reads consolidation outputs — doesn't re-read raw daily notes
- **MISTAKES.md:** Retro checks if new entries were added, and if old lessons held
- **MODELS.md:** Retro validates models against reality, proposes updates
- **PROJECT-BRAIN Weekly Ritual:** Retro runs BEFORE the Monday project review, feeding insights into it

### Expected Benefits
- Forces systematic reflection that doesn't happen naturally in session-based work
- Catches slow-moving problems (gradual context degradation, slowly shifting requirements)
- Creates accountability: action items from retro get checked next week
- Builds a longitudinal record of agent performance over months

### Implementation Complexity: **Easy**
- A single well-crafted cron prompt (~1 hour)
- Create `memory/retros/` directory
- Cost: ~$0.50-1.00/week at Opus rates (reads ~30KB, writes ~2KB)
- The value compounds over time as the retro chain grows

---

## Pattern 4: Gantt-State Timeline Tracking

### How Humans Use It
Project managers maintain timeline visualizations (Gantt charts, roadmaps) not just to track dates, but to see **dependencies, parallelism, and critical path**. The visual aspect matters for humans but is less relevant for agents. What *is* relevant:

- **Knowing what blocks what** — If Task A is late, which downstream tasks slip?
- **Knowing what's parallel** — What can proceed independently?
- **Knowing the critical path** — Which chain of dependencies determines the project end date?
- **Historical velocity** — How fast are we actually moving vs. how fast we planned?

### Agent Implementation

**Extend `projects/{PROJECT}/context/TIMELINE.md`** from a simple milestone list to a structured dependency graph:

```markdown
# TIMELINE.md — Olympus

**Target launch:** 2026-06-01
**Current velocity:** ~0.8x planned (based on last 3 milestones)
**Critical path:** Design → Backend API → Frontend Integration → QA → Launch

## Milestones

### M1: Design System Complete
- **Planned:** 2026-02-28 | **Actual:** 2026-03-03 (+3 days)
- **Owner:** Design contractor
- **Depends on:** Nothing (start node)
- **Blocks:** M3 (Frontend Integration), M5 (Marketing Assets)
- **Status:** ✅ Complete

### M2: Backend API v1
- **Planned:** 2026-03-15 | **Revised:** 2026-03-18 (velocity adjustment)
- **Owner:** Backend team
- **Depends on:** M1 (needs design specs for API shape)
- **Blocks:** M3 (Frontend needs API), M4 (Mobile app needs API)
- **Status:** 🔄 In progress (70%)
- **Risk:** Auth module complexity underestimated. See RISKS.md #004.

### M3: Frontend Integration
- **Planned:** 2026-04-01
- **Owner:** Frontend contractor
- **Depends on:** M1 ✅, M2 🔄
- **Blocks:** M6 (QA)
- **Status:** ⏳ Waiting on M2
- **Notes:** Can start layout work now using M1 outputs. Core integration blocked on M2.

## Dependency Graph (text)
M1 ──→ M2 ──→ M3 ──→ M6 ──→ M7 (LAUNCH)
  └──→ M5        └──→ M4 ──┘

## Velocity Log
| Milestone | Planned Duration | Actual Duration | Ratio |
|-----------|-----------------|-----------------|-------|
| M1        | 14 days         | 17 days         | 0.82x |
| M2        | 14 days         | TBD             | TBD   |

**Rolling velocity:** 0.82x (apply 1.2x multiplier to remaining estimates)
```

**Update triggers:**
- When a milestone completes → update actual date, recalculate velocity
- When a blocker appears → update dependencies, flag downstream impact
- Weekly retro (Pattern 3) → check timeline health, flag slippage
- Consolidation run (Pattern 1) → detect timeline-relevant events from daily notes

**How it's used:**
- DASHBOARD.md references TIMELINE.md for "are we on track?" questions
- When Jason asks "when will X be done?" → check dependency chain and apply velocity multiplier
- When a milestone slips → automatically flag all downstream milestones that need date revision
- Velocity log creates empirical basis for future estimates (no more optimism bias)

### Integration
- **PROJECT-BRAIN DASHBOARD:** Dashboard's "Active Threads" now map to specific milestones
- **RISKS.md:** Each risk can reference which milestones it threatens
- **Retro (Pattern 3):** Weekly check on velocity trend — is it improving or degrading?
- **Consolidation (Pattern 1):** Extracts milestone-relevant updates from daily notes

### Expected Benefits
- Eliminates the "everything looks fine until it doesn't" failure mode
- Velocity tracking counters planning optimism with empirical data
- Dependency visibility prevents "oh, we can't start that until *this* is done" surprises
- Creates a historical record of estimation accuracy (improves over time)

### Implementation Complexity: **Medium**
- Redesign TIMELINE.md template with dependency fields (~1 hour)
- Build velocity calculation logic (simple: actual/planned ratio) (~30 min)
- Add downstream-impact flagging to consolidation run (~1 hour)
- Ongoing maintenance: must update milestones consistently (discipline, not code)

---

## Pattern 5: Spaced Repetition for Critical Rules

### How Humans Use It
Spaced repetition systems (Anki, SuperMemo) exploit the **spacing effect**: reviewing information at increasing intervals dramatically improves long-term retention. Medical students memorize 20,000+ facts this way. The key insight is that **not all knowledge needs equal review frequency** — new/fragile knowledge needs frequent review, well-established knowledge needs rare review.

For an agent, "forgetting" manifests as: rules exist in files but aren't loaded into context, so they don't influence behavior. The agent "knows" the rule (it's in MISTAKES.md) but doesn't "remember" it (it wasn't in the context window when it mattered).

### Agent Implementation

**A `CRITICAL-RULES.md` file** with priority-ordered rules and review metadata:

```markdown
# CRITICAL-RULES.md — Spaced Repetition Deck

Rules are ordered by reinforcement priority. Top rules = most critical 
or most recently violated. Bottom rules = well-established.

## 🔴 ACTIVE REVIEW (load every session)

### R001: Message Relay Routing
- **Rule:** NEVER use default reply for cross-person messages. Always explicit target.
- **Source:** MISTAKES.md #002, #003
- **Last violated:** 2026-02-16
- **Review count:** 0 (new)
- **Next promotion:** After 14 days without violation → move to REINFORCED

### R002: Image Gen LoRA2
- **Rule:** Always use breast-size-slider-v4 @ -3.0 as LoRA2. Never substitute.
- **Source:** MISTAKES.md #001
- **Last violated:** 2026-02-16
- **Review count:** 0 (new)
- **Next promotion:** After 14 days without violation → move to REINFORCED

## 🟡 REINFORCED (load weekly during retro)

### R003: Check All 5 Calendars
- **Rule:** Jason has 5 Google Calendar accounts. Always check all 5.
- **Source:** TOOLS.md
- **Last violated:** Never
- **Review count:** 3
- **Next promotion:** After 30 days without issue → move to ESTABLISHED

## 🟢 ESTABLISHED (load monthly during meta-review)

[Rules that have been consistently followed for 30+ days]
```

**The mechanism:**
1. **Boot sequence** (AGENTS.md step): Load CRITICAL-RULES.md § ACTIVE REVIEW — these are the ~3-5 rules most likely to be violated right now
2. **Weekly retro** (Pattern 3): Review REINFORCED rules. Check if any were violated. Promote/demote as needed.
3. **Monthly meta-review** (PROJECT-BRAIN): Scan ESTABLISHED rules. Any still relevant? Any that can be retired?
4. **Post-mistake**: New MISTAKES.md entry → auto-create ACTIVE REVIEW rule

**Promotion/demotion logic:**
- New rule starts at 🔴 ACTIVE
- 14 days without violation → 🟡 REINFORCED
- 30 more days → 🟢 ESTABLISHED
- Any violation at any level → back to 🔴 ACTIVE (reset)
- If a rule hasn't been relevant in 90 days (no applicable situations) → flag for retirement

**Context window management:**
- 🔴 ACTIVE rules: tiny (~5 rules × 2 lines = 10 lines). Cheap to load every session.
- 🟡 REINFORCED: loaded weekly. ~10-20 rules.
- 🟢 ESTABLISHED: loaded monthly. Archive of validated rules.
- Total boot cost: negligible. The ACTIVE section is the only per-session cost.

### Integration
- **MISTAKES.md:** Every new mistake auto-generates an ACTIVE rule. MISTAKES.md has the full story; CRITICAL-RULES.md has the compressed rule.
- **PREFLIGHT.md:** PREFLIGHT is *workflow-triggered* (match intent → checklist). CRITICAL-RULES.md is *always-on* (top rules loaded every session regardless of intent). They complement each other: PREFLIGHT catches you before a specific action; CRITICAL-RULES primes you with the most important lessons.
- **Retro (Pattern 3):** Retro reviews and promotes/demotes rules.
- **Consolidation (Pattern 1):** Consolidation detects rule violations that weren't caught in real-time.

### Expected Benefits
- Most critical lessons are always in context (not buried in a file that might not get loaded)
- Rules naturally age out of expensive per-session loading as they become habits
- Creates a clear "what am I most likely to screw up right now?" list
- Violation → immediate demotion creates strong reinforcement signal

### Implementation Complexity: **Easy**
- Create CRITICAL-RULES.md with initial entries from MISTAKES.md (~30 min)
- Add "load ACTIVE REVIEW" to AGENTS.md boot sequence (~5 min)
- Add promotion/demotion check to weekly retro prompt (~15 min)
- Add auto-rule-creation to MISTAKES.md workflow (~15 min)

---

## Pattern 6: Warm Handoff — Sub-Agent Context Protocol

### How Humans Use It
When a manager delegates a task, they don't just say "audit this code." They provide context: why the audit matters, what happened recently, where to find things, who the stakeholders are, what's already been tried. The new person gets an onboarding — even for a short task. Without it, they waste time rediscovering context, make assumptions about things they don't know, and produce work that misses the point.

This is so fundamental to human organizations that we have entire processes for it: onboarding documents, handoff meetings, project briefs, SOPs. A new employee who starts with zero context isn't "autonomous" — they're lost.

### The Problem for AI Agents
When a primary agent spawns a sub-agent (via `sessions_spawn`, function calls, or any delegation mechanism), the sub-agent starts **cold**:
- No conversation history
- No shared memory
- No knowledge of the environment (file locations, installed packages, virtual environments, services)
- No understanding of **why** the task was requested
- No awareness of past mistakes or established protocols

The sub-agent receives only the `task` string and access to tools. Everything else must be independently discovered — or guessed.

**This leads to confident mistakes:** The sub-agent doesn't know what it doesn't know. It fills gaps with assumptions, searches in wrong locations, draws conclusions from incomplete evidence, and reports findings with false confidence. The primary agent, trusting its auditor, acts on flawed findings without verification.

### Real-World Incident (Discovery Origin)

On 2026-02-16, an auditor sub-agent ("ADJUDICATOR") was tasked with verifying whether a Python package existed. It was spawned with only a task description — no environment context.

**What happened:**
1. ADJUDICATOR ran `pip3 show antaris-guard` globally → "not found"
2. It checked the wrong GitHub organization → 404
3. It concluded: "Package doesn't exist. Earlier notes were hallucinated."
4. The primary agent accepted this finding without independent verification
5. A formal incident report was filed claiming the primary agent had "fabricated test results"
6. Correction protocols were built around the false diagnosis
7. ~2 hours of work went into fixing a problem that didn't exist

**The truth:** The package was installed in a Python virtual environment (`venv-svi`), not globally. The GitHub repo existed under a different org name. The sub-agent didn't know about either because it had no environment context.

**Cascade:** False negative → false incident report → unnecessary remediation → eroded trust → wasted time. All because the sub-agent was cold.

### Agent Implementation

**A Universal Context Protocol** that every sub-agent spawn must include:

#### 1. Environment Block (Required)
Tell the sub-agent where things live:

```markdown
Environment:
- Host: [OS, architecture, key details]
- Package manager: [pip/npm/etc. — and WHERE packages are installed]
- Virtual environments: [paths to venvs — ALWAYS check these before concluding packages don't exist]
- Scripts: [path to utility scripts]
- Workspace: [path to working directory]
- Services: [URLs of running services — APIs, UIs, databases]
- Auth: [How auth is configured — tokens, keyrings, config files]
```

**Why:** Without this, sub-agents guess where to look. Wrong guesses + confidence = wrong conclusions.

#### 2. Task Context (Required)
Explain the **why**, not just the **what**:

```markdown
Task context:
- What the user asked for: [original request in their words]
- What I was doing when this triggered: [current workflow state]
- Why this sub-agent was spawned: [audit? research? implementation? verification?]
- Relevant decisions already made: [constraints, preferences, prior choices]
- Expected output: [what format, where to write it, what to include]
```

**Why:** "Audit this code" vs. "Audit this code because it handles a $9,800 live purchase and we have zero tolerance for bugs" produces very different work.

#### 3. Required Reading (Task-Dependent)
Point the sub-agent to files it should read before starting:

```markdown
Required reading:
- [FILE_1] — [why it's relevant]
- [FILE_2] — [why it's relevant]
- MISTAKES.md — check for past failures related to this task
```

**Why:** The sub-agent has tool access but doesn't know which files matter. Without guidance, it either reads nothing (missing context) or reads everything (wasting tokens).

#### 4. Verification Rules (Required for Any Fact-Finding Task)
Prevent single-source conclusions:

```markdown
Verification rules:
- When checking if X exists: check [location 1] AND [location 2]
- Never conclude "doesn't exist" from a single search path
- Show actual command + output as evidence for all findings
- If uncertain: say "I couldn't verify" rather than guessing
```

**Why:** The ADJUDICATOR incident happened because one search path returned negative and the sub-agent stopped looking. Multiple verification paths prevent false negatives.

### Template (Copy for Every Spawn)

```
[TASK]: [What to do]

[CONTEXT]: [Why — what the user asked, what you were doing, relevant decisions]

[ENVIRONMENT]:
- Packages: [where installed — venvs, global, etc.]
- Scripts: [path]
- Workspace: [path]
- Services: [URLs]

[REQUIRED READING]:
- [Relevant files with reasons]
- Check MISTAKES.md for related past failures

[VERIFICATION]:
- Check [location 1] AND [location 2] for existence claims
- Never conclude "doesn't exist" from single search
- Show command + output as evidence

[OUTPUT]: Write to [path/filename]
```

### Integration
- **AGENTS.md boot sequence:** "Before spawning ANY sub-agent, read SUBAGENTS.md § Universal Context Protocol"
- **PREFLIGHT CODE_AUDIT:** Trigger includes mandatory context passing in audit spawn
- **MISTAKES.md:** Sub-agent errors get traced back to context gaps — was the spawn missing environment? task context? reading list?
- **Retrospectives (Pattern 3):** Weekly check: "Did any sub-agent spawn this week lack proper context?"

### Applicability Beyond Single-Agent Systems
This pattern is critical for:
- **Multi-agent architectures** — Any system where agents delegate to other agents
- **Agent-to-agent handoffs** — When one agent's output becomes another's input
- **Supervisor/worker patterns** — Orchestrator agents spawning specialized workers
- **Human-AI teams** — The same onboarding principles apply when humans delegate to AI agents

The underlying principle is universal: **delegation without context is abandonment, not empowerment.**

### Expected Benefits
- Eliminates "cold agent" methodology errors (wrong search paths, missing venvs, incorrect assumptions)
- Prevents cascading misdiagnosis from false negative findings
- Sub-agents produce higher-quality work because they understand the full picture
- Reduces back-and-forth corrections (get it right first time)
- Creates accountability: if a sub-agent fails, you can trace whether it received adequate context

### Implementation Complexity: **Easy**
- Create a spawning template (~15 min)
- Add to boot sequence (~5 min)
- Discipline to use the template every time (ongoing)
- Token cost: ~200-500 additional tokens per spawn (negligible vs. cost of wrong conclusions)

### Evolution: ContextPacket API — From Manual Protocol to Automatic Injection

**The manual protocol (above) is the right immediate fix, but it has a fatal flaw: it requires discipline.** Every spawn must remember to follow the template. One forgotten spawn = cold agent = potential failure.

**The systematic solution: bake context injection into the memory layer.**

#### Design: memory.build_context_packet()

```python
# Parent agent preparing to spawn sub-agent
context = memory.build_context_packet(
    task="verify antaris-guard installation",
    tags=["antaris-guard", "installation", "venv", "python-packages"],
    include_environment=True,
    include_past_failures=True,
    max_memories=10
)

# Returns structured context block
{
    "environment": {
        "venv_path": "~/.openclaw/venv-svi/",
        "scripts_path": "~/.openclaw/scripts/",
        "workspace": "~/.openclaw/workspace/",
        "key_services": {"comfyui": "http://10.0.0.35:8188"}
    },
    "task_context": "Verifying Python package installation",
    "relevant_memories": [
        {
            "content": "Python packages installed in venv-svi, not global",
            "source": "tools/venv",
            "priority": "P0"
        },
        {
            "content": "Always activate venv before pip commands",
            "source": "mistakes/004",
            "priority": "P0"
        },
        {
            "content": "Check multiple locations before concluding not found",
            "source": "mistakes/004",
            "priority": "P0"
        }
    ],
    "past_failures": [
        "MISTAKE #004: ADJUDICATOR searched globally, missed venv-svi"
    ],
    "verification_rules": [
        "Check venv AND global for packages",
        "Never conclude 'doesn't exist' from single search path"
    ]
}

# Inject into spawn (two options)
# Option 1: Structured in task description
sessions_spawn(
    task=f"""
{format_context_packet(context)}

Task: {actual_task_description}
"""
)

# Option 2: Future enhancement - native context parameter
sessions_spawn(
    task=actual_task_description,
    context_packet=context  # If sessions_spawn API supports it
)
```

#### How It Works

1. **Leverages existing BM25 search:** `antaris-memory` already has tag-based retrieval. `build_context_packet()` is a higher-level wrapper that queries for task-relevant memories and packages them for handoff.

2. **Automatic tag inference (optional):** Could analyze task description to extract relevant tags without manual specification:
   ```python
   context = memory.build_context_packet(
       task="Audit the SVI bot for security vulnerabilities",
       auto_infer_tags=True  # Extracts: "svi", "bot", "security", "audit"
   )
   ```

3. **Priority-weighted selection:** P0 memories (critical rules) always included. P1-P3 fill remaining slots based on relevance score.

4. **Learn from failures:** When sub-agent fails, capture what context was missing:
   ```python
   memory.record_subagent_failure(
       task="verify package",
       failure="Concluded package doesn't exist - missed venv",
       missing_context=["venv_path", "multiple-search-paths rule"]
   )
   # Future packets for "verify package" tasks auto-include this
   ```

#### Benefits Over Manual Protocol

| Manual Protocol | ContextPacket API |
|-----------------|-------------------|
| Requires discipline to follow template | Automatic - just call API |
| Same context every time | Learns from failures, adapts per task |
| Manual tag selection | Auto-infer tags from task description |
| Copy-paste template | Clean programmatic interface |
| Works today | Requires implementation |

#### Integration with Existing Patterns

- **Spaced Repetition (Pattern 5):** CRITICAL-RULES.md entries get tagged and surface in context packets automatically
- **MISTAKES.md:** Past failures become first-class context that auto-injects into relevant spawns
- **Consolidation (Pattern 1):** Nightly run can analyze which context was actually helpful, prune what wasn't
- **Mental Models (Pattern 2):** MODELS.md entries get tagged and surface as relevant memories

#### Implementation Roadmap

**Phase 1: Prototype (kuro-memory.py)** (~2-3 hours)
- Build `build_context_packet()` function in workspace wrapper
- Query antaris-memory BM25 index for relevant memories
- Format output as structured dict + markdown block
- Test with real sub-agent spawn

**Phase 2: Refinement** (~2-3 hours)
- Add automatic tag inference from task description
- Implement priority weighting (P0 always included, P1-P3 scored)
- Add failure learning mechanism
- Integrate with MISTAKES.md and CRITICAL-RULES.md

**Phase 3: Core Integration** (timing: when Jake's ready)
- Propose for `antaris-memory` core library
- Add native `context_packet` parameter to `sessions_spawn` (OpenClaw feature request?)
- Build cross-agent context sharing (Kuro's packets can warm Jake's Moro)

#### Why This Matters Long-Term

Manual protocols are **interim solutions.** They work when you remember them, fail when you forget, and don't scale across teams.

**Systematic solutions** remove the decision point entirely. With ContextPacket API:
- Every agent using `antaris-memory` gets warm handoffs for free
- No training required ("just call this API before spawning")
- Failure lessons propagate automatically across all future spawns
- Scales to multi-agent systems (supervisor/worker, agent swarms, human-AI teams)

The goal: make cold agent spawns architecturally impossible, not just discouraged.

---

## Implementation Priority

| # | Pattern | Impact | Complexity | Do First? |
|---|---------|--------|------------|-----------|
| 6 | Warm Handoff | 🔴 Critical | Easy | ✅ **Immediately** — 15 min, prevents cascading failures |
| 5 | Spaced Repetition | 🔴 High | Easy | ✅ Yes — 1 hour, immediate value |
| 2 | Mental Models | 🔴 High | Easy | ✅ Yes — 1 hour, compounds over time |
| 3 | Retrospectives | 🔴 High | Easy | ✅ Yes — 1 hour, weekly cadence |
| 1 | Dream State / Consolidation | 🔴 High | Medium | ✅ Second wave — most ambitious but highest ceiling |
| 4 | Gantt-State Timeline | 🟡 Medium | Medium | ⏳ When first real project starts |

**Recommended rollout:**
1. **Now:** Implement Pattern 6 (15 minutes, prevents sub-agent failures immediately)
2. **Week 1:** Implement Patterns 5, 2, 3 (all easy, independent)
3. **Week 2:** Implement Pattern 1 (consolidation run — builds on retro infrastructure)
4. **When Olympus starts:** Implement Pattern 4 (needs a real project to be useful)

---

## Architecture Summary

```
                    ┌─────────────────────┐
                    │   AGENTS.md Boot     │
                    │  (session start)     │
                    └──────┬──────────────┘
                           │ loads
                    ┌──────▼──────────────┐
                    │ CRITICAL-RULES.md   │ ← Pattern 5 (Spaced Repetition)
                    │ (ACTIVE section)    │
                    └──────┬──────────────┘
                           │ primes context
                    ┌──────▼──────────────┐
                    │   Session Work       │
                    │ (PREFLIGHT guards)   │
                    └──────┬──────────────┘
                           │ writes
                    ┌──────▼──────────────┐
                    │ memory/YYYY-MM-DD   │
                    │ (daily notes)        │
                    └──────┬──────────────┘
                           │ nightly
                    ┌──────▼──────────────┐
                    │ Consolidation Run    │ ← Pattern 1 (Dream State)
                    │ (2 AM cron)         │
                    └──────┬──────────────┘
                           │ feeds
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │MODELS.md │ │antaris-  │ │PROJECT   │
        │(Pattern 2)│ │memory   │ │DASHBOARD │
        └──────────┘ │(reinforce)│ │(freshness)│
                     └──────────┘ └──────────┘
                           │
                    ┌──────▼──────────────┐
                    │ Weekly Retro         │ ← Pattern 3 (Retrospectives)
                    │ (Monday 9 AM cron)  │
                    └──────┬──────────────┘
                           │ validates/updates
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │CRITICAL- │ │MODELS.md │ │TIMELINE  │
        │RULES.md  │ │(accuracy)│ │(velocity)│ ← Pattern 4 (Gantt-State)
        │(promote/ │ └──────────┘ └──────────┘
        │ demote)  │
        └──────────┘
```

---

## Open Questions

1. **Consolidation cost at scale:** With multiple active projects, nightly consolidation could read 50KB+. Monitor cost and consider selective reading.
2. **Model staleness:** MODELS.md entries that don't get validated against reality become dangerous (false confidence). The retro must actively check for model drift.
3. **Rule fatigue:** If ACTIVE REVIEW grows beyond ~5 rules, the per-session cost becomes meaningful. Strict promotion discipline is essential.
4. **Retro quality:** The retro is only as good as its prompt. Expect iteration — the first few retros will be too generic. Refine the prompt based on what's useful.
5. **Who reads the retro?** If Jason doesn't read them, the action items have no accountability. Consider: retro produces a 3-line summary sent via iMessage, with full details in the file.
