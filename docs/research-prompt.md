# local deep research prompt

## Prompt A — Bootstrap & File Tree

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Task:
1. Create ANALYSIS_MD with a title and a “Scope & Evidence Policy” section (copy the Shared header content).
2. Emit a flat file tree of @src/ollama_deep_research/ with rough LOC per file.
3. List *external dependency* references found in-scope files (symbols imported from outside @src/ollama_deep_research/).

Deliver to ANALYSIS_MD:
- Section: “File Tree”
- Section: “External Dependencies”

---

## Prompt B — State Model Audit (State First)

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Inventory graph state.

Tasks:
1. Enumerate state models/TypedDicts/Pydantic models.
2. For each state key: type, default, readers, writers.
3. Note reducers/mergers and where invoked.
4. Capture iteration/token/concurrency controls if stored in state.

Deliver to ANALYSIS_MD (section: State):
- Table: state_key | type | default | readers | writers | evidence
- Bullets for reducers/mergers with evidence.

---

## Prompt C — Node Inventory (Nodes Next)

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Contracts per runnable/node.

Tasks (for each node):
- Reads (state keys), Writes (state keys)
- Tools used (names)
- Retry/timeout controls
- Exit conditions / return signals
- Message/notes evolution

Deliver to ANALYSIS_MD (section: Nodes):
- Table: node | reads | writes | tools | exit_condition | evidence
- Bullets: message/state transformations with evidence.

---

## Prompt D — Tools & MCP Wiring

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Map builtin/search/MCP tools and selection flow.

Tasks:
1. Builtins (incl. search): how loaded; wrappers (auth/error/metrics).
2. MCP client(s): class, transport (stdio/http/stream), config surface (file:line).
3. Name-collision logic; allow/deny lists; dynamic vs static discovery.
4. Env/RunnableConfig keys that affect tool wiring.

Deliver to ANALYSIS_MD (section: Tools & MCP):
- Table: area | symbol | file | how loaded | filters/collisions | evidence
- Bullet list of config/env keys with evidence.

---

## Prompt E — Prompt Surfaces + Limits/Observability

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Prompts + controls in one pass.

Tasks:
1. System/user prompts that govern supervisor/researcher; note tool affordances.
2. Token budgets & recovery (compression/summarization windows).
3. Observability: logging/tracing/metrics calls and where configured.
4. Limits: iteration ceilings, retries, timeouts; override surfaces (env/config).

Deliver to ANALYSIS_MD:
- Section: Prompt Surfaces (table: module | prompt | role | tool refs | evidence)
- Section: Observability & Limits (table: control | default | override surface | owner | evidence)

---

## Prompt F — Graph Topology

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Concrete topology and state I/O.

Tasks:
1. Adjacency list: NodeA -> NodeB [condition=…] with evidence per edge.
2. Mermaid diagram (high-level flow).
3. State I/O matrix (state rows × node columns; R/W marks).

Deliver to ANALYSIS_MD (section: Graph Model):
- Adjacency list (with evidence)
- mermaid fenced diagram
- Matrix table with evidence notes.

---

## Prompt G — Capabilities Matrix + Executive Summary

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: What exists, where to extend, and headline insights.

Tasks:
1. Capability & Integration Matrix:
   Columns: Area (LLM, search, tools, MCP, memory/state, prompts, observability, limits),
   What exists (path + 1-liner), Integration seam (no-surgery approach), Risks/constraints, Test hook.
2. Executive Summary: 8–12 bullets (abilities, seams, configurability, risks, fastest MCP add path).
   Each bullet cites at least one evidence line.

Deliver to ANALYSIS_MD:
- Section: Capability & Integration Matrix (markdown table)
- Section: Executive Summary (bullets with evidence)

---

## Prompt H — Graph Manifest (JSON, local-only)

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Emit a strict manifest without repo/branch details.

Schema (use only local path + time):
{
  "workspace": {"path": "@src/ollama_deep_research/", "analyzed_at": "<UTC ISO>"},
  "graph": {
    "state_keys": [...],
    "nodes": [...],         // name, reads[], writes[], file
    "edges": [...],         // source, target, condition
    "loops": [...]          // name, exit_condition
  },
  "tools": {
    "builtin": [...],       // name, file
    "search": [...],        // name, file
    "mcp": [...]            // client, transport, tools (if resolvable), file
  },
  "integration_points": [
    {"surface": "tool_loading", "file": "<relative_path>", "pattern": "get_all_tools()/load_mcp_tools()", "best_practice": "config-driven injection"}
  ]
}

Task:
- Populate MANIFEST_JSON with concrete entries and file paths (no comments).
- In the chat reply, add a short “Evidence Notes” list mapping key manifest sections to evidence cites.

---

## Prompt I — MCP Rapid-Prototype Plan (YAML)

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Minimal changes to introduce one MCP tool (e.g., classify_text), respecting existing wiring.

Template (fill with discovered surfaces):
prototype:
  scope_rule: "Only modify files under @src/ollama_deep_research/; prefer config-based injection."
  goal: "Expose one new MCP tool and wire it into researcher flow without graph surgery."
  server:
    name: demo_tools
    transport: <stdio|http|stream>
    auth: <optional|bearer>
    endpoint: "<if http>"
    tools:
      - name: classify_text
        input_schema: {type: object, properties: {text: {type: string}}, required: [text]}
        output_schema: {type: object, properties: {label: {type: string}}}
  client_wiring:
    config_surface: "<file:line where MCP config is read>"
    discovery: "<client path/class>"
    selection: "<how to avoid name collisions>"
  prompts:
    edits:
      - file: "<prompt module>"
        change: "Advertise availability/scope of classify_text."
  acceptance:
    checks:
      - "Client discovers MCP tool at startup"
      - "Researcher invokes tool on suitable queries"
      - "State updates reflect tool outputs"
  rollout:
    dev: "local stdio/http"
    prod: "auth header via token store; retry/backoff"

Deliver MANIFEST_YAML to MCP_PLAN_YAML, then list evidence cites in the reply.

---

## Prompt J — Minimal Diffs + Runbook + TODOs

Role: Senior AI engineer (LangGraph/LangChain + MCP).

Scope: ONLY read files under @src/ollama_deep_research/

Output files:
- ANALYSIS_MD: docs/ldr-analysis-{{DATE}}.md
- MANIFEST_JSON: docs/ldr-graph-manifest-{{DATE}}.json
- MCP_PLAN_YAML: docs/ldr-mcp-prototype-{{DATE}}.yaml

Constraints:
- Deterministic (low temperature).
- Evidence discipline: cite as <relative_path>:<start>-<end>.
- Don’t edit code; propose unified diffs in fenced blocks.
- Prefer tables/manifests over prose.

Goal: Hand-off bundle.

Tasks:
1. Minimal diffs (unified): config injection, tool load path (collision-safe), prompt affordance.
2. Runbook:
   - Env/config to set
   - Commands to run the graph end-to-end
   - How to confirm MCP tool is discovered/invoked; which state keys/messages carry outputs
3. TODOs/Questions: ambiguities and *external dependency* follow-ups.

Deliver to ANALYSIS_MD:
- Section: Minimal Diffs (diff fenced)
- Section: Acceptance & Runbook (checklist + commands)
- Section: TODOs / Questions (bullets with evidence references)
