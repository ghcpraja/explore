# FORGE: Local Agent Harness & Autonomous Loop Framework
## Enterprise-Grade, Offline-First, Self-Improving AI Agent Blueprint
**Version:** 1.0 | **Date:** 2026-06-28 | **Target:** Mac Pro (Apple Silicon) + Local Models | **Orchestration:** LangGraph (approved) + Pure Python Scaffolding

**Core Philosophy (Synthesized from referenced X deep-dives + production best practices):**
- The **harness** (everything around the model) is the product. Models are commodities; the loop, context selection, verification, memory, and tooling determine reliability.
- **Context is the scarce resource.** Raw long-context + similarity retrieval creates rot and compounding errors. Solution: Deliberate **selection** (relevance + provenance + structure) over recall. Small high-signal context > large noisy context.
- **Sub-agents / Workflows + clean contexts** unlock scale without polluting the orchestrator. "Smart boss (capable model) + cheap/narrow workers."
- **Independent verification loops** (maker ≠ checker) prevent self-preferential bias and goal drift.
- **Self-improving systems** improve the *environment* (memory, skills, selectors, rubrics), not the model weights.
- **Local-first, offline, private:** No cloud dependencies for core operation. Sandboxed execution. Human-gatable high-impact actions. Apple Silicon optimized where possible.
- **Phased maturity with concrete self-testing:** Start simple and reliable → add multi-agent + structured memory → evolve into autonomous self-improving loop designer.
- **Extensible for your domains:** Architectural design viz (ComfyUI/Blender integration), trading systems (backtesting, SMC/CML indicators), business automation (content, SEO playbooks), general research/coding. Base framework is domain-agnostic with plug-in skills.
- **LangGraph + Local Python Harness:** Explicit stateful graphs for control, persistence (checkpointers), sub-graphs for sub-agents, easy verification nodes. Pure Python scaffolding for skills/tools/memory (no black-box frameworks beyond approved LangGraph/LangChain local integrations).
- **Production standards:** Typed (Pydantic), logged/observable, versioned (git-friendly), testable, permissioned, auditable. Agent-readable specs and modular code so the system can eventually improve itself.

This blueprint lets **you** (or a capable agent) build it incrementally. It matures from reliable single-agent tool to autonomous loop designer that compounds knowledge across projects.

---

## High-Level Architecture

```
User / CLI / UI
      ↓
Orchestrator (LangGraph Graph + Main Loop)
  - Prompt Assembler (selection-focused context)
  - LLM Call (local capable model)
  - Tool Dispatcher (sandboxed)
  - Memory Updater + Selector
  - Verification Node (independent sub-agent or rules)
  - Delegation Router → Sub-Agents (clean context, narrow goal, possibly smaller model)
      ↓ (parallel or sequential)
Sub-Agents / Workflows (isolated state, fresh context)
  - Specialist skills (design reviewer, code tester, backtester, content synthesizer, etc.)
      ↓
Tools Layer (permissioned, sandboxed)
  - File I/O (workspace only)
  - Code Execution (restricted Python / subprocess)
  - Domain Skills (ComfyUI API caller, Blender script runner, Pine Script generator, local search, etc.)
  - Memory Ops
      ↓
Memory Layer (Structured + Selection)
  - Obsidian-compatible Markdown Vault (human + agent readable)
  - Frontmatter: tags, provenance, supersession, confidence, timestamps
  - Hybrid Retrieval (BM25/keyword primary + local embeddings rerank)
  - LLM/Rule-based Selector (what actually enters context for *this* goal/state)
  - Hierarchical + Temporal (hot/warm/cold)
  - State/Progress Files per project/session
      ↓
Persistence & State
  - LangGraph Checkpointer (SQLite local or file-based for replay/time-travel)
  - Git-friendly versioning for skills/prompts/configs
  - Full audit logs + traces
```

**Key Innovations Addressing Common Failures:**
- **Selection Layer** (not just retrieval): Decides *what* enters the window based on current goal, state, provenance. Prevents "near-miss" distractors from similarity search.
- **Maker-Checker Split**: Every critical output/decisions gets fresh-context independent verification.
- **Clean-Context Sub-Agents**: Heavy lifting (reading 300 pages, exhaustive review) happens off the main thread → only condensed verdict returns.
- **Skill Distillation + Meta-Improvement**: Failures/successes → general skills/rules that persist and improve future runs.
- **Fail-Safe Guardrails**: Permission matrix, confirmation gates, token budgets, sandbox.

