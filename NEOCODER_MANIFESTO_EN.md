# neoCoder: The Cognitive Agent Manifesto

<div align="center">

*"An agent isn't just an LLM with tools.  
It's an engineer with memory, principles, and a conscience."*

</div>

---

## The Problem: Why Today's AI Agents Break

Here's a hard truth I've learned building coding agents: **the model is not the bottleneck anymore.**

You can have the most powerful LLM on the planet, but if your *agent scaffold* is weak — the orchestration, memory structures, and tool abstractions surrounding the model — you'll still fail on real-world tasks.

I've seen the same backbone model produce wildly different results depending on scaffolding strategy. The difference between "toy demo" and "production-ready" isn't the LLM. It's everything *around* it.

Picture a junior developer with no memory. Every 5 minutes, they forget everything they've done. They start from scratch every single time. Don't remember which files they touched. Don't remember what they already tried. Make the same mistakes over and over again.

**That's your typical AI agent.**

```
┌──────────────────────────────────────────────────────────────────────┐
│  STANDARD AGENT                                                      │
│                                                                      │
│  User: "Add authentication"                                          │
│      ↓                                                               │
│  LLM: *re-reads 50 files from scratch*                               │
│      ↓                                                               │
│  LLM: *writes some code*                                             │
│      ↓                                                               │
│  Context overflow → TRUNCATION → critical info lost                  │
│      ↓                                                               │
│  LLM: "Wait, what were we doing? What was the plan?"                 │
│      ↓                                                               │
│  FAILURE                                                             │
└──────────────────────────────────────────────────────────────────────┘
```

### The Two Core Challenges

After building and iterating on coding agents, I've distilled the problem down to two fundamental challenges:

**Challenge 1: Long-Context Reasoning**

Agents must efficiently localize relevant code within massive repositories and perform multi-hop reasoning across dispersed modules, long tool traces, and deep execution histories. You can't just throw a bigger context window at this — you need *structure*.

**Challenge 2: Long-Term Memory**

Agents should accumulate persistent knowledge across tasks and sessions — capturing reusable patterns, failure modes, and invariants. Not rediscovering information. Not reproducing past mistakes. *Learning*.

These challenges highlight that scalability in agentic software engineering requires more than longer context windows or larger models. It requires a **principled approach** to how agents structure, maintain, and interact with information.

### Three Root Causes:

| Problem | Symptom | Consequence |
|---------|---------|-------------|
| **Amnesia** | Forgets context on long tasks | Repeats mistakes, loses progress |
| **Recklessness** | Codes without planning | God Objects, hardcoded values, spaghetti |
| **Irresponsibility** | Doesn't verify results | Broken builds, security holes |

---

## The Solution: neoCoder — A Cognitive Architecture

neoCoder is built on **five fundamental principles**:

<div align="center">

### 🧠 REMEMBER &nbsp;&nbsp; 🎯 DETERMINISM &nbsp;&nbsp; 🛡️ ANTI-HALLUCINATION &nbsp;&nbsp; 💭 THINK &nbsp;&nbsp; ✅ VERIFY

</div>

---

# 1. MEMORY is not a feature, but a foundation
## 9-level Cognitive Memory
Regular agents:

they either use the entire chat (and hit limits) or truncate context (and lose important information).

A leak of Claude Code source code showed that context compression is poorly implemented there – it doesn't preserve reasoning and its chains, read and written files, conclusions, and solutions from the task – everything is thrown away. That's why by the 15th message, Claude Code hallucinates variable names and breaks what it just understood. NeoCoder uses cognitive memory inspired by the human brain and modern research (xMemory).

