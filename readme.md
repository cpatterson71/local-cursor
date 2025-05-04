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
![Architecture](https://github-production-user-asset-6210df.s3.amazonaws.com/209818798/440162845-5d8e3b21-2095-41f6-b1db-e0c6c047958f.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20250504%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250504T053828Z&X-Amz-Expires=300&X-Amz-Signature=7a44b959a613474452f0e24e21d884f080f7e78438a8e5c39aa2c8ff054c7ca7&X-Amz-SignedHeaders=host)

1. Natural language input from the terminal is sent to the model, along with a system prompt that lists the available tools.
2. The local LLM (via Ollama) analyzes the request and responds with either a plain-text answer or a structured tool call.
3. If a tool call is issued, the OllamaAgent executes the corresponding function (e.g., read/write a file, run a shell command) and sends the result back to the model.
4. This loop continues until the model produces a final answer, which is then printed in your terminal.

All logic is in [`main.py`](./main.py); the heavy lifting is done by the open‑source model running locally. 


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
requests       # To make EXA API calls
click          # To write CLI
colorama       # To format CLI output
openai         # To create an OpenAI client
python‑dotenv  # To load environment variables from our .env file
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


## Contributing

1. Fork & clone
2. Create a virtualenv and install dev dependencies (`pip install -r requirements-dev.txt`)
3. Create a PR 


### Acknowledgements

* [Ollama](https://ollama.ai/) for making local LLMs drop‑dead simple
* [Exa](https://exa.ai/) for the search API
* The open‑source community that keeps pushing offline AI forward

