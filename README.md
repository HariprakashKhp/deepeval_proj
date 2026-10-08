# DeepEval Agentic Testing Demo

A minimal teaching repo that shows how to evaluate a LangChain agent with
DeepEval's component-level evaluation: **Task Completion** and **Tool
Correctness** metrics. OpenAI GPT-6 Luna generates responses and synthetic
test cases; GPT-4o judges and critics evaluate them. RAG embeddings use
`text-embedding-3-small`. Only `OPENAI_API_KEY` is required.

---

## Files in this repo

| File | Who writes it (in real life) | What it does |
|---|---|---|
| `agent_plain.py` | Developer (before evals exist) | A LangChain + OpenAI agent with two lookup tools and no evaluation code. |
| `agent_instrumented.py` | Developer (after QA asks) | Adds DeepEval tracing and an MCP shipping-options tool. |
| `evals/test_TracingComponentsTest.py` | QA / Tester (you) | Imports the agent, defines goldens, and runs metrics. |
| `rag_agent.py` / `evals/test_rag_agent.py` | Developer / Tester | Retrieves policy chunks and evaluates retrieval quality. |
| `chatbot.py` / `evals/test_chatbot.py` | Developer / Tester | Runs and evaluates multi-turn conversations. |

---

## The teaching point

The key instrumentation changes are small:

1. `from deepeval.integrations.langchain import CallbackHandler`
2. `deepeval_callback = CallbackHandler()`
3. `config={"callbacks": [deepeval_callback]}` on `.invoke()`
4. `update_current_trace(output=reply)` to expose the final response

The evaluation scripts add `@observe(name="support_agent")` to their wrappers
and copy expected values from the current golden into the trace.

Show students both files side by side. Once they see the diff is tiny,
they stop being intimidated by component-level evaluation.

---

## How to run

```bash
# 1. Install dependencies
python -m pip install -r requirements.txt

# 2. Set your API keys
# Copy .env.example to .env and set OPENAI_API_KEY.

# 3. (Optional but recommended) log in to Confident AI to see traces in a UI
deepeval login

# 4. Sanity check: run the agent on its own
python agent_instrumented.py

# 5. Run the evaluation
python -m evals.test_TracingComponentsTest
```

On Windows, activate `.venv_new\Scripts\Activate.ps1` first, or use
`.\.venv_new\Scripts\python.exe` in place of `python`. In PyCharm, select
that executable as the project interpreter. The MCP subprocess uses the
same interpreter as the agent.

Run `python -m evals.test_rag_agent` or `python -m evals.test_chatbot` for
the other demos. `python -m evals.test_agent_synthesized` generates goldens
with Luna and evaluates them with GPT-4o. These commands make live API calls.

---

## What you'll see when you run `evals/test_TracingComponentsTest.py`

DeepEval will:
1. Invoke `support_agent(...)` for each golden input
2. Capture the full trace (LangChain LLM call + tool call + final message)
3. Apply **Task Completion** to the whole trace
4. Apply **Tool Correctness** to the tool-call span
5. Print scores + reasons per golden

The two goldens cover order status and refund policy. Compare task completion
with tool correctness: a plausible answer and the right tool choice are
separate evaluation targets.

---

## Suggested lesson flow

| Minute | Activity |
|---|---|
| 0–5 | Show `agent_plain.py`. Ask: "How would you test this?" |
| 5–10 | Show `agent_instrumented.py` side-by-side. Walk through tracing and MCP setup. |
| 10–25 | Walk through `evals/test_TracingComponentsTest.py` line by line. |
| 25–35 | Run it live. Discuss the borderline result. |
| 35+ | Open discussion: what other metrics? what other goldens? |

---

## Reality checks before teaching

DeepEval's integration surface evolves. Before your first session, run the
demo end-to-end once and confirm:

- Agents use `ChatOpenAI(model="gpt-6-luna", reasoning_effort="none", temperature=0)`.
  Luna's Chat Completions function calls require `reasoning_effort="none"`.
  Confirm your OpenAI account can access this model.
- `from deepeval.integrations.langchain import CallbackHandler` — works as
  written in recent DeepEval versions; older versions may use a different
  import path.
- `Golden(expected_tools=[ToolCall(name="...")])` — accepted in current
  DeepEval; in older versions you may need to pass via `additional_metadata`
  and unpack via `update_current_span`.

Fix any version issue once, then it's solid for the whole course.
