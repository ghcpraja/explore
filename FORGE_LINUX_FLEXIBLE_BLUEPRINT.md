# FORGE v2: Linux-Native, Modular, Future-Proof Agent Harness & Multi-Agent Orchestration Framework
## Modern, State-of-the-Art, Flexible & Scalable — Basics First, Upgrade Paths Built-In
**Version:** 2.0 (Linux Edition) | **Date:** 2026-06-28 | **Environment:** Linux (Ubuntu/Debian recommended) | **Core:** LangGraph + Pure Python (Clean Architecture) | **Entry:** CLI (Frontend-Ready)

**Design Goals (Directly Addressing Your Requirements)**
- **Linux-first, no assumptions**: Works on minimal Linux (fresh Ubuntu server or desktop). No Docker required initially. No vector DB required initially. No GPU required initially (CPU inference works, GPU accelerates).
- **Basics first + explicit upgrade paths**: Every major component has a "Basic" implementation + clear, documented steps to upgrade (e.g., swap memory backend, add Docker sandbox, switch to vLLM, add Postgres, enable frontend API).
- **Extreme flexibility**: Plugin architecture via Python protocols/interfaces + config-driven factories. Swap models, tools, memory, sandbox, persistence, even orchestration style without rewriting core logic.
- **Frontend-ready from day one**: CLI is thin. Core logic lives in a service layer with clean APIs. Easy migration to FastAPI backend + WebSocket/SSE streaming + future React/Next.js or Gradio/Streamlit frontend. No tight coupling to CLI.
- **Multi-agent & sub-agent orchestration (state-of-the-art)**: Full support for supervisor patterns, dynamic sub-agent spawning, handoff, parallel execution, clean isolated contexts per sub-agent, model routing, shared or namespaced memory. Covers modern patterns from LangGraph best practices, Anthropic/OpenAI agent research, and 2026 workflows (adversarial verification, fan-out/synthesize, tournaments, reflection loops).
- **No lapses — comprehensive & modern**: Context engineering, structured memory with selection, verification/guardrails, error recovery, observability, human-in-the-loop, self-improvement hooks, security/sandboxing, persistence/replay, evaluation harness. Scalable to server deployments (multiple workers, task queues later).
- **Offline/local by default**: Local models (Ollama recommended start). Everything file-based or local DB where possible. Privacy-first.
- **Phased with concrete self-testing**: Same 3-phase maturity but now modular so you can adopt pieces independently.
- **Your use cases covered**: Architectural design (ComfyUI/Blender local integration), trading (Python/Pine backtesting + SMC/CML), business automation, general agentic work. Extensible skills system.

This is the **production blueprint** a capable agent (or team) can implement directly. It is designed so the system can eventually help maintain and extend itself.

---

## High-Level Architecture (Flexible & Layered)

```
┌─────────────────────────────────────────────────────────────┐
│                      USER / FRONTEND                        │
│   (CLI today  →  FastAPI + WS/SSE  →  Web UI tomorrow)     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    SERVICE LAYER (FastAPI-ready)            │
│  OrchestratorService, AgentRunner, SessionManager,          │
│  SkillRegistry, ToolRegistry, MemoryService                 │
│  (Dependency Injection + Config Factories)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│              CORE DOMAIN (Pure Python, Interfaces)          │
│  • Agent / SubAgent (protocols)                             │
│  • Skill (callable with schema)                             │
│  • Tool (execute with permission check)                     │
│  • MemoryBackend (store/retrieve/select/compact)            │
│  • ModelClient (invoke with tools/structured output)        │
│  • SandboxExecutor (run code/file ops safely)               │
│  • GraphOrchestrator (LangGraph wrapper)                    │
│  • Verifier, Guardrails, ContextAssembler                   │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  ADAPTERS     │   │  ADAPTERS     │   │  ADAPTERS     │
│  (Swappable)  │   │  (Swappable)  │   │  (Swappable)  │
│               │   │               │   │               │
│ Model:        │   │ Memory:       │   │ Sandbox:      │
│ - Ollama      │   │ - FileJSON    │   │ - Subprocess  │
│ - vLLM        │   │ - SQLite      │   │ - Docker      │
│ - llama.cpp   │   │ - Chroma      │   │ - Firejail    │
│ - HF          │   │ - Postgres    │   │ (upgrade)     │
│               │   │ (upgrade)     │   │               │
│ Tool Exec     │   │ Persistence:  │   │ Observability │
│ etc.          │   │ - File        │   │ - JSONL logs  │
│               │   │ - SQLite      │   │ - OpenTelemetry│
│               │   │ - Postgres    │   │ (upgrade)     │
└───────────────┘   └───────────────┘   └───────────────┘
```

