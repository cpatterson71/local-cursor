# Local Cursor

> **Privacy‑first AI coding assistant that lives entirely in your terminal. No clouds, no compromises.**

Local Cursor wraps a local large‑language‑model (LLM) served by [Ollama](https://ollama.ai/) in a simple CLI, giving you Copilot‑style help without ever sending a byte of your code to external servers.


## Key features

| Capability     | What it means                                                    |
| -------------- | ---------------------------------------------------------------- |
| File tooling   | Read, write, list & search files via natural‑language requests   |
| Shell commands | Safely run whitelisted commands (`ls`, `grep`, `find`, …)        |
| Web search     | Optional Exa API integration for up‑to‑date answers              |
| Local LLM      | Ships with **`qwen3:32b`** by default—swap in any Ollama model   |
| Extensible     | Add new tools, commands or UI layers without touching model code |


## Quick start

### 1 · Prerequisites

* **Python ≥ 3.8**
* **Ollama** running locally (`brew install ollama` or see the [docs](https://ollama.ai/))
* *(Optional)* Exa API key for web search

### 2 · Install & run

```bash
# Clone and enter the repo
git clone https://github.com/towardsai/local-cursor.git
cd local‑cursor

# Set up a virtualenv
python -m venv .venv && source .venv/bin/activate

# Install Python deps
pip install -r requirements.txt

# Pull a model & start Ollama
ollama pull qwen3:32b
ollama serve      # keep this terminal running

# (Optional) add your Exa key
echo "EXA_API_KEY=sk‑..." > .env

# Fire up the assistant
python main.py --model qwen3:32b
```

*Tip →* run `python main.py --help` for all CLI flags (debug mode, model override, …).


## How it works

```
┌─────────────┐    messages    ┌───────────────┐
│ Your shell  │──────────────▶│   Ollama LLM   │
└─────┬───────┘                └───────────────┘
      │ tools (JSON‑RPC)               │
┌─────▼────────┐               ┌──────▼────────┐
│  OllamaAgent │——executes──▶  │  System tools │
└──────────────┘               └───────────────┘
```

1. **Natural‑language input** is sent to the model together with a *system prompt* that lists available tools.
2. The model replies with either plain text or a structured *tool call*.
3. `OllamaAgent` executes the requested tool (read/write file, run command, …) and feeds the result back to the model so it can continue reasoning.
4. The final answer is printed in your terminal.

All logic lives in [`main.py`](./main.py); the heavy lifting is done by the open‑source model running locally. 


## Available tools

| Tool                               | What it does                                         |
| ---------------------------------- | ---------------------------------------------------- |
| `list_files(path=".")`             | Show directories and files with human‑friendly icons |
| `read_file(path)`                  | Return the full text of a file                       |
| `write_file(path, content)`        | Create or overwrite a file                           |
| `find_files(pattern)`              | Glob search (e.g. `**/*.py`)                         |
| `run_command(cmd)`                 | Execute a whitelisted shell command                  |
| `web_search(query, num_results=5)` | Query the web via Exa                                |

Add your own by editing `get_tools_definition()`—the model will “see” them automatically at runtime.


## Python dependencies

Local Cursor keeps its runtime lean:

```text
requests       # HTTP for Exa API
click          # Ergonomic CLI interface
colorama       # ANSI colours for cross‑platform terminals
openai         # Thin client used to call the Ollama API
python‑dotenv  # Convenience loader for .env files
pytest         # Unit testing helpers
```

(See [`requirements.txt`](./requirements.txt) for exact versions.) 


## Security considerations

* Only a safe subset of shell commands is allowed by default—edit `allowed_commands` in `run_command()` to adjust. 
* All file paths are resolved inside the current working directory to avoid accidental system‑wide access.


## Troubleshooting

| Symptom                                      | Fix                                                        |
| -------------------------------------------- | ---------------------------------------------------------- |
| `Error: Exa API key not configured`          | Add `EXA_API_KEY` to `.env` or disable web search          |
| `command … not allowed for security reasons` | Add it to `allowed_commands` (understand the risks first!) |
| High memory usage                            | Try a smaller Ollama model like `phi3:4b`                  |


## Roadmap

* Tool calling (v0.1)
* Persistent chat history
* Web UI (FastAPI + React)
* Plugin system for custom workflows

Feel free to open issues or PRs with ideas!

## Contributing

1. Fork & clone
2. Create a virtualenv and install dev dependencies (`pip install -r requirements-dev.txt`)
3. Run `pytest` – tests must pass
4. Follow the [Conventional Commits](https://www.conventionalcommits.org/) spec for commit messages


### Acknowledgements

* [Ollama](https://ollama.ai/) for making local LLMs drop‑dead simple
* [Exa](https://exa.ai/) for the search API
* The open‑source community that keeps pushing offline AI forward

