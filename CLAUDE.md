# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Local Deep Researcher is a fully local web research assistant that uses LLMs via Ollama or LMStudio. It performs iterative web research using LangGraph, where it:

1. Generates search queries from research topics
2. Searches the web using multiple search engines (DuckDuckGo, Tavily, Perplexity, SearXNG)
3. Summarizes findings and reflects on knowledge gaps
4. Iterates through multiple research loops
5. Produces a final markdown report with citations

## Architecture

### Package Structure

- **Package name**: `ollama-deep-researcher` (with hyphens in pyproject.toml)
- **Python module**: `ollama_deep_researcher` (with underscores)
- **Source location**: `src/ollama_deep_researcher/`
- **LangGraph graph**: Defined in `langgraph.json` as `ollama_deep_researcher`

### Core Components

- **`graph.py`**: Main LangGraph implementation defining the research workflow with nodes for query generation, web search, summarization, and reflection
- **`configuration.py`**: Configuration management with environment variable support and Pydantic models
- **`state.py`**: Graph state definition using dataclasses for research topics, queries, results, and summaries
- **`utils.py`**: Utility functions for web search integration, source formatting, and content processing
- **`prompts.py`**: System prompts for LLM interactions in query generation, summarization, and reflection
- **`lmstudio.py`**: LMStudio integration for OpenAI-compatible API access

### LangGraph Workflow

The main graph is defined in `src/ollama_deep_researcher/graph.py:graph` and includes:

- Query generation (structured output with tool calling or JSON mode)
- Web search with multiple provider support
- Content summarization with source tracking
- Reflection for knowledge gap identification
- Iterative research loops with configurable depth

### LLM Provider Support

- **Ollama**: Direct integration via `langchain-ollama`
- **LMStudio**: OpenAI-compatible API wrapper in `lmstudio.py`
- Both support structured output via tool calling or JSON mode fallback

## Development Commands

### Environment Setup

```bash
# Copy environment template
cp .env.example .env

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\Activate.ps1  # Windows

# Install with uv (recommended)
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
uvx --refresh --from "langgraph-cli[inmem]" --with-editable . --python 3.11 langgraph dev

# Alternative: install with pip
pip install -e .
pip install -U "langgraph-cli[inmem]"
```

### Running the Application

```bash
# Start LangGraph development server
langgraph dev

# The server will be available at:
# - API: http://127.0.0.1:2024
# - Docs: http://127.0.0.1:2024/docs
# - Studio: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

### Code Quality

```bash
# Install development dependencies with uv (recommended)
uv sync --group dev

# Alternative: install with pip
pip install -e ".[dev]"

# Linting and formatting
ruff check .
ruff format .

# Type checking
mypy src/

# Check specific files
ruff check src/ollama_deep_researcher/
mypy src/ollama_deep_researcher/
```

### Docker

```bash
# Build image
docker build -t local-deep-researcher .

# Run container (requires external Ollama)
docker run --rm -it -p 2024:2024 \
  -e SEARCH_API="duckduckgo" \
  -e LLM_PROVIDER=ollama \
  -e OLLAMA_BASE_URL="http://host.docker.internal:11434/" \
  -e LOCAL_LLM="llama3.2" \
  local-deep-researcher
```

## Configuration

### Environment Variables

Configuration follows this priority order:

1. Environment variables (highest)
2. LangGraph UI configuration
3. Default values in Configuration class (lowest)

Key variables in `.env`:

- `LLM_PROVIDER`: "ollama" or "lmstudio"
- `LOCAL_LLM`: Model name (e.g., "llama3.2", "qwen_qwq-32b")
- `SEARCH_API`: "duckduckgo", "tavily", "perplexity", or "searxng"
- `MAX_WEB_RESEARCH_LOOPS`: Research iteration count (default: 3)
- `USE_TOOL_CALLING`: Use tool calling vs JSON mode for structured output
- `FETCH_FULL_PAGE`: Include full page content in search results (default: true)

### LLM Configuration

- Ollama: Configure `OLLAMA_BASE_URL` and `LOCAL_LLM`
- LMStudio: Configure `LMSTUDIO_BASE_URL` and `LOCAL_LLM`
- Some models (like DeepSeek R1 variants) may require tool calling instead of JSON mode

### Search APIs

- DuckDuckGo: No API key required (default)
- Tavily: Requires `TAVILY_API_KEY`
- Perplexity: Requires `PERPLEXITY_API_KEY`
- SearXNG: Requires `SEARXNG_URL`

## Code Patterns

### Adding New Search Providers

1. Add enum value to `SearchAPI` in `configuration.py`
2. Implement search function in `utils.py` following existing patterns
3. Add provider handling in graph search node
4. Update configuration validation

### Structured Output Handling

The codebase supports both tool calling and JSON mode for LLM structured output:

- Tool calling: Preferred for newer models
- JSON mode: Fallback for models without tool calling support
- Configure via `USE_TOOL_CALLING` environment variable

### State Management

All workflow state uses dataclasses in `state.py` with LangGraph annotations:

- `SummaryState`: Main graph state with research data
- `SummaryStateInput`: Graph input schema
- `SummaryStateOutput`: Graph output schema

## Testing and Validation

### Dependency Management

- **Primary**: Use `uv` for fast dependency management and virtual environments (`uv sync`, `uv sync --group dev`)
- **Alternative**: Standard `pip` for traditional Python workflows (`pip install -e ".[dev]"`)
- **Recommended**: Always use `uv` for new development work

### Model Compatibility

Test new models with both structured output modes:

- Verify JSON parsing works correctly
- Check tool calling functionality
- Validate multi-turn conversation handling

### Search Integration Testing

When adding search providers:

- Test API key validation
- Verify result formatting consistency
- Check error handling for rate limits/failures

## Browser Compatibility

LangGraph Studio works best with:

- Firefox (recommended)
- Chrome/Edge (good support)
- Safari may have mixed content issues with HTTPS/HTTP