**Key Modularity Features**
- Every major piece is behind a **Protocol** (Python `typing.Protocol`) or ABC.
- **Factory functions** or simple registry + config pick the implementation at runtime.
- **Dependency injection** (simple class or `fastapi.Depends` later) wires everything.
- Config is king: `config.yaml` or Pydantic `Settings` with environment overrides. Profiles: `basic`, `dev`, `prod`, `gpu`, `cpu-only`.
- Adding a new tool, memory backend, or model client = implement the protocol + register in config/factory. No core changes.

---

## Linux Tech Stack — Basics First + Upgrade Paths

**Recommended Base (Minimal Linux, e.g., Ubuntu 24.04 server/desktop)**
- Python 3.11+ (use `uv` or `pip` + venv; `uv` strongly recommended for speed).
- **Models**: Ollama (easiest, excellent Linux support, CPU or GPU). Start here.
- **Orchestration**: LangGraph + LangChain (local components only).
- **Memory/Persistence**: Pure file-based (Markdown vault + JSON/JSONL) + SQLite (built-in, zero deps).
- **Sandbox**: Python `subprocess` with strict limits (cwd, timeout, env whitelist, no network by default).
- **CLI**: Typer (modern, beautiful, auto-completion).
- **Config**: Pydantic v2 `BaseSettings` + YAML.
- **Logging/Tracing**: `loguru` + structured JSONL (local viewer script later).
- **Other basics**: `pathlib`, `tenacity` (retries), `pydantic`, `pyyaml`, `rich` (pretty CLI).

**Explicit Upgrade Paths (Documented in Code & README)**
- **Models**:
  - Basic: Ollama HTTP client (simple `requests` or `httpx`).
  - Upgrade 1: `langchain-ollama` or `langchain-community` ChatOllama (tool calling, structured output).
  - Upgrade 2: vLLM (OpenAI-compatible server, much higher throughput for multiple agents; `vllm` package + server).
  - Upgrade 3: llama.cpp server or Hugging Face `transformers` + `bitsandbytes` (CPU/GPU offload).
  - Later: Any OpenAI-compatible local endpoint.
- **Memory Backends**:
  - Basic: `FileMemoryBackend` (Markdown files + simple keyword/BM25 search via `rank-bm25` or pure Python).
  - Upgrade 1: `SQLiteMemoryBackend` (structured tables + FTS5 full-text search).
  - Upgrade 2: `ChromaMemoryBackend` or `LanceDBMemoryBackend` (local vector + metadata filtering + hybrid search).
  - Upgrade 3: PostgreSQL (with pgvector or JSONB) for team/shared memory or larger scale.
  - Selection layer always on top (protocol method `select_for_context(goal, candidates, state)`).
- **Sandbox / Tool Execution**:
  - Basic: `SubprocessSandbox` (Linux-friendly: `subprocess.run` with `cwd`, `timeout`, `env`, `preexec_fn` for limits, seccomp ideas later).
  - Upgrade 1: Firejail or bubblewrap for stronger isolation (common on Linux desktops/servers).
  - Upgrade 2: Docker-backed executor (resource limits, network none, user mapping). Docker is easy on Linux.
  - Upgrade 3: gVisor or Kata Containers for very high security (server use).
- **Persistence / Checkpoints**:
  - Basic: LangGraph `SqliteSaver` (file-based SQLite) or custom JSONL + file checkpointer.
  - Upgrade: PostgreSQL checkpointer (LangGraph supports it) or Redis for high-scale.
- **Orchestration Scaling**:
  - Basic: Single process, asyncio for parallel sub-agents (limited by CPU/RAM).
  - Upgrade: Celery/RQ + Redis broker for background/long-running agent jobs (Linux server classic).
  - Later: Kubernetes if you go very large (but overkill for most personal/ small-team use).
- **Frontend / API**:
  - Basic: Pure CLI (Typer) calling core services directly.
  - Upgrade 1 (recommended path): Add FastAPI layer. Expose `/sessions`, `/run`, `/resume`, streaming endpoints. CLI becomes thin HTTP client (`httpx`).
  - Upgrade 2: Add WebSocket or Server-Sent Events for real-time streaming of thoughts/tool calls.
  - Upgrade 3: Full frontend (Gradio/Streamlit first for speed, then custom React/Next.js or Tauri desktop app). All state via API.