---

## Recommended Local Tech Stack (Offline, Mac-Optimized)

- **LLM Serving**: Ollama (easiest, Metal/Apple Silicon support) or MLX-LM for maximum speed on Mac. Start with Ollama.
  - Orchestrator/Planner: Strong model e.g. `llama3.1:70b` (Q4_K_M or better if RAM allows) or `qwen2.5:32b` / `mixtral:8x7b`. Quantize aggressively.
  - Sub-agents/Workers: Smaller/faster e.g. `qwen2.5:7b`, `gemma2:9b`, `phi4`, or same family quantized lighter. Route by complexity.
- **Orchestration**: LangGraph (core) + LangChain local components (Ollama integration, tool calling). Pure Python classes for custom harness logic.
- **Memory/Storage**: SQLite (for state/checkpoints) + Markdown files (vault) + optional local LanceDB/Chroma or pure BM25 (rank-bm25 lib) for reliability. Avoid over-reliance on embeddings alone.
- **Sandbox/Code Exec**: `subprocess` with restricted env + PATH limits + optional Docker for heavier isolation (if you set it up). RestrictedPython for pure-Python exec.
- **Scheduling/Autonomous Loops**: `APScheduler` or simple `while` + sleep for timers/background jobs.
- **UI/Interface**: CLI first (Typer or argparse). Optional Gradio/Streamlit for visual monitoring + chat. Keep fully local.
- **Versioning/Observability**: Git for code/skills/prompts. Structured JSONL logs + simple local dashboard (Streamlit or CLI commands). LangGraph traces exportable to files.
- **Other**: Pydantic v2 for all schemas, `loguru` or structlog, `tenacity` for retries, `pytest` + custom evals.

**Learning Path (if new to LangGraph)**: Official LangGraph docs (quickstart + persistence + multi-agent examples) + their ReAct tutorial. 1-2 days to productive. The graph is explicit: nodes = functions (think, tool, verify, delegate), edges = conditional routing (tool calls? verify? done?).

---

## Phase 1: Core Single-Agent Harness (Foundation — Target: 1-2 Weeks to Reliable MVP)

**Goal**: A trustworthy single agent that executes multi-step tasks (10-30+ turns) without rapid drift, using basic tools and memory. Focus on reliability and observability over features.

**Key Deliverables**:
- Working `forge/` Python package installable locally.
- Config-driven local model + tool permissions.
- LangGraph-based ReAct-style loop with explicit context assembly, error recovery, basic verification.
- Core tools + simple skill registry.
- Basic structured memory (markdown notes + SQLite scratchpad).
- Self-testing eval suite with concrete pass/fail + traces.
- CLI to run agent on a goal + workspace.

**Detailed Components**:

1. **Orchestration Loop (LangGraph StateGraph)**:
   - State: `messages` (history), `goal`, `scratchpad` (current plan/notes), `tool_results`, `verification_status`, `turn_count`, `metadata`.
   - Nodes:
     - `assemble_prompt`: Selects relevant memory + positions critical info (start/end of window). Uses simple rules + optional small LLM for prioritization.
     - `llm_call`: Calls local model (tool-calling format preferred).
     - `parse_and_dispatch`: Handles tool calls or final answer. Routes to verification if needed.
     - `execute_tool`: Sandboxed dispatch + permission check.
     - `update_memory`: Persists observations, decisions, provenance.
     - `verify` (optional for Phase 1): Simple rules or small LLM critique against goal/rubric.
     - `decide`: Continue / handoff / done / escalate.
   - Edges: Conditional based on output (tool calls present? verification passed? max turns?).
   - Persistence: `SqliteSaver` or custom file checkpointer → full replay, resume after interrupt, time-travel debugging.
   - Termination: No tool calls + verifier ok, max turns, token budget, guardrail trip, user interrupt.

