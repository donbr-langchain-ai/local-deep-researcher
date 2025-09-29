# Local Deep Researcher - LangGraph Architecture Analysis

**Analysis Date:** 2025-09-29T04:13:39Z
**Workspace:** `src/ollama_deep_researcher/`

## Executive Summary

Local Deep Researcher implements an iterative web research workflow using LangGraph. The system employs a state-driven architecture with 5 core nodes orchestrating query generation, web search, summarization, reflection, and finalization phases. The design supports multiple LLM providers (Ollama, LMStudio) and search APIs (DuckDuckGo, Tavily, Perplexity, SearXNG) with configurable structured output modes.

## State Architecture

| Field | Type | Purpose | Evidence |
|-------|------|---------|----------|
| `research_topic` | str | User's research subject | <src/ollama_deep_researcher/state.py:8> |
| `search_query` | str | Generated web search query | <src/ollama_deep_researcher/state.py:9> |
| `web_research_results` | list | Accumulated search results | <src/ollama_deep_researcher/state.py:10> |
| `sources_gathered` | list | Formatted citation sources | <src/ollama_deep_researcher/state.py:11> |
| `research_loop_count` | int | Iteration counter for loop control | <src/ollama_deep_researcher/state.py:12> |
| `running_summary` | str | Evolving research synthesis | <src/ollama_deep_researcher/state.py:13> |

## Graph Flow Analysis

### Node Sequence
1. **generate_query** → **web_research** → **summarize_sources** → **reflect_on_summary**
2. **Conditional Loop**: Back to **web_research** if `research_loop_count <= max_web_research_loops`
3. **Finalization**: **finalize_summary** → **END**

### Critical Control Points

| Node | Reads | Writes | Decision Logic |
|------|-------|--------|----------------|
| `generate_query` | research_topic | search_query | LLM-driven query optimization |
| `web_research` | search_query, research_loop_count | sources_gathered, web_research_results | API-specific search routing |
| `summarize_sources` | research_topic, running_summary, web_research_results | running_summary | Incremental summary evolution |
| `reflect_on_summary` | research_topic, running_summary | search_query | Knowledge gap identification |
| `finalize_summary` | running_summary, sources_gathered | running_summary | Source deduplication & formatting |

Evidence: <src/ollama_deep_researcher/graph.py:444-466>

## Tool Ecosystem

### Built-in Tools
- **Query**: Structured search query generation with rationale (<src/ollama_deep_researcher/graph.py:161-171>)
- **FollowUpQuery**: Knowledge gap analysis and follow-up query creation (<src/ollama_deep_researcher/graph.py:352-364>)

### Search Provider Integration
| Provider | Function | Key Features |
|----------|----------|--------------|
| DuckDuckGo | `duckduckgo_search()` | No API key, max_results configurable |
| Tavily | `tavily_search()` | Raw content support, API key required |
| Perplexity | `perplexity_search()` | Citation-based results, sonar-pro model |
| SearXNG | `searxng_search()` | Self-hosted search, configurable endpoint |

Evidence: <src/ollama_deep_researcher/utils.py:166-387>

## LLM Provider Abstraction

### Dual Provider Support
- **Ollama**: Direct integration via `ChatOllama` (<src/ollama_deep_researcher/graph.py:122-135>)
- **LMStudio**: OpenAI-compatible wrapper with JSON response cleanup (<src/ollama_deep_researcher/lmstudio.py:19-98>)

### Structured Output Handling
- **Tool Calling Mode**: Preferred for newer models supporting function calls
- **JSON Mode Fallback**: Legacy support with response format enforcement
- **Adaptive Selection**: `use_tool_calling` config flag controls behavior

Evidence: <src/ollama_deep_researcher/graph.py:44-96>

## Configuration Management

### Hierarchy Priority
1. **Environment Variables** (highest)
2. **LangGraph UI Configuration**
3. **Default Values** (lowest)

### Key Configuration Points
| Setting | Default | Impact |
|---------|---------|--------|
| `max_web_research_loops` | 3 | Research depth control |
| `use_tool_calling` | false | Structured output method |
| `fetch_full_page` | true | Content depth vs. speed tradeoff |
| `strip_thinking_tokens` | true | Response cleanup for reasoning models |

Evidence: <src/ollama_deep_researcher/configuration.py:16-81>

## MCP Integration Readiness

### Current State
- **No MCP implementations found** in codebase analysis
- **Integration surfaces identified** for future MCP adoption

### Proposed Integration Points

| Surface | Current Pattern | MCP Enhancement Opportunity |
|---------|-----------------|----------------------------|
| **LLM Provider Selection** | Hard-coded if/else logic | MCP-driven provider registry |
| **Search API Routing** | Manual conditional blocks | MCP tool discovery & routing |
| **Configuration Management** | Environment + Pydantic | MCP configuration server |
| **Structured Output** | Dual-mode implementation | MCP capability negotiation |

