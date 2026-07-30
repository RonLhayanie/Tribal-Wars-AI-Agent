# Tribal Wars AI Agent

A Telegram bot that acts as a tactical advisor for the browser strategy game Tribal Wars.
It wraps Google's Gemini model in an agentic loop: the model receives the user's Hebrew
message, decides on its own which calculation tools to call, and answers from the tool
results rather than from its own arithmetic.

The bot is in production use by a single client and answers in Hebrew.

## How it works

`main.py` registers nine Python functions as Gemini tools and enables automatic function
calling. The model selects and invokes them without any routing logic in the application code:

| Tool | Purpose |
| --- | --- |
| `calculate_timing` | Back-times a launch so troops land at a given clock time |
| `calculate_distance` | Distance between two villages, by coordinate or saved name |
| `find_villages_in_range` | Scans saved villages for those within a travel-time radius |
| `nuke_calculator` | Attack and defence force requirements, including overstacking advice |
| `catapult_calculator` | Catapults needed to demolish a building, split into wave trains |
| `simulate_battle` | Rough attacker/defender power comparison with wall bonus |
| `get_optimal_scavenge` | Splits units across scavenging levels |
| `manage_memory` | save / get / get_all / delete against the JSON store |
| `set_reminder` | Schedules an asyncio task that messages the user later |

Two design points are worth calling out:

**Image analysis.** Telegram photos are downloaded, opened with Pillow and passed to Gemini
alongside the text prompt. The system prompt contains an explicit column-mapping procedure for
reading in-game report screenshots, where troop counts sit under unit icons and empty columns
must be preserved as zeros rather than skipped.

**Persistent memory.** `manage_memory` reads and writes `users_db.json`, a flat key-value store
of village names, coordinates and user-declared world rules. The system prompt instructs the
model to consult it before asking the user for data it may already have.

**Retry handling.** `TribalAgent.ask` retries up to three times on `503`, `UNAVAILABLE`, `429`
or quota errors, sleeping 2 seconds and then 4 seconds between attempts (linear, not
exponential). After the final attempt it returns a Hebrew message telling the user to wait.
Other exceptions are logged with a traceback and re-raised.

## Stack

- Python 3
- `google-genai` with `gemini-2.5-flash`
- `python-telegram-bot` (polling)
- `Pillow` for image handling
- `python-dotenv`

## Running it

```bash
pip install -r requirements.txt
```

Create a `.env` file in the project root:

```
GEMINI_KEY=your_gemini_api_key
TELEGRAM_TOKEN=your_telegram_bot_token
```

Then:

```bash
python main.py
```

The bot starts long polling. Send it a message, or `/help` for the command menu.

`data.json` holds static game data (unit speeds, attack and defence values, wall bonus
constants) and is loaded once at startup. It is injected into the system prompt so the model
can reason about units without a tool call.

## Structure

```
main.py                  everything: data layer, tools, agent class, Telegram handlers
data.json                static game constants
users_db.json            persistent key-value memory written by manage_memory (gitignored)
users_db.example.json    sample memory file showing the expected shape
requirements.txt
```

`users_db.json` holds live client data and is not tracked. Copy the example to get started:

```bash
cp users_db.example.json users_db.json
```

The file is a flat map of names to values. Coordinates are stored as `"x|y"` strings;
`manage_memory` will also write free-text entries such as world rules.

## Language

Code comments and log output are English. Everything the model reads or the user sees is
Hebrew: the system prompt, all tool return strings, the help menu and error replies. This is
deliberate. The bot's output quality in Hebrew depends on the prompt and tool descriptions
staying in Hebrew.

## Known limitations

- Everything lives in one 437-line module. The tools, the agent and the Telegram layer would
  be better as separate modules.
- A single `chats.create` session is built at import time and shared by every user, so
  conversation history is global rather than per-chat. The bot is currently single-user, which
  is why this has not caused problems.
- `current_bot` and `current_chat_id` are module-level globals set on each incoming message so
  that `set_reminder` can reach Telegram. This is not safe under concurrent chats.
- `users_db.json` is a single flat namespace with no per-user separation.
- Reminders are in-memory asyncio tasks and are lost if the process restarts.
- `simulate_battle` is a coarse heuristic and returns a verdict string, not a casualty model.