2. **Tools (Core + Extensible)**:
   - `read_file`, `write_file`, `list_dir` (sandboxed to `./workspace/{project}/` — never outside without explicit gate).
   - `execute_python` (restricted: no network, limited imports, cwd in workspace, output capture + timeout).
   - `search_memory` (basic).
   - `shell` (very restricted, read-only by default; write requires confirmation gate).
   - Skill registration: Decorator or YAML/JSON registry that turns Python functions or prompt+tools bundles into callable tools with schema.

3. **Memory (Basic Structured)**:
   - Session: LangGraph `messages` + scratchpad.
   - Persistent: Per-project SQLite or JSONL for key facts/decisions + Markdown notes vault (`notes/{project}/` with YAML frontmatter: `tags`, `provenance`, `date`, `supersedes`).
   - Retrieval: Keyword search first (reliable), simple embedding if desired (local model).
   - Compaction: On threshold, summarize old tool outputs/observations while preserving architectural decisions, open issues, user preferences.

4. **Context Management**:
   - Assembler always puts: System prompt + tool schemas + high-priority memory (selected) + recent turns + current goal/scratchpad at strategic positions.
   - Observation masking or summarization for old tool results.
   - JIT: Load full files only when tool needs them (agent uses `read_file`).

5. **Prompt Construction & Output Parsing**:
   - Hierarchical system prompt (role, principles, available skills, verification standards).
   - Native tool calling or Pydantic-structured output + retry-on-parse-error.
   - Error feedback loop: Failed parse/tool → append error as observation → model self-corrects.

6. **Error Handling & Guardrails**:
   - Transient errors: Retry with backoff (tenacity).
   - LLM-recoverable: Return as ToolMessage for model to fix.
   - High-risk (write outside workspace, delete): Human confirmation gate.
   - Unexpected: Log + escalate with full trace.

7. **Self-Testing & Concrete Guidelines (Phase 1)**:
   - **Eval Suite** (`evals/phase1/`): 15-20 tasks across categories:
     - File ops chains (create project structure, edit configs, verify).
     - Simple research/synthesis from local docs.
     - Code generation + basic test execution (e.g., write a small function + run pytest on it).
     - Multi-step planning with verification (e.g., "Plan a trading indicator backtest setup, write skeleton, verify logic").
   - **Metrics** (logged automatically):
     - Success rate (% goals fully achieved without manual intervention).
     - Avg turns / tokens per success.
     - Drift indicators (contradictions detected via simple rules or LLM judge on final vs initial goal).
     - Context efficiency (high-signal tokens retained vs total window).
   - **Run Protocol**: `python -m forge.eval phase1` → runs all, compares to baseline, generates trace HTML/JSON + failure analysis.
   - **Pass Criteria for Phase 1 Complete**: >85% success on core tasks, <5% unexplained drift on 10+ turn tasks, full traces replayable, basic memory persists across sessions.
   - **Improvement Loop**: After each eval run, agent (or you) reviews failures → proposes 1-2 skill/prompt tweaks → re-test in isolated branch.

**Milestone**: You have a reliable "co-pilot" agent for daily work in one domain (e.g., exploring a design project folder or prototyping a trading script). It rarely hallucinates actions or forgets key constraints.

---

## Phase 2: Multi-Agent Orchestration, Structured Memory & Verification (Maturity — Target: 2-4 Weeks)

**Goal**: Handle complex, long-horizon, multi-perspective tasks reliably. Clean context for heavy work. Independent verification. Memory that scales without rot. Easy domain extension.

**Key Additions**:

1. **Sub-Agents & Dynamic Workflows**:
   - **Sub-Agent Definition**: Markdown files in `agents/{name}.md` with YAML frontmatter:
     ```yaml
     name: plan_roaster
     description: "Expert reviewer that finds weakest assumptions in plans. Read-only. Max 8 turns."
     model: qwen2.5:7b  # or hint for routing
     tools_allowed: [read_file, search_memory]
     permissions: read_only
     max_turns: 8
     output_schema: critique_report  # Pydantic or JSON
     ```
   - **Orchestrator Delegation**: LLM (or rules) decides to spawn sub-agent based on description match + current need. Spawns isolated LangGraph (or separate process/thread) with **fresh context window**, only the goal + minimal handoff context + allowed tools.
   - **Patterns Supported** (inspired by dynamic workflows):
     - Fan-out & synthesize (parallel specialists → merge).
     - Adversarial verification (maker + independent checker with fresh context + rubric).
     - Tournament / generate-and-filter (multiple candidates, pairwise or rubric judging).
     - Loop-until-done (with independent stop condition).
     - Classify-and-route.
   - **Economics/Routing**: Orchestrator on strongest model. Heavy readers/reviewers on smaller/faster. Parallelism managed (resource limits on 24GB Mac — queue or sequential fallback).
   - **Handoff**: Only condensed summary/verdict returns to main. Full work stays in sub-agent's isolated state/logs.

