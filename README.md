# Agent Harness

A runnable Python example of how to put reliable controls around an AI agent. The model can request tools; the harness decides whether to execute them, records what happened, and checks whether each run meets its contract.

The demo is a travel agent with flight search and purchase tools. Purchases are simulated in memory—no real bookings or payments are made.

## What this demonstrates

- **Bounded runs:** the harness limits model turns and applies model and tool deadlines.
- **Action-specific approval:** a purchase is allowed only when approval matches both the tool and its arguments.
- **Duplicate protection:** an idempotency key prevents repeating an approved action within the same harness instance.
- **Observable behavior:** model requests, tool calls, policy decisions, failures, and termination are recorded as structured events.
- **Contract-based evaluation:** JSON scenarios check expected event paths, forbidden events, final-answer criteria, and side effects.
- **Release gating:** deployment is allowed only when every scenario passes.

## Project files

| File | Purpose |
| --- | --- |
| `app.py` | Agent loop, tools, approval policy, event recording, scenario evaluation, and release decision |
| `scenarios.json` | Search, denied-purchase, and approved-purchase scenario contracts |
| `test_harness.py` | Deterministic tests for policy enforcement, idempotency, isolation, limits, and error handling |
| `pyproject.toml` / `uv.lock` | Python project metadata and locked dependencies |

## Requirements

- Python 3.11 or newer
- [uv](https://docs.astral.sh/uv/)
- [Ollama](https://ollama.com/) 0.12.7 or newer

## Quick start

Pull the model used by default:

```sh
ollama pull llama3.2:latest
```

Start the Ollama service in a separate terminal if it is not already running:

```sh
ollama serve
```

From the project directory, install dependencies and run the deterministic tests:

```sh
uv sync
uv run pytest -q
```

Run the live scenarios:

```sh
uv run python app.py
```

The final line should report:

```text
Deployment allowed: True
```

To confirm that a failing scenario blocks release, run:

```sh
uv run python app.py --demonstrate-failure
```

That command deliberately changes one expected event path. The affected scenario should fail and the report should end with `Deployment allowed: False`.

Use a different installed Ollama model with:

```sh
uv run python app.py --model <model-name>
```

Live model wording can vary. The scenario evaluator still checks required outcomes and side effects, so a model that misses a contract correctly blocks release.

## Safety and production notes

This repository demonstrates application-level controls; it is not a production purchasing service. Approval and idempotency state are in memory and scoped to one harness instance. A production system should use trusted approval identities, durable atomic idempotency coordinated with the transaction, tool-native deadlines and cancellation, concurrency controls, and appropriately protected telemetry.

## License

Licensed under the [MIT License](LICENSE).
