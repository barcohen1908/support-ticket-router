# Homework 3 — TaskNest Support Ticket Router

## State and categories

TaskNest is a small to-do app. Accept a support ticket as text and assign exactly one category. Use a `TypedDict` with four keys. The caller supplies `ticket`; the graph fills `category`, `team`, and `reply`.

- `billing` covers charges, refunds, invoices, and subscriptions.
- `technical` covers crashes, errors, bugs, and login problems.
- `general` covers other questions, suggestions, and compliments.

Classify by the primary problem. If the app crashes when the customer tries to pay, choose `technical` because the crash is the primary problem.

## Model and classification

Use `gpt-5.4-nano` as the sole model name in a notebook constant. It is a low-cost text model with Structured Outputs, and the task requires only a single label from a small set. Set `reasoning_effort="none"` so the model accepts `temperature=0`. Disable automatic retries to keep each ticket to one API attempt. [OpenAI model guide](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-5.4)

The `classify` node invokes `ChatOpenAI` once through `with_structured_output` using JSON Schema and a Pydantic model with `category: Literal["billing", "technical", "general"]`. It writes the parsed category into state.

## Graph

Connect `START` to `classify`. Use exactly one `add_conditional_edges` call after classification. Its routing function returns only `state["category"]`. Map `billing` to `billing_team`, `technical` to `tech_team`, and `general` to `general_team`. Each team node uses plain Python to set a team name and a short fixed English reply, then proceeds to `END`. The graph has no loops, tools, memory, or additional LLM calls.

## Colab and verification

Install dependencies with `!pip -q install langgraph langchain-openai`. Read `OPENAI_API_KEY` from Google Colab Secrets with `userdata.get` and assign it to `os.environ["OPENAI_API_KEY"]` for the runtime. Keep the key out of notebook source and output.

Display the compiled graph using `draw_mermaid_png()`. Explain the LLM call and conditional edge in short Markdown cells. Run `graph.invoke({"ticket": "I was charged twice this month"})` and at least six original tickets, with at least two per category and at least one ambiguous ticket. Print the ticket, category, team, and reply for each invocation. Run cells from top to bottom in a fresh Colab session.
