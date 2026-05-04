# AGENTS.md

## Repo Overview

Educational repository for "Agentic Design Patterns" by Antonio Gulli.
Contains a PDF book and 66 Jupyter notebooks demonstrating AI agent design patterns.

**No build system, CI, tests, or package manager config exists.** This is a notebook-only repo.

## Structure

- `Agentic_Design_Patterns_Complete.pdf` — full book (424 pages)
- `chapter_notebooks/` — all code examples, one or more `.ipynb` per chapter

## Frameworks Used (varies by notebook)

| Framework | Prefix/Pattern | Import |
|-----------|---------------|--------|
| LangChain | `Chapter_01_*`, `Chapter_02_Routing_(LangGraph)*`, `Chapter_05_Tool_Use_(LangChain)*` | `langchain_openai`, `langchain_google_genai`, `langchain_core` |
| Google ADK | `*_ADK_*`, `*_Gemini_*` | `google.adk.agents` |
| CrewAI | `*_CrewAI*` | `crewai` |
| FastMCP | `*_FastMCP*`, `*_MCP_*` | `fastmcp` |
| OpenAI direct | some chapters | `openai` |

## Running Notebooks

```bash
pip install jupyter notebook
jupyter notebook chapter_notebooks/Chapter_01_Prompt_Chaining_(Code_Example).ipynb
```

## API Keys Required

Notebooks expect API keys as environment variables. Common ones:
- `OPENAI_API_KEY` — for OpenAI/LangChain examples
- `GOOGLE_API_KEY` — for Gemini/ADK examples

No `.env` file or key management is included. Set keys before running.

## Notebook Naming Convention

`Chapter_{NN}_{PatternName}({Variant}).ipynb`

- Multiple variants per chapter show the same pattern with different frameworks
- Appendices labeled `Appendix_A_*` through `Appendix_G_*`

## Key Constraints

- Python 3.8+ required
- No `requirements.txt` — install deps per notebook comments (first cell usually has `pip install` instructions)
- Notebooks are standalone; no cross-notebook dependencies
- No tests to run; verify by executing notebooks
- Some notebooks use simulated/mock data (e.g., stock prices) rather than real APIs