2. **Advanced Structured Memory & Selection Layer** (Addresses Context Rot Loop):
   - **Vault**: Obsidian-compatible `memory/vault/{project}/` — folders + linked Markdown notes. Human can browse/edit in Obsidian; agent reads/writes via tools.
   - **Frontmatter Schema** (enforced):
     - `tags`, `domain`, `provenance` (agent_id, run_id, source_file, timestamp), `supersedes` (links to prior versions), `confidence`, `status` (hypothesis / verified / deprecated), `related` (wikilinks).
   - **Retrieval + Selection**:
     - Primary: BM25 / keyword (reliable, no embedding hallucinations).
     - Secondary: Local embeddings (sentence-transformers or local LLM) for semantic.
     - **Selection Layer** (critical): Before injecting into context, a small "selector" (rule-based or tiny LLM) answers: "Given current goal + state, which of these candidates are *relevant and non-redundant* right now?" Prioritizes provenance, recency of verification, direct dependencies. Prevents near-miss pollution.
     - Hierarchical: Global knowledge → Project → Session scratchpad. Hot (in-memory recent) → Warm (frequently accessed) → Cold.
     - Compaction: Dedicated compaction agent or node that produces high-signal summaries preserving decisions/unresolved issues.
   - **State/Progress Files**: Per-project `STATE.md` or JSON that loop reads at start / writes at end (as in loop designer patterns). Enables continuity across days/sessions without full history dump.
   - **Provenance & Supersession**: Every fact/note tracks what it replaced and why. Selector prefers verified/superseding entries.

3. **Verification Loops (Mandatory for Maturity)**:
   - Post-critical-action or pre-final-answer: Spawn independent verifier sub-agent (fresh context, read-only on output + rubric + goal).
   - Rubrics: Task-specific (e.g., for design plan: scale accuracy, contextual fidelity, dopamine potential, material realism; for code: correctness, tests pass, edge cases).
   - Adversarial option: One sub-agent attacks assumptions.
   - Feedback: Verifier output → main loop for correction or acceptance. Never let maker grade itself on high-stakes.

4. **Enhanced Error/Drift Handling & Guardrails**:
   - Context rot detection (simple heuristics + LLM judge on coherence).
   - Self-correction via error observations + reflection node.
   - Permission matrix expanded (per-skill/tool). High-impact (e.g., committing code, sending content) → human gate or explicit approval skill.
   - Token budgets per run/sub-agent.

5. **Self-Testing & Concrete Guidelines (Phase 2)**:
   - Extended eval suite: Long-horizon (50+ simulated steps), multi-perspective reviews, parallel sub-agent workflows, context-rot stress tests (inject distractors, measure if selector filters them).
   - Metrics: Context efficiency (signal:noise ratio), verification pass rate, sub-agent utilization & token savings, compounding error (does performance hold over 30+ turns?), memory growth vs recall quality.
   - **Protocol**: Automated nightly/weekly evals. Compare Phase 1 vs Phase 2 harness on same tasks. Generate reports with traces, failure root-cause (context pollution? missing verification? poor selection?).
   - **Pass Criteria**: >90% success on complex tasks, verifiable reduction in drift (e.g., <2% contradiction rate), sub-agents demonstrably save tokens/context while improving quality, memory selector measurably outperforms naive top-k.
   - **Improvement**: Failures automatically logged → trigger meta-review (Phase 3 early) or manual skill addition.

**Milestone**: Agent can reliably handle complex workflows like "Review full design project folder, generate multiple AXO/rendering concepts with verification, produce client-ready spec with options" or "Develop + backtest custom trading indicator with rigorous verification" using sub-agents and clean memory. Context stays manageable; quality compounds.

