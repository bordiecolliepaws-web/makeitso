# Make It So — Design V2: Greedy Lock-In Architecture

> From intent to product: **Charter. Explore. Engage.**

## Evolution

V1 assumed Phase 1 (Explore) was "generate a PRD with options." Testing against a real research project (ADDM) revealed that exploration means **running experiments, diagnosing failures, and iterating on coupled design variables.** This led to the greedy lock-in model.

---

## The Three Phases

```
Charter (knowledge transfer)
    ↓
Phase 1: Explore (design the sequential plan)
    ↓
Phase 2: Engage (execute with greedy lock-in)
    ↓
Product
```

---

## Phase 0: Charter

### What It Is

A **knowledge transfer document** — everything a brilliant autonomous agent needs to operate without derailing. Not a brief. Not a one-liner. The full spirit of the project.

### Structure

```markdown
# Charter: [Project Name]

## I. Thesis (WHY — don't change this)
The core idea. What we're proving/building and why it matters.
The research contribution or product vision.

## II. Method (HOW — understand deeply before touching)
### [Component A]
How it works, why it's designed this way, key invariants.

### [Component B]
...

### Infrastructure
How experiments run / how the system works end-to-end.
Data pipelines, execution flow, costs.

## III. Current State (WHERE we are)
- Benchmark results / current functionality
- Known strengths
- Known weaknesses

## IV. Mission (WHAT to do)
The specific goal for this run.

## V. Degrees of Freedom (WHAT can change)
### CANNOT change (foundational)
Core architecture, thesis, fundamental concepts.

### CAN change (with caution, must not regress)
Parameters, thresholds, calibration, component internals.

### SHOULD change (likely needed for this mission)
Domain-specific hardcoding, known limitations.

### FREE to change
Implementation details, prompts, experiment parameters.
```

### Key Properties

- **Same structure whether building from scratch or extending.** For greenfield: Section III is empty. For extension: Section III has the baseline to protect.
- **Section V (Degrees of Freedom) is the core innovation.** It's a permission system for the agent — like constitutional law where fundamental rights need supermajority but regulations can change freely.
- **The charter IS the constitution.** It persists across all phases as the guardrail.

### Example: ADDM Amazon Expansion

```markdown
## I. Thesis
AMOS (Agenda-driven Method for Opinion Summarization) proves 
that structured agenda-driven retrieval outperforms naive 
baselines at judging entity attributes from reviews.

## II. Method
### Agenda Spec
[Full explanation: structure, version history v1→v7, 
 what makes a good agenda, verdict rules, term definitions]

### Ground Truth Generation
[How derived from agendas, LLM judgment process, 
 aggregation, human overrides, cost per recalculation]

### AMOS Phase 1: ASC (Agenda Spec Compiler)
[How arbitrary queries → agenda spec, generalization 
 mechanism, what's hardcoded vs flexible]

### AMOS Phase 2: Tekoa
[Retrieval gates, LLM verification calibration, 
 skip logic, verdict aggregation]

### Baselines
[Direct, CoT, etc. — how each works, what they test]

### Experiment Infrastructure
[run_experiment pipeline, data flow, how to interpret 
 results, cost structure]

## III. Current State
- Yelp: AMOS beats baseline on X/18 topics
- [Benchmark numbers per topic]
- Strengths: [...]
- Weaknesses: [agenda may be Yelp-specific]

## IV. Mission
Expand to Amazon product reviews while maintaining Yelp performance.

## V. Degrees of Freedom
### CANNOT change
- AMOS 2-phase architecture (ASC → Tekoa)
- Agenda spec concept
- Evaluation methodology
- Existing Yelp results must not regress

### CAN change (with caution)
- Tekoa gate thresholds and calibration
- ASC prompt templates
- Agenda format extensions

### SHOULD change
- Any Yelp-hardcoded logic in ASC
- Topic definitions for Amazon domain
- Ground truth for new domain

### FREE to change
- Internal implementation details
- Experiment parameters
- Prompt wording
```

---

## Phase 1: Explore

### What It Is

Human + Agent **design a sequential plan** with lock-in criteria and dependency rules. The plan defines what to build/explore and in what order.

### The Greedy Lock-In Insight

Don't solve everything at once. Solve step 1, **lock it in**, build on top. If a later step fails, go back to the nearest unlocked dependency — not to the beginning.

### Plan Structure

Each step has:
- **Goal**: What this step achieves
- **Lock-in criterion**: How we know it's done (measurable)
- **Failure diagnosis**: If this fails, what might be wrong?
- **Fallback rules**: What to revisit if this step fails
- **Frozen dependencies**: What prior steps are locked and must not change

### Example: ADDM from Scratch

```
PART 1: MVP

Step 1: Find agendas where baseline fails
  Lock-in: ≥3 agenda topics where direct baseline < 70% accuracy
  Failure: try different agenda questions, different topics
  Frozen: nothing yet

Step 2: Systemize agenda format (v1 → v7 evolution)
  Lock-in: structured format handles all Step 1 agendas
  Verify: baseline still fails on these agendas
  Failure: format too rigid → adjust format, re-verify Step 1
  Frozen: Step 1 topic choices (but agenda wording can adjust)

Step 3: AMOS Phase 1 (ASC) works for systemized format
  Lock-in: ASC produces consistent agenda spec across all formats
  Failure: ASC prompt needs work → adjust ASC, don't touch format
  Frozen: Steps 1-2 (agenda format locked)

Step 4: AMOS Phase 2 (Tekoa) outperforms baselines
  Lock-in: AMOS > baseline on all Step 1 topics
  Failure: gates too aggressive → adjust Tekoa calibration
  Fallback: if Tekoa fundamentally can't work → revisit Step 3
  Frozen: Steps 1-2 (but Step 3 can be revisited)

PART 2: EXPAND

Step 5: Full agenda pipeline (policy → prompt, policy + context → ground truth)
  Lock-in: pipeline runs end-to-end for any topic
  Frozen: Steps 1-4

Step 6: For each topic T in [allergy, noise, parking, ...]:
  Step 6.T.1: Design agenda for T (verdict rule, term definitions)
  Step 6.T.2: Verify baseline fails on T
  Step 6.T.3: Verify AMOS works on T
  Lock-in: AMOS > baseline on topic T
  Failure: adjust agenda for T, don't touch other topics
  Frozen: Steps 1-5, other topics
```

