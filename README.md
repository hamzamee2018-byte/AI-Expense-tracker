# SpendWise - AI-Powered Personal Expense Tracker

A personal finance dashboard built with **Python + Reflex**, **SQLite** storage and
**Google Gemini** for spending insights.

Features: dashboard (income, expenses, balance, monthly spending, charts, recent
transactions), add/edit/delete/search/filter transactions, category budgets with
progress bars, an AI assistant that analyzes your real data, and CSV export.

> **Testing status:** the database, validation, totals, budget and CSV logic were
> tested in isolation. The full Reflex UI and the live Gemini call have **not** been
> run end to end by the author. If something fails on first start, see
> [Troubleshooting](#troubleshooting).

## Requirements

- Python **3.11 or newer** (required by Reflex 0.10)
- A free Gemini API key (only needed for the AI Assistant page)

## 1. Project setup

Reflex expects a specific folder layout: a project folder containing `rxconfig.py`
and an inner package with the same name as the app. Use the project name
`expense_tracker`.

```bash
mkdir expense_tracker
cd expense_tracker

# create and activate a virtual environment
python -m venv venv
source venv/bin/activate          # macOS / Linux
venv\Scripts\activate             # Windows (PowerShell or cmd)

# put requirements.txt in this folder, then:
pip install -r requirements.txt

# create the Reflex project skeleton (choose the blank template if asked)
reflex init
```

`reflex init` creates `rxconfig.py` and the inner folder `expense_tracker/`
containing `expense_tracker.py` and `__init__.py`.

**Now replace the generated app file with `app.py`:**

```bash
# macOS / Linux
cp /path/to/app.py expense_tracker/expense_tracker.py

# Windows
copy C:\path\to\app.py expense_tracker\expense_tracker.py
```

Your folder should look like this:

```
expense_tracker/                 <- project root (run commands here)
|-- rxconfig.py                  <- created by `reflex init`
|-- requirements.txt
|-- .env                         <- optional, holds your API key (step 2)
`-- expense_tracker/
    |-- __init__.py
    `-- expense_tracker.py       <- the contents of app.py
```

### Reflex configuration

`rxconfig.py` must name the app after the inner folder. `reflex init` does this for
you, and it should look like:

```python
import reflex as rx

config = rx.Config(
    app_name="expense_tracker",
)
```

The app module must define `app = rx.App(...)` at module level; `app.py` already does.
No other Reflex configuration is required. Reflex downloads its own Node/Bun tooling
on the first run, so the first start needs internet access and takes a while.

## 2. Configure the Gemini API key

1. Create a key in [Google AI Studio](https://aistudio.google.com/apikey).
2. Provide it to the app as the **`GEMINI_API_KEY`** environment variable. Pick one way:

**Option A - `.env` file (easiest).** Create a file named `.env` in the project root
(next to `rxconfig.py`):

```
GEMINI_API_KEY=your-key-here
```

**Option B - set it in the terminal before running.**

```bash
export GEMINI_API_KEY="your-key-here"        # macOS / Linux
$env:GEMINI_API_KEY="your-key-here"          # Windows PowerShell
set GEMINI_API_KEY=your-key-here             # Windows cmd
```

**Never** paste the key into the code or commit `.env` to Git. If you use Git, add
`.env` and `*.db` to `.gitignore`.

Without a key, everything except the AI Assistant still works, and that page shows a
setup message instead.

### Optional settings

| Variable | Default | Purpose |
|---|---|---|
| `GEMINI_MODEL` | `gemini-3.8-flash` | Gemini model ID to use |
| `CURRENCY_SYMBOL` | `$` | Symbol shown before amounts (for example `Rs `) |
| `EXPENSE_DB_PATH` | next to the app file | Location of the SQLite file |

**About the model ID:** `gemini-3.8-flash` was the stable Flash model in Google's Gemini
API docs on 22 Oct 2026 (`gemini-3.7-flash` is deprecated). Models are retired over time.
If you see "model not found", check the current list at
<https://ai.google.dev/gemini-api/docs/models> and set `GEMINI_MODEL`. The app will
never switch models silently.

## 3. Run the app

From the project root, with the virtual environment active:

```bash
reflex run
```

Open <http://localhost:3000>. Stop with `Ctrl+C`.

## Using the app

- **Dashboard** - totals, this month's spending, charts and the latest transactions.
- **Transactions** - add, edit and delete records; search by note or category; filter
  by type, category and month; **Export CSV** downloads the transactions that match
  the current filters (clear the filters to export everything).
- **Budgets** - set a monthly limit per expense category. Bars turn amber at 80% and
  red when you go over.
- **AI Assistant** - click *Analyze my spending* (optionally type a question).

## Data and privacy

- All records live in a local SQLite file (`expense_tracker.db`, created automatically
  inside the inner `expense_tracker/` folder unless you set `EXPENSE_DB_PATH`). Back it
  up by copying the file.
- When you use the AI Assistant, a **summary** of your data is sent to Google's Gemini
  API: totals, category breakdowns, budgets, and the date, category, amount and note
  of your 8 largest recent expenses. Avoid putting sensitive details in notes if that
  concerns you. Free-tier Gemini usage may be subject to Google's data-use terms, so
  review them before use.
- AI output is general guidance, not professional financial advice.

## Troubleshooting

| Problem | Fix |
|---|---|
| `reflex: command not found` | Activate the virtual environment, then rerun `pip install -r requirements.txt`. |
| Python version error | Install Python 3.11 or newer. |
| App fails with an import error for the app module | Check that `app_name` in `rxconfig.py` equals the inner folder name and that the file is `expense_tracker/expense_tracker.py`. |
| AI page says the key is not set | Confirm the variable is named `GEMINI_API_KEY`, then **restart** `reflex run`. |
| "Gemini rejected the API key" | Regenerate the key in Google AI Studio and update it. |
| "Model ... was not found" | Set `GEMINI_MODEL` to a current Flash model ID from the docs. |
| Rate limit / quota message | The free tier has limits. Wait a minute and retry. |
| A Reflex component or prop error on start | The UI targets Reflex 0.10 but was not run by the author. Share the full error message for a fix. |
| Want to start fresh | Stop the app and delete `expense_tracker.db`. |