---

## Phase 3: Self-Improving Adaptive Framework (Evolution — Ongoing, Start Parallel in Phase 2)

**Goal**: The system becomes a "loop designer" that improves *itself* and your workflows over time. Distills experience into reusable skills. Adapts to new domains with minimal human input. Autonomous background loops.

**Key Additions**:

1. **Meta-Agent / ForgeMaster**:
   - Special privileged agent (or background loop) with tools to inspect: run logs, failure traces, eval results, current skills/prompts/graphs, memory vault.
   - Capabilities:
     - Analyze patterns: "What failures recur? What skills would have prevented X?"
     - Propose improvements: New skills (as Markdown + code), prompt tweaks, better selector rules, new verifier rubrics, graph node additions.
     - Test proposals: In isolated sandbox eval environment (copy of current harness + proposed change). Run relevant evals. Compare metrics.
     - Integrate if passes: Auto or human-gated commit (git-style or explicit skill registry update). Version everything.
   - Triggered: After major runs, on eval failures, on schedule, or manually ("Forge, review last week and improve memory selection").

2. **Skill & Rule Distillation**:
   - From state files + verified memory + traces: Extract general procedures, anti-patterns, decision rules.
   - Turn into reusable skills (prompt template + tools + verification rubric) or global notes that all agents read.
   - Example: Recurring "always verify scale/context in design plans" → permanent skill or system prompt addition.

3. **Autonomous Loops (Timer + Goal + Verifier)**:
   - Scheduler starts background agents on cadence (e.g., daily project review, backlog processing, self-eval).
   - Each loop: Clear goal + independent verifier + state file read/write at start/end.
   - Fail-safe: Strict read-only or low-permission by default for unattended runs. Human merge/approval for outputs affecting real projects.

4. **Advanced Self-Optimization**:
   - Can propose/test changes to its own components (e.g., "optimize the prompt assembler for lower token use while preserving signal").
   - A/B testing harness: Run parallel variants on eval subsets, promote winner.
   - Domain adaptation: Auto-generate or refine domain-specific system prompts/skills from project memory (e.g., "luxury interior design principles from our past projects").
   - Recursive improvement: Use the agent to improve Forge itself (with heavy gates).

5. **Self-Testing & Concrete Improving Guidelines (Phase 3)**:
   - **Core Eval Set**: Fixed, growing benchmark across domains. Run on every major change or weekly. Track trend lines (success rate, efficiency, drift resistance over time).
   - **Meta-Eval**: "Did the proposed improvement actually raise metrics on held-out tasks without regressions?"
   - **Protocol for Self-Improvement**:
     1. Trigger meta-review (manual or scheduled).
     2. ForgeMaster proposes 1-3 concrete changes (with rationale + expected metric lift).
     3. Sandbox test on relevant evals + regression suite.
     4. If passes thresholds (e.g., +5% success, no regressions >2%, human spot-check for safety), integrate + log.
     5. Full audit trail of every self-mod.
   - **Safety Gates**: All self-mods require verification + optional human approval for core harness changes. Never auto-modify permissions or high-risk tools.
   - **Success Criteria**: Measurable compounding — e.g., after 10 cycles, same tasks complete in fewer turns/higher quality; new domains onboard faster; recurring failure classes drop to near-zero.
   - **Concrete Example Loop**: "Every Sunday: Review all project STATE.md files + recent failures. Distill 3 new skills or rubric improvements. Test them. Integrate winners. Update global memory with lessons."

**Milestone**: You have an autonomous "Forge" that runs background loops, learns from every project, proposes and validates its own upgrades, and lets you focus on high-level goals and creative direction while it handles execution, verification, and continuous improvement. It becomes your "loop designer" partner.

---

## Implementation Roadmap & Getting Started

1. **Week 1 (Phase 1 Core)**:
   - Set up repo + virtualenv + Ollama + basic models.
   - Implement minimal LangGraph ReAct with 2-3 tools + session memory.
   - Add CLI runner.
   - Build first 5 evals + run them.
   - Iterate until Phase 1 pass criteria met.

