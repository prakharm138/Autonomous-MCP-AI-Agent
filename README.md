# Autonomous MCP Agent

A small, beginner-friendly **AI agent** built from scratch in Python. You type a request in plain English, an LLM decides which tools to use, the tools run, and you get a natural-language answer back.

No big framework. Just a few small files that each do one job, so you can read the whole thing in ten minutes and see how agents actually work.

```
You:   what is the time and roll a dice
Agent: It's currently 12:16 AM on 3 November 2026, and your dice roll came up 4.
```

---

## What is an "AI agent"?

An agent is a loop of three things:

> **Decide → Act → Remember**, repeated until the job is done.

1. **Decide:** an LLM looks at your request and picks the next tool to use.
2. **Act:** the chosen tool actually runs (get the time, roll a dice, etc.).
3. **Remember:** the result is written to a "notepad" (the *state*), so the LLM knows what's already done on the next round.

When the LLM decides nothing is left to do, it says `FINISH` and a final answer is written for you.

---

## How it works

```
 You type a request
        │
        ▼
   runner.py ──► connects to the tool server, lists the tools, asks you for input
        │
        ▼
    loop.py  ──► repeats the steps below
        │
        ├─ planner.py   → asks the LLM: "what is the ONE next action?"
        ├─ executor.py  → runs that tool through the MCP client
        │      └─ mcp_client.py ──► server.py (the tool actually runs here)
        └─ state.py     → saves what happened
        │
        ▼
 Planner says FINISH
        │
        ▼
 runner.py asks the LLM to write a friendly final answer
```

### The files

| File | What it does | Think of it as |
|---|---|---|
| `server.py` | Defines the tools: `current_time`, `roll_dice`, `generate_password` | The toolbox |
| `mcp_client.py` | Connects to the server, lists tools, calls them | The phone line to the toolbox |
| `state.py` | Stores the request, actions, observations, step count | The notepad |
| `planner.py` | Asks the LLM which tool to use next (or `FINISH`) | The brain |
| `executor.py` | Runs the chosen tool and returns plain text | The hands |
| `loop.py` | The main loop: plan → execute → remember → repeat | The manager |
| `runner.py` | Entry point; wires everything together and writes the final answer | The front door |
| `test_state.py` | Tests the state logic with no AI involved | A practice run |

### A walk-through

Request: **"what is the time and roll a dice"**

| Round | Planner picks | Tool result | Notepad |
|---|---|---|---|
| 1 | `current_time` | `03-11-2026 00:16:42` | 1 action recorded |
| 2 | `roll_dice` | `4` | 2 actions recorded |
| 3 | `FINISH` | n/a | marked finished |

Then a second LLM call turns the raw results into a friendly sentence.

---

## Getting started

### Requirements

- Python 3.10 or newer
- An API key for any **OpenAI-compatible** LLM endpoint (OpenAI, OpenRouter, a local server like Ollama or LM Studio, etc.)

### 1. Clone the repo

```bash
git clone https://github.com/YOUR-USERNAME/simple-mcp-agent.git
cd simple-mcp-agent
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Add your settings

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

Then edit `.env`:

```
BASE_URL=https://api.openai.com/v1
API_KEY=your-api-key-here
MODEL=gpt-4o-mini
```

> Your `.env` file is listed in `.gitignore`, so your key will not be uploaded to GitHub.

### 5. Run it

```bash
python runner.py
```

You'll see the connected tools, then a prompt:

```
Connected to MCP Server.

Available Tools
----------------
current_time
roll_dice
generate_password

User : what is the time and roll a dice
Planner Selected: current_time
Planner Selected: roll_dice
Planner Selected: FINISH

Final Answer:
It's currently 12:16 AM on 3 November 2026, and your dice roll came up 4.
```

---

## Things to try

- `Generate a password and tell me the current time`
- `Roll a dice`
- `What time is it?`

## Adding your own tool

This is the nice part: you only touch **one file**. Add a function to `server.py`:

```python
@mcp.tool()
def flip_coin():
    """Flip a coin and return heads or tails."""
    return random.choice(["heads", "tails"])
```

Restart the agent. It discovers the new tool automatically, and the planner can start using it. The docstring matters: the LLM reads it to understand what the tool does, so write it clearly.

## Running the state test

`test_state.py` checks the notepad logic without any LLM or server. If you run it from this folder, change the first line to:

```python
from state import *
```

then run:

```bash
python test_state.py
```

---

## Known limitations

These are good first improvements if you want to extend the project:

- **No tool arguments.** Tools are called without arguments, so `generate_password` always makes a 12-character password. Supporting arguments would mean the planner returns structured JSON such as `{"tool": "generate_password", "arguments": {"length": 20}}`.
- **Fragile planner output.** The planner's reply is used directly as a tool name. If the LLM adds extra words, the call fails. Validating the reply against the tool list (and retrying) would fix this.
- **Empty answer at the step limit.** If `max_steps` is reached before the planner says `FINISH`, no final answer is generated.

## Tech used

- [Python](https://www.python.org/)
- [FastMCP](https://github.com/jlowin/fastmcp) for the MCP server and client
- [OpenAI Python SDK](https://github.com/openai/openai-python) for talking to the LLM
- [python-dotenv](https://github.com/theskumar/python-dotenv) for loading settings

## Contributing

Suggestions and pull requests are welcome. If you find a bug or have an idea, open an issue.

## License

This project is licensed under the MIT License. Add a `LICENSE` file to your repo to make this official (GitHub can generate one for you when you create the repository).