- **Observability**:
  - Basic: Rich console + JSONL log files + simple CLI `forge logs` viewer.
  - Upgrade: OpenTelemetry (local collector) or Langfuse/Phoenix self-hosted (but keep optional; local JSONL is sufficient for most).

**Installation on Linux (Basic)**
```bash
# Ubuntu/Debian example
sudo apt update && sudo apt install -y python3.11 python3.11-venv python3-pip git curl
curl -fsSL https://ollama.com/install.sh | sh   # Ollama
ollama serve &   # or systemctl if packaged
ollama pull llama3.1:8b   # start small; upgrade model later

# Project
git clone ... or mkdir forge && cd forge
python3.11 -m venv .venv && source .venv/bin/activate
pip install uv && uv pip install "langgraph>=0.2" "langchain>=0.3" typer pyyaml pydantic loguru rich tenacity
# or use uv sync with pyproject.toml (recommended for prod)
```

---

## Core Interfaces (Protocols) — The Flexibility Contract

All major components implement these (or similar). This is what makes swapping trivial.

```python
from typing import Protocol, Any, Optional, List, Dict
from pydantic import BaseModel

class ModelClient(Protocol):
    def invoke(self, prompt: str, tools: Optional[List[dict]] = None, 
               structured_output: Optional[type[BaseModel]] = None, 
               **kwargs) -> Any: ...
    async def ainvoke(self, ...): ...

class MemoryBackend(Protocol):
    def store(self, key: str, value: Any, metadata: dict): ...
    def retrieve(self, query: str, top_k: int = 5, filters: dict = None) -> List[dict]: ...
    def select_for_context(self, goal: str, candidates: List[dict], current_state: dict) -> List[dict]: ...
    def compact(self, session_id: str): ...

class SandboxExecutor(Protocol):
    def execute(self, command: str, cwd: str, timeout: int = 30, 
                allow_network: bool = False, env: dict = None) -> dict: ...
    def write_file(self, path: str, content: str, safe: bool = True): ...
    def read_file(self, path: str) -> str: ...

class Tool(Protocol):
    name: str
    description: str
    parameters: dict  # JSON schema
    def __call__(self, **kwargs) -> Any: ...

class Skill(Protocol):
    name: str
    def execute(self, goal: str, context: dict, tools: List[Tool]) -> Any: ...
```

Factories read config and return the right implementation. Example config snippet:

```yaml
# config/basic.yaml
model:
  provider: "ollama"
  model_name: "llama3.1:8b"
  base_url: "http://localhost:11434"

memory:
  backend: "file"          # options: file, sqlite, chroma, postgres
  vault_path: "./memory/vault"
  db_path: "./memory/forge.db"

sandbox:
  type: "subprocess"       # options: subprocess, docker, firejail

persistence:
  checkpointer: "sqlite"   # file, sqlite, postgres

orchestration:
  max_parallel_subagents: 4
  default_model_for_subagents: "qwen2.5:7b"
```

Adding a new backend = implement the protocol + add `elif` in factory (or use entry points / plugin system later).

---

## Multi-Agent & Sub-Agent Orchestration (State-of-the-Art, No Lapses)

**Patterns Supported Out of the Box**
1. **Supervisor / Router Pattern** (recommended start): One main graph with a router node (LLM decides) that hands off to specialist sub-graphs (or spawns them). LangGraph has excellent support via `add_conditional_edges` and compiled sub-graphs.
2. **Dynamic Sub-Agent Spawning**: Sub-agents defined in `agents/*.md` (YAML frontmatter + prompt) or as Python classes. Orchestrator can create isolated `StateGraph` instances with fresh `messages` history, limited tools (via permission filter), and specific model.
3. **Clean Context Isolation**: Each sub-agent gets its own conversation state + memory namespace (LangGraph checkpointer supports namespaces). Only summary/hand-off message returns to parent.
4. **Parallel Execution**: `asyncio.gather` or LangGraph's parallel node support for fan-out. Resource-aware (config limit on concurrent heavy models).
5. **Handoff & Resume**: Explicit handoff nodes. Sub-agent can return control with structured output (Pydantic model).
6. **Advanced Workflow Patterns**:
   - Fan-out → synthesize (map-reduce style).
   - Adversarial verification (maker sub-agent + independent checker sub-agent with rubric).
   - Tournament / generate-and-filter.
   - Reflection / self-critique loops.
   - Plan-and-Execute (separate planning graph then execution graph).
