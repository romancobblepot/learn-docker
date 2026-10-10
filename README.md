# Learn Docker

I wanted to learn Docker fundamentals in a concise, well-structured way, so I put together this interactive course and learned from it. It's built around a real project — a todo app with a database — so the concepts (containers, images, networking, persistence) stick because you actually use them, not just read about them.

## What's inside

| Lesson | Topic |
|---|---|
| 1 | **Docker Fundamentals** — containers & images |
| 2 | **Building Images** — package your own app |
| 3 | **Running Applications** — networks, data, and multi-container apps |

Plus deeper dives I added once I had the basics down: **image layers**, **container isolation**, **networking**, and **Compose orchestration** (`architecture/`).

## Structure

- `SKILL.md` — the guided-tutoring script (how the course is taught, step by step)
- `lessons/` — the three core lessons
- `architecture/` — deep-dive explainers on how things work under the hood
- `checkpoints/` — short exercises to check your understanding after each topic
- `troubleshooting/` — common errors and how to verify your setup

## Who it's for

Anyone new to Docker who'd rather learn by building something real than by reading command references.

## Browsing it locally

Clone the repo and run the included preview server (no dependencies beyond Python 3):

```bash
git clone git@github.com:romancobblepot/learn-docker.git
cd learn-docker
python3 preview.py
```

Then open `http://127.0.0.1:8000`. It lists this repo's folders and files, and renders any `.md` file as formatted markdown in the browser.
