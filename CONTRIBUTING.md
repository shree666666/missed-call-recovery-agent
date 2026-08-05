# Contributing — How this project will be developed in Part B

This is a solo Master's capstone. This document sets out how the work will run in Part B so that a sponsor, mentor or marker can see the working method, not just the output.

## Method

- **Lean Startup (build–measure–learn)** with design-thinking discovery, and a **light Scrum cadence** adapted for solo delivery (see [`docs/methodology.md`](docs/methodology.md)).
- **Two-week increments.** Each ends with a written self-retrospective and a fortnightly review with the industry mentor.
- **Definition of done** for a card: code/design complete, compliance check passed where messaging is touched, board updated, and (where relevant) a note in `/docs`.

## Workflow

1. Every unit of work is a **board card** with an owner, an estimate (hours), and a **milestone label** (`Part B Week N`) plus a **category label** (`build` / `docs` / `research` / `ethics`).
2. Work happens on a branch; merges go through a pull request using the template in `.github/`.
3. **Scope is locked at the A3 charter.** Anything beyond it requires a written change request with mentor sign-off (this is a named risk mitigation, R7).

## Compliance gate (non-negotiable)

No live message is sent until the **Spam Act / APP compliance review** passes (milestone M2). Any change to outbound messaging re-triggers the checklist. **Caller data never enters version control** — see `.gitignore`.

## Branch naming

`feature/<short-name>`, `fix/<short-name>`, `docs/<short-name>`.

## Contact

Shreekrishna Gyawali (SHEA25124). Questions: open an issue or contact via the details in the README.