## Architectural Strengths

1. **Clean State Separation**: Dataclass-based state with clear field purposes
2. **Provider Agnostic**: Abstract LLM and search provider interfaces
3. **Configurable Depth**: Research loop control prevents infinite iteration
4. **Error Resilience**: Fallback mechanisms for JSON parsing and API failures
5. **Source Deduplication**: Prevents citation redundancy in final reports

## Technical Debt & Improvement Areas

1. **Search Provider Coupling**: Hard-coded conditional logic in `web_research()` node
2. **Tool Definition Duplication**: Similar Pydantic models for Query/FollowUpQuery tools
3. **Response Cleaning Logic**: LMStudio-specific JSON cleanup could be generalized
4. **Configuration Sprawl**: 12 configuration fields with complex interaction patterns

## Recommendations for MCP Migration

1. **Tool Registry Pattern**: Replace search API conditionals with MCP tool discovery
2. **Provider Plugin System**: Abstract LLM provider selection behind MCP interface
3. **Configuration Service**: Centralize configuration management via MCP server
4. **Capability Negotiation**: Use MCP to detect LLM structured output capabilities

---

# MCP Integration Hand-off Bundle

## Minimal Diffs

### 1. Configuration Extension

```diff
--- a/src/ollama_deep_researcher/configuration.py
+++ b/src/ollama_deep_researcher/configuration.py
@@ -60,6 +60,16 @@ class Configuration(BaseModel):
         description="Use tool calling instead of JSON mode for structured output",
     )

+    mcp_enabled: bool = Field(
+        default=False,
+        title="Enable MCP Tools",
+        description="Enable Model Context Protocol tool discovery"
+    )
+    mcp_server_url: str = Field(
+        default="stdio://demo_tools",
+        title="MCP Server URL",
+        description="MCP server connection string (stdio:// or http://)"
+    )
+
     @classmethod
     def from_runnable_config(
         cls, config: Optional[RunnableConfig] = None
```

### 2. Tool Discovery Function

```diff
--- a/src/ollama_deep_researcher/utils.py
+++ b/src/ollama_deep_researcher/utils.py
@@ -387,3 +387,20 @@ def perplexity_search(
         )

     return {"results": results}
+
+
+def get_mcp_tools(mcp_server_url: str) -> List[Any]:
+    """Discover and return MCP tools from configured server.
+
+    Args:
+        mcp_server_url: Connection string (stdio://demo_tools or http://localhost:3000)
+
+    Returns:
+        List of LangChain-compatible tool objects
+    """
+    if not mcp_server_url or mcp_server_url == "disabled":
+        return []
+
+    # TODO: Implement MCP client connection and tool discovery
+    # Return tools with prefixed names to avoid collisions (e.g., "mcp_classify_text")
+    return []
```

### 3. Graph Integration (Collision-Safe)

```diff
--- a/src/ollama_deep_researcher/graph.py
+++ b/src/ollama_deep_researcher/graph.py
@@ -13,6 +13,7 @@ from ollama_deep_researcher.configuration import Configuration, SearchAPI
 from ollama_deep_researcher.utils import (
     deduplicate_and_format_sources,
     tavily_search,
+    get_mcp_tools,
     format_sources,
     perplexity_search,
     duckduckgo_search,
@@ -299,6 +300,10 @@ def summarize_sources(state: SummaryState, config: RunnableConfig):

     # Run the LLM
     configurable = Configuration.from_runnable_config(config)
+
+    mcp_tools = []
+    if configurable.mcp_enabled:
+        mcp_tools = get_mcp_tools(configurable.mcp_server_url)

     # For summarization, we don't need structured output, so always use regular mode
     if configurable.llm_provider == "lmstudio":
@@ -313,6 +318,9 @@ def summarize_sources(state: SummaryState, config: RunnableConfig):
             model=configurable.local_llm,
             temperature=0,
         )
+
+    if mcp_tools:
+        llm = llm.bind_tools(mcp_tools)

     result = llm.invoke(
         [
```

### 4. Prompt Affordance

```diff
--- a/src/ollama_deep_researcher/prompts.py
+++ b/src/ollama_deep_researcher/prompts.py
@@ -69,6 +69,13 @@ When EXTENDING an existing summary:
 <Task>
 Think carefully about the provided Context first. Then generate a summary of the context to address the User Input.
 </Task>
+
+<TOOLS>
+If classification or analysis tools are available, use them to enhance the summary by:
+- Categorizing content type (technical, academic, news, opinion)
+- Assessing source authority and reliability
+- Rating relevance to the research topic
+</TOOLS>
 """

 reflection_instructions = """You are an expert research assistant analyzing a summary about {research_topic}.
```

### 5. State Extension (Optional)

