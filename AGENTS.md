# AGENTS.md

Instructions for agents working in this repo.

## What this is

Lecture Today: an anti-feed tutor. It reads a Telegram export (`messages.html` plus the downloaded media in `files/`), classifies the backlog, and serves one lecture at a time from a local Flask app. Progress is tracked in SQLite (`lecture_today.db`).

The code is the `app/` package plus the project-root tests in `tests/`. This repo was split out of the old `readlist` repository; Flashcap lives separately and is not used here.

## Layout

- `app/app.py` — Flask routes and the entry point.
- `app/parser.py` — parses `messages.html` and indexes `files/`.
- `app/classifier.py`, `app/curriculum.py`, `app/lecture_generator.py`, `app/metadata.py`, `app/anki.py` — the pipeline.
- `app/models.py` — SQLite schema and access. `DB_PATH` is `lecture_today.db` at the repo root.
- `app/templates/` — setup, lecture, graveyard, celebration pages.
- `tests/` — pytest suite. `conftest.py` puts the repo root and `app/` on `sys.path`.
- `messages.html`, `files/`, `photos/`, `images/`, `css/`, `js/` — the Telegram export and its media. `files/` is served by the `/files/<path>` route.

## Run and test

```bash
.venv/bin/python app/app.py            # http://localhost:8080
.venv/bin/python -m pytest tests -q
```

There is no deploy target. The app runs locally on the Mac.

## Environment

- `.env` at the repo root is ignored and never committed. It holds the LLM key.
- `app/config.py` reads `ANTHROPIC_API_KEY`, or `LLM_API_KEY` with `LLM_BASE_URL` and `LLM_MODEL` for any OpenAI-compatible provider.
- The database is `lecture_today.db` at the repo root and is gitignored. Do not commit it.

## Data flow

1. **Setup** parses `messages.html`, pairing each link with the media file named in the export.
2. **Classification** tags each item and decides whether it is lecture-worthy.
3. **Curriculum** picks the next item (or a lighter one when asked).
4. **Lecture generation** builds chunks, fetches metadata, and produces the page.
5. **Completion** records progress and queues Anki cards.

Re-running setup must not duplicate rows. Parser and classifier output should stay deterministic enough for tests.

## Testing

- `pytest tests` must pass. Tests use temporary databases and never call the network or the LLM.
- Add behavior tests at the route or function boundary.
- `tests/conftest.py` adds the repo root and `app/` to `sys.path`; `app/app.py` does the same so `python app/app.py` works without installing a package.

## Git

- `main` is trunk, and this repo has a single remote (`origin`). The old combined `readlist` repo is separate; do not push this work there.
- Do not commit `.env`, `lecture_today.db`, or anything under `.venv/`.

## Traps

1. `app/app.py` uses flat imports (`from parser import ...`) and inserts `app/` into `sys.path`. Keep new modules consistent with that.
2. Paths in `models.py` and `parser.py` are resolved from the repo root, so run commands from the repo root.
3. `files/` is 29 MB of exported PDFs and media. Do not add more large binaries without a reason.
4. The export has no link titles. Titles come from metadata fetching, with a URL-path fallback.