- work context (what's happening right now)

- episodes (what's already been done)

- knowledge (facts and conclusions)

- themes (generalized experience)

Plus:

- notes
- patterns
- session context
```
┌─────────────────────────────────────────────────────────────────────────┐
│                         THE AGENT'S COGNITIVE BRAIN                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ╭─────────────────────────────────────────────────────────────────╮   │
│   │                    NEOCORTEX (L3: THEMES)                       │   │
│   │              "Auth & Security"  "API Design"  "Testing"         │   │
│   │                   High-level knowledge themes                   │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                 TEMPORAL LOBE (L2: SEMANTICS)                   │   │
│   │   "JWT expires in 15min"  "Use parameterized queries"          │   │
│   │                   Discrete knowledge units                      │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                  HIPPOCAMPUS (L1: EPISODES)                     │   │
│   │        "Implement OAuth" → success, files: auth.py, jwt.py      │   │
│   │                     Completed task records                      │   │
│   ╰───────────────────────────┬─────────────────────────────────────╯   │
│                               │                                         │
│   ╭───────────────────────────┴─────────────────────────────────────╮   │
│   │                 FRONTAL LOBE (L0: WORKING)                      │   │
│   │                   Active context + compression                  │   │
│   ╰─────────────────────────────────────────────────────────────────╯   │
│                                                                         │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │
│   │  THALAMUS   │  │BASAL GANGLIA│  │  PARIETAL   │  │  BRAINSTEM   │   │
│   │  Adaptive   │  │ Procedural  │  │   Notes     │  │   Session    │   │
│   │  Tuning     │  │  Patterns   │  │  Storage    │  │   Context    │   │
│   └─────────────┘  └─────────────┘  └─────────────┘  └──────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Hierarchical Retrieval vs Flat RAG

**Standard RAG:** top-k across all chunks → redundancy, imprecision, token waste.

**xMemory in neoCoder:** smart top-down cascade.

```
Query: "How do I implement authentication?"
              │
              ▼
┌─────────────────────────────────────────────────────┐
│ L3: Themes                                          │
│ ┌─────────────────────────────────────────────────┐ │
│ │ "Auth & Security" (similarity: 0.89) ✓          │ │
│ │ Uncertainty: LOW                                │ │
│ │ → Theme summary is enough (50 tokens)           │ │
│ └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘

Query: "Why did login crash last Friday?"
              │
              ▼
┌─────────────────────────────────────────────────────┐
│ L3 → L2 → L1 (high uncertainty)                     │
│ ┌─────────────────────────────────────────────────┐ │
│ │ Expanding down to Episodes:                     │ │
│ │ "Debug login 500" (2024-03-29)                  │ │
│ │ → Race condition, fixed with mutex              │ │
│ │                                                 │ │
│ │ Full context retrieved (500 tokens)             │ │
│ └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### Context Compression: Not Truncation, Transformation

Here's the key insight: when context grows too long, you don't just *cut* it — you *transform* it.

When working memory approaches configurable thresholds, an internal Architect Agent analyzes the conversation and constructs a structured summary that explicitly preserves:
- Task goals and current state
- Key decisions made and why
- Open TODOs and next steps  
- Critical error traces

The system replaces old messages with this compressed summary while maintaining a rolling window of recent messages in their original form. This is **structured summarization triggered only when needed** — not the brittleness of fixed-window truncation.

### Hindsight Notes: Learning from Failure

Flat chat logs are terrible for long-term memory. They're verbose and impossible to reuse without rereading entire transcripts.

neoCoder's note-taking mechanism distills trajectories into persistent, hierarchical Markdown notes — including **hindsight notes that capture failure modes**. When the agent fails, it doesn't just move on. It records *why* it failed and stores that knowledge for future retrieval.

```
+-- instance_auth_implementation/
    +-- hierarchical_memory/
        +-- jwt_token_handling/
            |-- analysis.md
            |-- implementation_summary.md
        +-- failure_modes/
            |-- race_condition_fix.md      ← Captured from past failure
            |-- token_expiry_edge_case.md
    +-- todo.md
```

This persistent "memory" is available for retrieval in subsequent tasks, enabling **test-time self-improvement**.

### The Memory Effect:

| Metric | Without Memory | With neoCoder Memory |
|--------|----------------|----------------------|
| Token efficiency | 100% | **40-60%** |
| Cross-session continuity | ❌ | ✅ |
| Experience reuse | ❌ | ✅ |
| Similar task speedup | 1x | **2-3x** |
| Multi-file robustness | Degrades | **Stable** |

---

## Principle 2: DETERMINISM — Predictable Decisions

> *"Randomness is the enemy of production. Same task, same result. Every time."*

neoCoder is engineered for **maximum determinism** — minimizing random variations in decisions.


### The Idiomatic Decision Rule

The agent **must** choose the idiomatic solution for the stack:

```
⛔ NEVER take the simplest solution
⛔ ALWAYS prefer the FRAMEWORK BEST PRACTICES solution

IDIOMATIC = DESIGNED: Choose the approach that works THROUGH
the tool's designed architecture — not AROUND it.

"Too complex" or "overkill for this case" are NOT valid reasons
to bypass a tool's own extension architecture.
```

**Result:** Two runs with the same task produce **structurally identical** code.

---

## Principle 3: ANTI-HALLUCINATION — Grounded Reasoning

> *"If you haven't read the code — you don't know what it does. Period."*

Hallucinations are the #1 reason AI agents fail. neoCoder has **multi-layered protection**.

### Anti-Hallucination Rules in Brainstorming

### Mental Simulation Protocol

### Semi-Formal Evidence

### Self-Questioning (AgentEvolver)

### Tool-Grounded Discovery

```python
# ❌ WRONG (hallucination)
"Based on my knowledge, React projects usually have..."

# ✅ RIGHT (grounded)
"Let me check: glob('**/package.json') → found src/package.json"
"Reading it... → React 18.2, uses TypeScript"
```

### The Result: A Chain of Evidence

```
┌──────────────────────────────────────────────────────────────────────┐
│                     GROUNDED REASONING CHAIN                         │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Task: "Add authentication"                                          │
│       │                                                              │
│       ▼                                                              │
│  [grep "auth" **/*.py] → Found: app/middleware.py:12                 │
│       │                                                              │
│       ▼                                                              │
│  [file_read app/middleware.py] → Existing CORS middleware           │
│       │                                                              │
│       ▼                                                              │
│  [Brainstorming] "I SEE existing middleware pattern at line 12"      │
│                  "Framework: FastAPI 0.100 (from pyproject.toml)"    │
│       │                                                              │
│       ▼                                                              │
│  [Mental Simulation] Happy: ✓  Edge: ✓  Failure: ✓                  │
│       │                                                              │
│       ▼                                                              │
│  [Implementation] Based on verified patterns                         │
│       │                                                              │
│       ▼                                                              │
│  [pre_commit_check] Evidence: middleware.py:45-67 ← file:line proof │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Principle 4: THINK — The Decision Funnel

> *"Code without a plan isn't development. It's gambling."*

neoCoder never writes code right away. It follows a **decision funnel**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DECISION FUNNEL                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  1. BRAINSTORMING                                             ║    │
│    ║     ┌─────────────────────────────────────────────────────┐   ║    │
│    ║     │ EXPLORE: What do we REALLY need? What constraints?  │   ║    │
│    ║     │ CHALLENGE: 2-3 approaches. Why would each work/fail?│   ║    │
│    ║     │ DECIDE: Pick the IDIOMATIC solution for the stack   │   ║    │
│    ║     │ SIMULATE: Happy path, Edge cases, Failure paths     │   ║    │
│    ║     └─────────────────────────────────────────────────────┘   ║    │
│    ║                          ↓ APPROVAL                           ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                                                                         │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  2. PLAN                                                      ║    │
│    ║     ┌─────────────────────────────────────────────────────┐   ║    │
│    ║     │ 1. Create auth/jwt.py with token generation         │   ║    │
│    ║     │ 2. Add middleware in app/middleware.py              │   ║    │
│    ║     │ 3. Update routes with @requires_auth decorator      │   ║    │
│    ║     │ ...                                                 │   ║    │
│    ║     │ N-1. Call pre_commit_check (MANDATORY)              │   ║    │
│    ║     │ N. Run build to verify                              │   ║    │
│    ║     └─────────────────────────────────────────────────────┘   ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  3. IMPLEMENT                                                 ║    │
│    ║     Follow the plan. SOLID principles. No boilerplate.        ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  4. PRE-COMMIT CHECK (Semi-Formal Verification)               ║    │
│    ║     ⛔ Every claim requires file:line evidence                ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                              ↓                                          │
│    ╔═══════════════════════════════════════════════════════════════╗    │
│    ║  5. BUILD/TEST                                                ║    │
│    ║     Only after pre_commit_check passes                        ║    │
│    ╚═══════════════════════════════════════════════════════════════╝    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Self-Questioning (AgentEvolver-inspired)

When a tool fails twice in a row, neoCoder triggers **reflection**:

```
Previous action failed: FileNotFoundError: config.yaml

## Self-Questioning Analysis

1. What assumption did I make that could be wrong?
   - About file paths, existence, or permissions?
   
2. What alternative hypotheses should I check?
   - Different path? Different file format?
   
3. What information am I missing?
   - Did I verify the file exists?
   
4. What is the REAL problem vs what I assumed?

→ Do NOT repeat the same approach that just failed.
```

---

## Principle 5: VERIFY — CRITICAL Rules

> *"Working code is the minimum. Correct code is the standard."*

neoCoder has **7 inviolable rules** (CRIT-0 through CRIT-6). If any is violated — the code is **broken**, even if it runs.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      ⛔ CRITICAL RULES                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  [CRIT-0] SECURITY-FIRST DESIGN                                         │
│                                                                         │
│  [CRIT-1] ARCHITECTURE PRINCIPLES                                      │         
│                                                                         │
│  [CRIT-2] SECRETS NEVER LEAK                                           │  
│                                                                         │
│  [CRIT-3] GUARANTEED CLEANUP                                           │               
│                                                                         │
│  [CRIT-4] EXPLICIT FAILURE HANDLING                                    │      
│                                                                         │
│  [CRIT-5] DECLARE ALL DEPENDENCIES                                     │    
│                                                                         │
│  [CRIT-6] PROTECT SHARED STATE (conditional)                           │
│                           
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Semi-Formal Pre-Commit Check

Before completing any task, the agent **must** call `pre_commit_check` with **evidence**:

```json
{
  "evidence_table": [
    {
      "rule_id": "CRIT-3",
      "claim": "All DB connections use context manager",
      "file_line": "db/client.py:45-52",
      "verified_how": "grep 'with session' + traced close()"
    },
    {
      "rule_id": "CRIT-2", 
      "claim": "API key loaded from env",
      "file_line": "config.py:12",
      "verified_how": "os.environ.get('API_KEY'), no hardcoded values"
    }
  ],
  "ready_to_commit": true
}
```

**No `file:line` evidence = code is not ready.**

---

## The Core: Orchestrator

The heart of neoCoder is the **Orchestrator**: the LLM ↔ Tools ↔ Memory interaction loop.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           ORCHESTRATOR                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  while iteration < max_iterations:                              │    │
│  │                                                                 │    │
│  │      # 1. Invoke LLM with memory context                        │    │
│  │      response = llm.invoke(                                     │    │
│  │          messages=memory.get_messages_for_llm(),                │    │
│  │          system=system_prompt,                                  │    │
│  │          tools=extensions.get_tool_definitions()                │    │
│  │      )                                                          │    │
│  │                                                                 │    │
│  │      # 2. Parse response into actions                           │    │
│  │      actions = parser.parse(response)                           │    │
│  │                                                                 │    │
│  │      # 3. Execute actions via extensions                        │    │
│  │      for action in actions:                                     │    │
│  │          result = extensions.execute(action, context)           │    │
│  │          memory.add_tool_result(result)                         │    │
│  │                                                                 │    │
│  │          # Track errors for episodic memory                     │    │
│  │          if not result.success:                                 │    │
│  │              pending_errors.append(result.error)                │    │
│  │              consecutive_errors += 1                            │    │
│  │                                                                 │    │
│  │              # Self-questioning after repeated failures         │    │
│  │              if consecutive_errors >= 2:                        │    │
│  │                  inject_self_questioning_prompt()               │    │
│  │                                                                 │    │
│  │      # 4. Check for context compression                         │    │
│  │      if memory.needs_compression():                             │    │
│  │          summary = compress_context(memory)                     │    │
│  │          cognitive_memory.retain_compression(summary)           │    │
│  │                                                                 │    │
│  │      # 5. Check for task completion                             │    │
│  │      if is_complete(response):                                  │    │
│  │          cognitive_memory.complete_task(task, result, errors)   │    │
│  │          break                                                  │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Async Parallelism

Independent read-only operations run **in parallel**:

```python
# Safe to parallelize: different files, read-only
parallel_safe = all(
    action.name in {"file_read", "grep", "glob"}
    for action in actions
)

if parallel_safe:
    results = await asyncio.gather(*[
        execute_action_async(action) for action in actions
    ])
```

---

## Life Cycle: The Journey of a Task

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TASK LIFECYCLE                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────┐                                                            │
│  │  USER   │  "Add OAuth2 authentication"                               │
│  └────┬────┘                                                            │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 1: INITIALIZATION                                          ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Adaptive Memory analyzes task → sets parameters           │  ║  │
│  ║  │ • Cognitive Memory retrieves relevant episodes              │  ║  │
│  ║  │ • xMemory provides semantic context                         │  ║  │
│  ║  │ • Session Context restored (if resuming)                    │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 2: PLANNING (Decision Funnel)                              ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Brainstorming: EXPLORE → CHALLENGE → DECIDE → SIMULATE    │  ║  │
│  ║  │ • Plan: Step-by-step with pre_commit_check                  │  ║  │
│  ║  │ • Session Context saved to .neoCoder/context.md             │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 3: EXECUTION (Iterative Loop)                              ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ while not complete:                                         │  ║  │
│  ║  │   • LLM generates next action                               │  ║  │
│  ║  │   • Extension executes (bash, file_edit, grep...)           │  ║  │
│  ║  │   • Working Memory updated                                  │  ║  │
│  ║  │   • Context compressed if needed                            │  ║  │
│  ║  │   • Self-questioning on repeated failures                   │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 4: VERIFICATION                                            ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • pre_commit_check with semi-formal evidence                │  ║  │
│  ║  │ • Build verification                                        │  ║  │
│  ║  │ • Test execution (if applicable)                            │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ╔═══════════════════════════════════════════════════════════════════╗  │
│  ║  PHASE 5: CONSOLIDATION                                           ║  │
│  ║  ┌─────────────────────────────────────────────────────────────┐  ║  │
│  ║  │ • Episode created (task, summary, files, decisions, errors) │  ║  │
│  ║  │ • Hierarchical Memory updated                               │  ║  │
│  ║  │ • xMemory extracts semantic units                           │  ║  │
│  ║  │ • Procedural Memory updated (if pattern detected)           │  ║  │
│  ║  │ • Session Context cleared for next task                     │  ║  │
│  ║  └─────────────────────────────────────────────────────────────┘  ║  │
│  ╚═══════════════════════════════════════════════════════════════════╝  │
│       │                                                                 │
│       ▼                                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  RESULT: OrchestratorResult                                     │    │
│  │  {                                                              │    │
│  │    success: true,                                               │    │
│  │    output: "OAuth2 implemented with JWT tokens",                │    │
│  │    iterations: 47,                                              │    │
│  │    total_tokens: 125000,                                        │    │
│  │    input_tokens: 98000,                                         │    │
│  │    output_tokens: 27000,                                        │    │
│  │    artifacts: { files_changed: [...], tests_passed: true }      │    │
│  │  }                                                              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---


## Workflow Preset: 8-Stage Engineering Pipeline

For complex enterprise tasks:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     WORKFLOW PRESET (8 STAGES)                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1️⃣  PROBLEM BREAKDOWN     │ Decompose into subtasks     │ ❌ Approval  │
│                             │                             │              │
│  2️⃣  BRAINSTORMING         │ High-level approach, risks  │ ❌ Approval  │
│                             │                             │              │
│  3️⃣  IMPLEMENTATION PLAN   │ Detailed step-by-step       │ ❌ Approval  │
│                             │                             │              │
│  4️⃣  IMPLEMENTATION        │ Execute the plan            │ ✅ Auto      │
│                             │                             │              │
│  5️⃣  INTEGRATION TESTING   │ Run tests with retry logic  │ ✅ Auto      │
│                             │                             │              │
│  6️⃣  TECH STACK REVIEW     │ Validate best practices     │ ✅ Auto      │
│                             │                             │              │
│  7️⃣  CODE REVIEW           │ CRIT rules, anti-patterns   │ ✅ Auto      │
│                             │                             │              │
│  8️⃣  MERGE PREPARATION     │ Commit-ready summary        │ ✅ Auto      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Stateful execution:** Workflow state is saved to `.neoCoder/context.md` — you can pause and resume anytime.

---

## The Bottom Line: neoCoder Philosophy

neoCoder isn't "just another AI agent." It's the embodiment of a simple principle:

<div align="center">

### *"An agent should work like a SENIOR ENGINEER: remember context, think before coding, verify the result."*

</div>

## Oreilly books : 

- Beyond Vibe Coding

- Building AI Agents with LLMs, RAG, and Knowledge Graphs

- Building Agentic AI - Workflows, Fine-Tuning, Optimization, and Deployment

- Agentic Architectural Patterns for Building Multi-Agent Systems

- Generative AI Design Patterns

- Designing Intelligent Delivery Systems

- Your Code as a Crime Scene, Second Edition, 2nd Edition

- AI Agents - The Definitive Guide

- Advances in Artificial Intelligence Applications in Industrial and Systems Engineering

- Enterprise AI in the Cloud

- Generative AI Application Integration Patterns

- Adaptive Artificial Intelligence



# Arxiv papers:

- From Prompt–Response to Goal-Directed Systems:
- The Evolution of Agentic AI Software Architecture
- Beyond RAG for Agent Memory: Retrieval by Decoupling and Aggregation

- MemoryArena: Benchmarking Agent Memory in Interdependent Multi-Session Agentic Tasks

- Agentic Code Reasoning

- SWE-Compass: Towards Unified Evaluation of Agentic Coding Abilities for Large Language Models

- A Practical Guide for Designing, Developing, and Deploying Production-Grade Agentic AI Workflows

- AlphaEvolve: A coding agent for scientific and algorithmic discovery

- The Potential of CoT for Reasoning: A Closer Look at Trace Dynamics

- Team of Thoughts: Efficient Test-time Scaling of Agentic Systems through Orchestrated Tool Calling

- Think Fast and Slow: Step-Level Cognitive Depth Adaptation for LLM Agents

- AI Agent Systems: Architectures, Applications, and Evaluation

- Fundamentals of Building Autonomous LLM Agents

- Beyond Task Completion: An Assessment Framework for Evaluating Agentic AI Systems

</div>
