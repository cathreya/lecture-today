# Lecture Today

An anti-feed tutor. Point it at a Telegram export and it turns the backlog you saved into a daily lecture: one item at a time, with context, chunks, and flashcards, instead of an infinite feed.

## Run

```bash
uv sync --extra anthropic
cp .env.example .env        # add your LLM key
.venv/bin/python app/app.py # http://localhost:8080
```

On first launch, open `http://localhost:8080`, point setup at `messages.html` (and the exported `files/` directory), and start.

## Test

```bash
uv run pytest tests -q
```

## What's here

- `app/` — Flask app, parser, classifier, curriculum, lecture generator, Anki export.
- `tests/` — pytest suite.
- `messages.html`, `files/`, `photos/`, `images/`, `css/`, `js/` — the Telegram export this instance was built from.

Progress lives in `lecture_today.db`, created on first run and gitignored.
