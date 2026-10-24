# Emergency Budget Agent

A minimal Python + OpenAI integration for the
`calculate_emergency_focused_budget` tool. It includes:

- The function-calling JSON schema.
- Deterministic budget calculations and input validation.
- A chat-completions tool-call loop, including parallel tool-call handling.
- Structured errors returned to the model when arguments are invalid.
- A command-line interface and offline unit tests.

## Allocation policy

The target emergency fund is:

```text
fixed monthly expenses × target months (3–6, default 6)
```

Until the target is met, **all positive monthly surplus** goes to the emergency
fund. In the final month, only the remaining gap is allocated; any surplus left
over is available for other goals. If income does not cover fixed expenses, the
tool returns a zero contribution and a clear deficit or no-surplus status.

## Project files

```text
emergency_budget_agent/
├── agent.py                  # OpenAI agent loop and CLI
├── budget_tool.py            # Schema, calculation, validation, dispatch
├── requirements.txt
└── tests/
    └── test_budget_tool.py   # Offline calculation and dispatch tests
```

## Setup

Python 3.10 or newer is recommended.

```bash
cd emergency_budget_agent
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
export OPENAI_API_KEY="your-api-key"
```

The default model is `gpt-4.1-mini`. You can select another model available to
your account with `--model` or the `BUDGET_AGENT_MODEL` environment variable.

## Run the agent

```bash
python agent.py "My monthly take-home income is 100000, fixed expenses are 50000, and I already have 100000 reserved for emergencies. Use a 6-month target."
```

You can also run `python agent.py` and enter the prompt interactively.

The amounts are currency-agnostic. Include a currency in your prompt if you want
the response to display one.

## Use the calculator without an LLM

```python
from budget_tool import calculate_emergency_focused_budget

result = calculate_emergency_focused_budget(
    monthly_salary=100_000,
    fixed_expenses=50_000,
    current_emergency_fund=100_000,
    target_months=6,
)
print(result)
```

For this example, the target fund is 300,000, the remaining gap is 200,000, and
the monthly emergency contribution is 50,000. At the same surplus, the target is
reachable in four months.

## Run tests

No API key or network access is needed:

```bash
python -m unittest discover -s tests -v
```

## Integration notes

`TOOLS` uses the nested function schema expected by OpenAI chat completions.
If you migrate to the Responses API, convert each tool to that API's flattened
function-tool format rather than passing the nested schema unchanged.

The tool performs general budgeting arithmetic only; it does not provide
individualized investment, tax, or legal advice.