7. **Model Routing**: Orchestrator on strong model; heavy reading/research on lighter/faster models; verification on another. Config per sub-agent type.
8. **Shared vs Namespaced Memory**: Global knowledge in one namespace; per-project or per-session in others. Selector respects namespaces.

**Implementation Sketch (LangGraph)**
- Main `orchestrator_graph = StateGraph(ForgeState)`
- Nodes: `think`, `route_or_delegate`, `execute_tool`, `verify`, `update_memory`, `finalize`
- Conditional edges based on LLM output (tool call? delegate to "design_reviewer"? done?).
- Sub-graphs registered and invoked via `graph.invoke(..., subgraph="design_reviewer")` or dynamic compilation.
- For very dynamic: Use a "spawn_subagent" tool that the model can call; it creates a new runner with isolated state.

This covers modern 2026 agent research (separation of concerns, clean contexts, verification, economics of model sizing) while remaining fully local and controllable.

---

## 3-Phase Maturity Plan (Same Goals, More Modular)

**Phase 1: Core Single-Agent + CLI (Basics — 1-2 weeks)**
- Implement protocols + basic adapters (Ollama client, FileMemory + simple search, SubprocessSandbox, SQLite checkpointer).
- Single LangGraph with ReAct-style loop + basic context assembler + compaction.
- Core tools: file read/write/list (sandboxed), code exec (restricted), memory search.
- Skill registry (simple decorator or YAML-loaded).
- Typer CLI: `forge run "goal here" --project my-design-project`
- Basic eval harness (Python functions or small JSON tasks) with success rate, trace logging.
- Self-test: Run on 10-15 tasks; measure success, tokens, drift (simple contradiction detection).
- **Upgrade notes in every file**: "To switch to vLLM: change config + implement VLLMModelClient (OpenAI-compatible)."

**Phase 2: Multi-Agent, Structured Memory, Verification**
- Add sub-agent spawning + Markdown agent definitions + isolated graph execution.
- Implement `select_for_context` in memory (critical for avoiding rot).
- Add verification node + independent checker sub-agent pattern.
- Expand sandbox options (document Docker path).
- Add workflow patterns (fan-out, adversarial).
- Enhanced evals: long-horizon, multi-agent workflows, context pollution tests.
- **Frontend readiness**: Extract service layer. Add optional FastAPI skeleton (even if not used yet). CLI can stay direct or switch to HTTP mode via flag.

**Phase 3: Self-Improving + Autonomous + Scale**
- ForgeMaster meta-agent (inspects logs, proposes skill/graph/prompt improvements, sandbox-tests them).
- Skill distillation from traces + verified memory.
- Autonomous loops via APScheduler or simple cron + goal + verifier.
- Full API layer (FastAPI) with session management, streaming.
- Optional task queue (Celery) for background runs.
- Advanced memory (Chroma/LanceDB or Postgres) + better selection.
- Self-eval + A/B testing of improvements.
- **Linux server deployment**: systemd service, multiple workers, resource monitoring.

Each phase has concrete pass criteria and automated eval scripts.

---

## Folder Structure (Clean, Extensible, Frontend-Ready)

```
forge/
├── core/                    # Pure domain logic & protocols
│   ├── protocols.py
│   ├── models.py            # Pydantic state, schemas
│   ├── agent.py
│   ├── skill.py
│   ├── tool.py
│   └── context.py
├── adapters/                # All swappable implementations
│   ├── models/
│   │   ├── ollama.py
│   │   ├── vllm.py          # placeholder + upgrade guide
│   │   └── factory.py
│   ├── memory/
│   │   ├── file_backend.py
│   │   ├── sqlite_backend.py
│   │   ├── chroma_backend.py # upgrade
│   │   └── factory.py
│   ├── sandbox/
│   │   ├── subprocess_exec.py
│   │   ├── docker_exec.py   # upgrade
│   │   └── factory.py
│   └── persistence/
├── graphs/                  # LangGraph definitions
│   ├── orchestrator.py
│   ├── sub_agent.py
│   ├── workflows.py         # fan-out, verification patterns
│   └── factory.py
├── services/                # Service layer (API-ready)
│   ├── orchestrator_service.py
│   ├── session_manager.py
│   └── registry.py
├── cli/                     # Typer commands
│   ├── main.py
│   ├── run.py
│   └── utils.py
├── api/                     # FastAPI skeleton (commented or optional)
│   ├── main.py
│   ├── routers/
│   └── dependencies.py
├── memory/                  # Runtime vault + db (gitignored or configurable)
│   └── vault/
├── skills/                  # Your custom skills (Python + YAML)
├── agents/                  # Sub-agent Markdown definitions
├── tools/                   # Custom tool implementations
├── evals/                   # Test tasks, runners, metrics
├── config/                  # YAML profiles + Pydantic settings
├── utils/                   # logging, sandbox helpers, tracing
├── tests/                   # pytest + eval integration
├── pyproject.toml
└── README.md                # Full setup, upgrade guides, examples
```

