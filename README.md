# Learn Docker

An interactive, hands-on Docker course: you build a real todo app (app + database) while learning containers, images, networking, and persistence along the way.

## What's inside

| Lesson | Topic |
|---|---|
| 1 | **Docker Fundamentals** — containers & images |
| 2 | **Building Images** — package your own app |
| 3 | **Running Applications** — networks, data, and multi-container apps |

Plus deeper dives once you've got the basics: **image layers**, **container isolation**, **networking**, and **Compose orchestration** (`architecture/`).

## Structure

- `SKILL.md` — the guided-tutoring script (how the course is taught, step by step)
- `lessons/` — the three core lessons
- `architecture/` — deep-dive explainers on how things work under the hood
- `checkpoints/` — short exercises to check your understanding after each topic
- `troubleshooting/` — common errors and how to verify your setup

## Who it's for

Anyone new to Docker who wants to learn by building something real instead of just reading command references.

## Browsing it locally

Clone the repo and run the included preview server (no dependencies beyond Python 3):

```bash
git clone git@github.com:romancobblepot/learn-docker.git
cd learn-docker
python3 preview.py
```

Then open `http://127.0.0.1:8000`. It lists this repo's folders and files, and renders any `.md` file as formatted markdown in the browser.