### Key Properties

- **Sequential with dependencies.** Each step builds on locked-in prior steps.
- **Failure cascades are bounded.** Step 4 failing can revisit Step 3, but NOT Steps 1-2.
- **Loops get flattened.** "For each topic" becomes explicit steps per topic. This makes it Ralph-compatible.
- **Human involvement is in designing the plan**, not executing it. The agent proposes steps, human approves/adjusts the sequence and lock-in criteria.

### How Phase 1 Works (The Conversation)

```
Agent reads charter
    ↓
Agent proposes sequential plan
    ↓
Human: "Step 2 and 3 should swap" / "Add a step for X"
    ↓
Agent adjusts plan
    ↓
Human: "Lock-in criterion for Step 4 is too loose"
    ↓
Agent tightens criteria
    ↓
Human: "Looks good. Engage."
    ↓
Plan locked → Phase 2
```

---

## Phase 2: Engage

### What It Is

The agent **executes the plan** step by step. Each step runs as a Ralph-style loop until its lock-in criterion is met.

### Execution Model

```
for each step in plan:
    while not step.lock_in_criterion_met():
        # Ralph-style iteration
        spawn fresh agent context
        feed: charter + plan + current step + progress
        agent works on current step
        run verification (experiments, tests, etc.)
        
        if step.lock_in_criterion_met():
            lock step → move to next
        elif step.failure_diagnosed():
            check fallback rules:
                if can_fix_current_step(): continue
                if must_revisit_prior(): unlock that step, go back
                if fundamental_failure(): alert human
        
        update progress.txt
```

### Greedy Lock-In Rules

1. **Forward progress is default.** Complete step, lock it, move on.
2. **Fallback is bounded.** Can only revisit what the plan's fallback rules allow.
3. **Frozen means frozen.** Locked steps with no fallback pointing to them cannot change.
4. **Human escalation.** If the agent is stuck (looping on same step > N iterations), alert human.
5. **Regression check.** After any fallback, re-verify all downstream locked steps.

### Flattening Dynamic Elements

"For each topic" in the plan becomes a flat story list for Ralph:

```json
{
  "userStories": [
    {"id": "S6-allergy-1", "title": "Design agenda for allergy topic", ...},
    {"id": "S6-allergy-2", "title": "Verify baseline fails on allergy", ...},
    {"id": "S6-allergy-3", "title": "Verify AMOS works on allergy", ...},
    {"id": "S6-noise-1", "title": "Design agenda for noise topic", ...},
    ...
  ]
}
```

This keeps Phase 2 compatible with Ralph's existing flat-story implementation.

---

## Full Flow Summary

```
Human thoughts
    ↓
Phase 0: Charter
    Human + Agent create knowledge transfer document
    Output: charter/ (thesis, method, state, mission, degrees of freedom)
    ↓
Phase 1: Explore  
    Human + Agent design sequential plan with lock-in criteria
    Agent proposes, human approves/adjusts
    Output: plan.json (steps, lock-in criteria, fallback rules, frozen deps)
    ↓
Phase 2: Engage
    Agent executes plan step-by-step
    Each step = Ralph loop until lock-in
    Greedy: lock and move forward
    Fallback: bounded by plan's rules
    Output: product
```

### CLI

```bash
makeitso charter              # Phase 0: create/edit charter
makeitso explore              # Phase 1: design the plan
makeitso engage               # Phase 2: execute with greedy lock-in
makeitso engage --step 3      # Resume from step 3
makeitso status               # Show plan progress + lock-in state
```

---

## What Makes This Different

| Framework | Planning | Execution | Failure Handling |
|-----------|----------|-----------|-----------------|
| Vibe coding | None | One-shot | Start over |
| Ralph | Human writes PRD | Loop until done | Retry same story |
| BMAD | Multi-agent planning | Agent codes | No structured fallback |
| Spec Kit | 4-phase gated | Agent codes | Human reviews each gate |
| **Make It So** | **Charter + sequential plan with lock-in** | **Step-by-step Ralph loops** | **Bounded fallback + regression check** |

The key innovations:
1. **Charter as knowledge transfer** (not a brief)
2. **Degrees of freedom** (permission system for what agent can change)
3. **Greedy lock-in** (solve → lock → build on top)
4. **Bounded fallback** (failure can only cascade to specified prior steps)
5. **Plan-aware execution** (Ralph loop per step, not one flat PRD)

---

## Prior Art

- **Ralph Wiggum Loop** — Phase 2 execution model. We use it directly.
- **BMAD Method** — Multi-agent planning is closest to Phase 1, but no lock-in or fallback structure.
- **GitHub Spec Kit** — 4-phase gated process, but gates are human-reviewed, not criterion-based.
- **Anthropic Constitutional AI** — Inspiration for charter as persistent constraint.
- **Scientific method** — Phase 2's observe → diagnose → hypothesize → act → validate loop.
- **Greedy algorithms** — Lock-in the best local solution, build on it.

---

*Authors: Jimmy & Bordie 🐕*
*Created: 2026-02-12*
