# Repository Guidelines

## Project Structure & Module Organization

This Python teaching project demonstrates customer-support agents and DeepEval evaluations. Root modules contain the baseline (`agent_plain.py`), instrumented tool-calling agent (`agent_instrumented.py`), retrieval agent (`rag_agent.py`), and conversational chatbot (`chatbot.py`). `mcp_server.py` exposes a shipping-options tool through MCP. `policies.txt` supplies policy text for synthesized evaluation cases; the RAG knowledge base lives in `rag_agent.py`.

Evaluation scripts and teaching notes live in `evals/`. Dependencies are listed in `requirements.txt`. Consult source files for current paths and behavior: some examples in `README.md` and `CLAUDE.md` reference older filenames or configurations.

## Build, Test, and Development Commands

Run from the repository root with a virtual environment activated:

- `python -m pip install -r requirements.txt` — install declared dependencies.
- `python -m pip install nest_asyncio langchain-mcp-adapters mcp` — install additional imports used by the instrumented agent and MCP server that are currently absent from the requirements list.
- `python agent_instrumented.py` — smoke-test the tool-calling agent; it launches the MCP subprocess automatically.
- `python rag_agent.py` — smoke-test retrieval and response generation.
- `python chatbot.py` — start an interactive conversation; enter `quit` to exit.
- `python -m evals.test_TracingComponentsTest` — evaluate task completion and tool correctness.
- `python -m evals.test_rag_agent` / `python -m evals.test_chatbot` — run retrieval or conversational evaluations.

There is no separate build step.

## Coding Style & Naming Conventions

Use four-space indentation, `snake_case` functions and variables, and uppercase constants. Add type hints and concise docstrings to public functions and tools. Name new evaluation scripts `evals/test_<feature>.py`. Preserve the teaching examples' clarity and DeepEval tracing when changing agent behavior. No formatter or linter configuration is present.

## Testing Guidelines

Evaluations use DeepEval metrics, goldens, and conversational test cases. They execute live model calls, including during imports. Run the relevant script after changes and review scores and reasons against its configured thresholds. Add representative cases for changed behavior; supply expected outputs, tools, or retrieval context where required. No coverage target is configured.

## Commit & Pull Request Guidelines

Git history is unavailable in this checkout. Use concise imperative commit subjects, such as `Add shipping tool evaluation`. PRs should describe behavior changes, link relevant issues, and report evaluation commands, results, model choices, and dependency changes.

## Security & Configuration

Copy `.env.example` to `.env` and configure `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` for the selected agents and judges. Keep credentials and generated traces out of commits. Live evaluations incur API usage.
