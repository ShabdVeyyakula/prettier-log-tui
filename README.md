# prettier

A drop-in wrapper that turns any noisy log stream into a live, readable TUI.

```
prettier uv run python -m app.temporal.worker
prettier -- some-command --with --flags
```

It runs your command in a PTY, parses `timestamp - LEVEL - message` log
lines, pulls embedded JSON out of each message, and renders everything as
a scrolling list of entries you can expand into proper tables — no more
squinting at a wall of raw JSON.

## Features

- **Auto-detected JSON** anywhere in a log line becomes a nested Rich
  table when expanded (dicts → key/value tables, lists of dicts → columnar
  tables), recursively.
- **Best-effort repair of logger-truncated JSON** — if your logger cuts a
  line off mid-object, prettier closes the open string/brackets, trims the
  dangling key, and still renders a clean table (flagged with `⚠`) instead
  of falling back to broken raw text.
- **12-hour timestamps** (`6:08:36 PM` instead of `18:08:36`).
- **Live search bar** docked at the top — type a phrase, non-matching
  entries hide instantly, including new lines as they stream in.
- **Full-log tee** — the raw, untouched output is always saved to
  `~/.cache/prettier/<timestamp>.log`.
- Tracebacks and other bare continuation lines attach to the entry above
  them instead of becoming noise.
- **Built-in AI assistant** — press `a`, ask a question in plain English
  ("any errors in the last few minutes?", "why did this monitor drop that
  message?"), and it sends the currently visible log lines to the OpenAI
  API and streams the answer back inline as a normal, expandable,
  searchable, copyable entry. Requires `OPENAI_API_KEY` to be set in your
  environment; optionally set `OPENAI_MODEL` (defaults to `gpt-4o-mini`).
  Only the log lines currently visible (respecting an active search
  filter) are sent, capped at the last 200 lines / ~12k characters.

## Keys

| Key | Action |
| --- | --- |
| `↑↓` / `j k` | move selection |
| `enter` / click | toggle selected entry between collapsed and full |
| `y` | copy the selected entry (message + pretty-printed JSON) to the clipboard |
| `c` | collapse all |
| `f` | toggle follow (auto-scroll) |
| `/` | focus the search bar |
| `a` | ask the AI assistant about the visible logs |
| `esc` | clear search / cancel the ask bar |
| `g` / `G` | jump to top / bottom |
| `ctrl+c` | send SIGINT to the wrapped process |
| `q` | quit (terminates the child if still running) |

## Install

`prettier` is a self-contained [uv](https://github.com/astral-sh/uv) script
(PEP 723 inline dependencies — just [Textual](https://github.com/Textualize/textual)).
Put it on your `PATH`:

```sh
curl -o ~/.local/bin/prettier https://raw.githubusercontent.com/ShabdVeyyakula/prettier-log-tui/master/prettier
chmod +x ~/.local/bin/prettier
```

Requires `uv` on your `PATH`. The first run downloads Textual once; after
that it starts instantly.

## Self-test

```sh
prettier --selftest
```

Runs the log-line parser and JSON-repair logic against a handful of
fixture cases with no TUI involved.
