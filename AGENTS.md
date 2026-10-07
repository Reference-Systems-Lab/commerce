# Agent instructions

This is the commerce platform, one system of the Reference Systems Lab organization, split across
several repositories. The [README](README.md) has the big picture. Each project folder has its own
`AGENTS.md`, which narrows this file and wins when the two conflict.

## Boundaries

- Each folder is its own repository with its own history and CI. Keep a change inside one repository
  unless the task is explicitly cross-repository.
- Repositories talk to each other only through contracts:
  - the backend's API specification and the SDK generated from it;
  - the design system's published packages;
  - the configuration each application declares and the platform provides;
  - browser-safe real-time events.
- Never reach around a contract. No frontend reads the database, and no application assumes how the
  platform provisions its infrastructure.

## Rules for every repository

- The backend is authoritative for authentication, authorization, prices and totals. Frontends
  display these; they do not decide them.
- Frontends never talk to PostgreSQL, Redis or RabbitMQ.
- Payments are sandbox only. Never store raw card numbers or CVVs; the payment gateway tokenizes
  them.
- No real secrets anywhere. Keep sandbox credentials clearly separate from anything production.
- Retries and duplicates are normal. Anything that touches money, orders or messages must be
  idempotent.
- Every technology must solve a real problem in the system. Do not add one because it looks
  impressive.

## How we work

- Planning stays local until each repository's initial commit. Do not open issues, push or publish
  anything until asked.
- Do what was asked, at the depth asked. Raise bigger decisions instead of making them in passing.
- Record architecture decisions as ADRs in the repository they affect, covering the context, the
  decision, the alternatives and the consequences.