2. **Week 2-3 (Phase 2)**:
   - Add sub-agent spawning + Markdown agent defs.
   - Build structured vault + selection layer.
   - Add verification node + 2-3 workflow patterns.
   - Expand evals + run comparative tests.

3. **Week 4+ (Phase 3 Parallel)**:
   - Implement ForgeMaster meta-agent + sandbox testing.
   - Add scheduler for autonomous loops.
   - Start distilling skills from real usage.
   - Enable self-improvement cycles with gates.

**Folder Structure (Starter)**:
```
forge/
├── agents/              # Sub-agent Markdown defs + parsed runners
├── skills/              # Registered skills (Python + YAML)
├── tools/               # Core + domain tools (sandbox wrappers)
├── memory/              # Vault, selectors, state files, SQLite helpers
├── graphs/              # LangGraph definitions (orchestrator, sub-graphs)
├── evals/               # Test tasks, metrics, runners, reports
├── config/              # YAML: models, permissions, paths, budgets
├── harness/             # Core classes: Orchestrator, ContextAssembler, Verifier, etc.
├── cli.py               # Main entrypoint
├── utils/               # Logging, sandbox, persistence helpers
└── examples/            # Design project, trading backtest, content workflow
```

**Next Immediate Actions for You**:
- Install Ollama + pull 1-2 test models.
- `pip install langgraph langchain langchain-community pyyaml pydantic loguru` (and others as needed).
- Clone or create the `forge/` structure.
- Start with Phase 1 graph in `graphs/orchestrator.py`.

I can generate the full starter code scaffolding (key files with comments, runnable minimal version) in `/home/workdir/artifacts/forge_blueprint/` or directly in a new repo if you prefer. Just say the word.

---

## Tailoring to Your Context (from known preferences)

- **Design Viz Workflows**: Skills for "load 2D layout/PDF/CAD → propose accurate 3D/AXO in ComfyUI/Blender → verify scale/context/adherence → generate tiered options (subpar/exotic) with dopamine-inducing prompts". Sub-agents for different reviewer personas (client happy-place, technical feasibility, material realism).
- **Trading Systems**: Skills for "implement SMC order blocks / CML indicator in Pine/Python → backtest on local data → verify with metrics → distill rules into memory". Heavy use of code exec sandbox + verification.
- **Business Automation**: Content/SEO agents that research (local vault), draft (local LLM), verify against brand voice/rubrics, output to files for manual review/publish. Multi-engine domination playbooks as skills.
- **Privacy/Local**: Everything file-based or local DB. No external calls unless you explicitly add (and gate) them. Workspace per project.
- **Incremental + Scalable**: Start with one-room/test project. Framework designed so skills/workflows transfer and compound across projects.
- **No Video/Audio**: Skipped entirely. Focus on text/code/image-ref (for design) + structured outputs.

This framework will mature with you. Phase 1 gives immediate value. Phase 3 makes it your co-pilot that gets better every week.

**Questions to Refine (max 5, only if helpful)**:
1. Which domain should we prioritize first for concrete skill examples in the starter scaffolding (architectural interiors/design viz, trading indicators/backtesting, or business content/automation)?
2. Any strong preference on memory implementation details (pure Obsidian-style Markdown vault with frontmatter + BM25, or add local embeddings/LanceDB from day one)?
3. Current LLM serving setup or preferred models (Ollama ready? Specific quantized models you like on Mac)?
4. For code execution sandbox: How strict initially (RestrictedPython in-process vs subprocess with limits vs Docker jail)? Any must-allow libraries for your trading/design work?
5. Human gate strictness for Phase 1-2 (e.g., all writes require confirmation, or only outside-workspace / high-impact)?

Answer any/all and I will generate the production-quality starter code, detailed skill examples for your priority domain, or refine any section. Ready to build the first runnable harness when you are.

This is the blueprint. Let's make it real. 🚀

*References synthesized from the X threads you shared (Akshay on harness components, Jey on sub-agent fleets & clean contexts, Khairallah on context selection vs rot, de1lymoon on loop designers + independent graders, unicodef1wn on dynamic workflows + adversarial verification) plus enterprise agent patterns. All adapted strictly for offline local Python + LangGraph.*