```diff
--- a/src/ollama_deep_researcher/state.py
+++ b/src/ollama_deep_researcher/state.py
@@ -12,6 +12,7 @@ class SummaryState:
     sources_gathered: Annotated[list, operator.add] = field(default_factory=list)
     research_loop_count: int = field(default=0)  # Research loop count
     running_summary: str = field(default=None)  # Final report
+    mcp_tool_results: Annotated[list, operator.add] = field(default_factory=list)


 @dataclass(kw_only=True)
```

## Acceptance & Runbook

### Environment Configuration

```bash
# Required environment variables
export MCP_ENABLED=true
export MCP_SERVER_URL="stdio://demo_tools"

# Optional LLM settings
export LOCAL_LLM="llama3.2"
export LLM_PROVIDER="ollama"
export USE_TOOL_CALLING=true
```

### End-to-End Commands

```bash
# 1. Start MCP demo server (external dependency)
# TODO: Implement demo_tools MCP server with classify_text tool

# 2. Start LangGraph development server
langgraph dev

# 3. Test via API
curl -X POST "http://127.0.0.1:2024/runs/stream" \
  -H "Content-Type: application/json" \
  -d '{
    "assistant_id": "ollama_deep_researcher",
    "input": {"research_topic": "machine learning transformers"},
    "config": {
      "configurable": {
        "mcp_enabled": true,
        "mcp_server_url": "stdio://demo_tools"
      }
    }
  }'

# 4. Verify via LangGraph Studio
# https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

### Acceptance Checklist

- [ ] **MCP Tool Discovery**: `get_mcp_tools()` returns non-empty list when server available
- [ ] **Tool Binding**: LLM in `summarize_sources` node includes MCP tools in available functions
- [ ] **Collision Prevention**: MCP tools have prefixed names (e.g., `mcp_classify_text`)
- [ ] **State Updates**: `mcp_tool_results` field accumulates tool invocation outputs
- [ ] **Graceful Degradation**: Research workflow continues normally when MCP disabled
- [ ] **Configuration Override**: Environment variables `MCP_ENABLED=false` disables integration

### Verification Points

| Component | Check Method | Evidence Location |
|-----------|-------------|-------------------|
| Config Loading | Log `configurable.mcp_enabled` value | `summarize_sources()` entry |
| Tool Discovery | Log `len(mcp_tools)` | After `get_mcp_tools()` call |
| LLM Tool Binding | Inspect `llm.bound_tools` | Before `llm.invoke()` |
| Tool Invocation | Monitor `result.tool_calls` | LLM response processing |
| State Updates | Check `state.mcp_tool_results` | Graph state inspection |

## TODOs / Questions

### External Dependencies

- **MCP Server Implementation**: Need actual `demo_tools` MCP server with `classify_text` tool
  - Reference: <src/ollama_deep_researcher/utils.py:388-407> shows stub implementation
  - **Question**: What transport protocol to prioritize? (stdio vs HTTP)

- **MCP Client Library**: Requires `mcp` package installation and LangChain compatibility layer
  - Reference: <src/ollama_deep_researcher/graph.py:66> shows existing `bind_tools()` pattern
  - **Question**: How to handle MCP tool response format conversion to LangChain tools?

### Implementation Ambiguities

- **Tool Name Collision**: Current builtin tools (`Query`, `FollowUpQuery`) vs MCP tools
  - Reference: <src/ollama_deep_researcher/graph.py:162-171> and <src/ollama_deep_researcher/graph.py:352-364>
  - **Solution**: Namespace MCP tools with `mcp_` prefix

- **Error Handling**: MCP server unavailable or tool invocation failures
  - Reference: <src/ollama_deep_researcher/graph.py:69-77> shows existing fallback pattern
  - **Question**: Should MCP failures halt research or continue with degraded capability?

- **Configuration Validation**: Invalid MCP server URLs or unreachable endpoints
  - Reference: <src/ollama_deep_researcher/configuration.py:64-81> shows config loading pattern
  - **Question**: Validate MCP connectivity at startup or lazy discovery?

### Performance Considerations

- **Tool Discovery Caching**: Avoid repeated MCP server calls per graph execution
  - Reference: <src/ollama_deep_researcher/graph.py:300> shows per-invocation config loading
  - **Optimization**: Cache discovered tools at graph compilation time

- **Parallel Tool Invocation**: Multiple MCP tools in single LLM call
  - Reference: <src/ollama_deep_researcher/graph.py:316-321> shows single LLM invocation
  - **Question**: How to handle concurrent MCP tool calls within one summarization step?

### Integration Testing

- **Mock MCP Server**: Need test harness for CI/CD pipeline
  - **Requirement**: Deterministic responses for regression testing

- **LangGraph Studio Compatibility**: MCP tools should appear in visual interface
  - **Verification**: Tool calls visible in execution trace

---

**Analysis Methodology**: Static code analysis of 7 Python files, focusing on LangGraph patterns, tool definitions, and integration surfaces. No dynamic execution or MCP servers were involved in this analysis.