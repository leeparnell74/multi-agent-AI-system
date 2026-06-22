# Multi-Agent AI System

A customer support system for a digital music store, built with LangGraph and LangSmith. It combines specialized sub-agents under a supervisor, with human-in-the-loop verification and long-term memory.

## Overview

The system routes customer queries to two specialized ReAct sub-agents:
- **Music Catalog sub-agent** — handles questions about artists, albums, songs, and genres
- **Invoice Information sub-agent** — handles billing and purchase history queries

A **supervisor agent** decides which sub-agent to call based on the query and can chain multiple sub-agents for multi-part questions. The full workflow adds customer verification and memory:

`verify_info → load_memory → supervisor (sub-agents) → create_memory`

## Getting Started

Requires Python 3.10+.

```bash
git clone https://github.com/leeparnell74/multi-agent-AI-system
cd Multi-Agent-AI-System
pip install -r requirements.txt
```

Set your API keys as environment variables (`OPENAI_API_KEY`, `LANGSMITH_API_KEY`), and enable LangSmith tracing.

## Key Components

**Dataset** — Uses the [Chinook Database](https://github.com/lerocha/chinook-database) (SQLite), a sample digital music store with customers, invoices, and a music catalog. Loaded in-memory from the official SQL script.

**Memory** — Two types:
- *Short-term* (`MemorySaver` checkpointer) keeps context within a single conversation
- *Long-term* (`InMemoryStore`) persists user preferences across conversations

**Sub-agents** — The music agent is built manually from State, Tools, and Nodes (a ReAct loop with conditional edges). The invoice agent uses LangGraph's pre-built `create_react_agent` for the same pattern with less code.

**Supervisor** — Built with `create_supervisor`, routing between sub-agents and returning control after each completes.

**Human-in-the-Loop** — A `verify_info` node extracts a customer identifier (ID, email, or phone) and validates it against the database. If missing, a `human_input` node uses `interrupt()` to pause and request it. Once verified, `customer_id` persists in state so the user isn't asked again.

**Long-Term Memory** — `load_memory` loads saved preferences at the start; `create_memory` analyzes the conversation at the end and saves new music interests using an LLM with a structured `UserProfile` schema.

## Evaluation

Evaluation uses LangSmith and has three parts: a **dataset** (test questions with expected answers), a **target function** (the agent under test), and **evaluators** (which score outputs). Common evaluation types include final response, single step (tool choice), and trajectory (full reasoning path). This project uses an LLM-as-judge to compare agent responses against ground-truth answers.

## Supervisor vs. Swarm

| Feature | Supervisor | Swarm |
| :--- | :--- | :--- |
| **Control Flow** | Centralized; one agent directs traffic | Decentralized; agents hand off directly |
| **Hierarchy** | Hierarchical "boss" delegates to sub-agents | Peer-to-peer; no central authority |
| **Predictability** | More structured, predictable path | Adaptive, dynamic collaboration |
| **Resilience** | Control returns to the supervisor | Potentially more resilient via direct collaboration |

## Tech Stack

LangGraph · LangSmith · OpenAI · SQLite (Chinook) · Python 3.10+

## License

MIT