**Key Files to Implement First (Phase 1)**
- `core/protocols.py`
- `adapters/models/ollama.py` + factory
- `adapters/memory/file_backend.py` + factory
- `adapters/sandbox/subprocess_exec.py`
- `graphs/orchestrator.py` (basic single agent)
- `services/orchestrator_service.py`
- `cli/main.py`

---

## Frontend & API Upgrade Path (Explicit Directions)

1. Keep all heavy logic in `services/` and `core/`.
2. CLI calls services directly (current).
3. Add `api/main.py` (FastAPI):
   - Endpoints: `POST /sessions`, `POST /sessions/{id}/run`, `GET /sessions/{id}/status`, `POST /sessions/{id}/resume`, streaming via `StreamingResponse` or WebSocket.
   - Use same `OrchestratorService`.
4. CLI can grow a `--api-mode` flag that talks HTTP to local server (or always run server in background).
5. Later frontend (Gradio first for 1-day prototype, then full SPA) consumes the API. All state (goals, traces, memory) lives server-side or in shared DB.
6. Session management: Each run gets a persistent session ID with LangGraph checkpointer → resume, branch, inspect history.

This structure means when you add the frontend, you change almost nothing in the agent logic.

---

## Security, Observability, Guardrails (No Lapses)

- **Sandbox**: Always run tools in restricted environment. Basic subprocess already limits a lot on Linux. Document exact commands for Docker upgrade.
- **Permissions**: Per-tool / per-skill matrix in config or memory. High-risk actions (write outside workspace, network, delete) require explicit confirmation node or human interrupt.
- **Guardrails**: Input/output filtering (simple + optional LLM judge). Tool schema validation.
- **Observability**: Every LLM call, tool execution, decision logged with tokens, latency, session_id, sub-agent_id. Rich console during CLI run + persistent JSONL.
- **Human-in-the-Loop**: LangGraph `interrupt` or custom nodes at verification, high-risk tool, or final answer stages.
- **Error Recovery**: Tenacity retries, LLM self-correction via error observations, fallback models.
- **Audit**: Full event log + ability to replay any session via checkpointer.

---

## Self-Testing & Continuous Improvement

Same strong eval harness as v1, now even easier to extend because of modularity.
- `evals/runner.py` runs tasks, compares against expected (or LLM judge rubric), logs metrics.
- Metrics tracked over time (success rate, efficiency, drift).
- Phase 3 adds meta-eval of the harness itself.

---

## Next Immediate Steps for You on Linux

1. Set up minimal Linux env + Ollama + small model.
2. Create the folder structure above.
3. Implement the protocols + basic Ollama + file memory + subprocess sandbox.
4. Build the first single-agent LangGraph.
5. Add Typer CLI.
6. Run first eval tasks.
7. Iterate to Phase 1 complete.

I have created this complete revised blueprint as a file:

**File saved to:** `/home/workdir/artifacts/forge_blueprint/FORGE_LINUX_FLEXIBLE_BLUEPRINT.md`

It contains even more detail, code skeletons for interfaces/factories, exact upgrade instructions per component, multi-agent implementation guidance, frontend migration steps, and Linux-specific commands.

**Would you like me to now generate the actual starter code scaffolding** (pyproject.toml, key protocol files, basic Ollama adapter, file memory backend, subprocess sandbox, minimal LangGraph orchestrator, Typer CLI skeleton, and example config + one sample skill for e.g. design or trading)? 

This will give you a runnable (even if minimal) Phase 1 foundation you can immediately test and extend.

Just confirm the priority domain for the first example skill, and say "build the starter code" — I'll write all the files into the artifacts or a clean `forge-starter/` directory.

This version is fully flexible, Linux-native, basics-first with clear upgrades, multi-agent capable from Phase 2, and frontend-ready without any wasted work. No assumptions, no lapses, state-of-the-art patterns throughout.

Ready when you are.