# commerce

One commerce platform, built the way a production team would build it. It has a customer
storefront, a secure checkout, a restricted admin application, a backend that owns the business
domain, a shared design system, and the local platform that runs all of it. The repositories form
one system. They are not a collection of unrelated demos.

The aim is to show how a whole platform fits together: architecture, security, data integrity,
asynchronous processing, payments, recurring billing, failure recovery, CI/CD, observability and
developer experience. Every technology here is here because it solves a real problem in the system.

> **SANDBOX ONLY. NO REAL PAYMENTS ARE PROCESSED.** Payments run against the Authorize.Net sandbox.

## The product

The storefront is the product; everything else supports it. These are local development addresses,
each pointed at the machine through a hosts-file entry. A production deployment would use its own
domain.

| Address                      | Serves                                      |
| ---------------------------- | ------------------------------------------- |
| `rsl-commerce.test`          | Customer-facing storefront                  |
| `api.rsl-commerce.test`      | Backend API                                 |
| `admin.rsl-commerce.test`    | Restricted administration                   |
| `checkout.rsl-commerce.test` | Secure checkout and payment                 |
| `docs.rsl-commerce.test`     | Design system and engineering documentation |

## Repositories

| Repository                                         | Purpose                                                                                                  |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| [`platform`](platform)                             | Runs the whole ecosystem locally: routing, TLS, infrastructure, observability, integration tests         |
| [`backend`](backend)                               | The business domain and the API: catalog, orders, payments, billing, subscriptions, messaging, real-time |
| [`storefront`](storefront)                         | The public face of the business: marketing, discovery and buying                                         |
| [`admin`](admin)                                   | The restricted operations application                                                                    |
| [`checkout`](checkout)                             | The checkout application, kept separate because of its security profile                                  |
| [`design-system`](design-system)                   | Components, design tokens, documentation and shared frontend tooling, published as packages              |
| [`agent-workflow-tooling`](agent-workflow-tooling) | The agent skills, workflows and CLI used to build the rest                                               |

Each repository has its own lifecycle and its own CI/CD. They connect only through explicit
contracts: the API specification and the SDK generated from it, the design system's packages, the
configuration each application declares, and real-time events.

## Principles

- **Explicit boundaries.** Each application knows only what it needs to know.
- **Infrastructure abstraction.** Applications depend on services through configuration and
  contracts, so a local container and a managed production service are interchangeable.
- **Reliable messaging.** Messages survive failures, retry safely, dead-letter when they cannot be
  handled, and are processed idempotently.
- **Strong domain modeling.** Billing, payments, orders and subscriptions are explicit domains.
- **Security by design.** Security shapes the boundaries; it is not added at the end.
- **Observable systems.** Failures are visible and diagnosable.
- **Developer experience.** The system is easy to understand and easy to run.
- **Pragmatic complexity.** Sophisticated techniques appear only where they solve a genuine problem.

## What this deliberately avoids

Complexity has to earn its place. That rules out microservices for their own sake, Kubernetes for
the résumé, a second database or message broker without a reason, a repository for each piece of
infrastructure, dozens of tiny services, and event sourcing where a table will do.

## Local versus production

Locally, containers stand in for the database, cache, message broker, mail and search. In production
each would be a managed service. The applications cannot tell the difference, and that is the point.
The local setup does not claim to be production infrastructure. It runs the same application
architecture on convenient local services.

## Git hooks

Run `.githooks/setup` once after cloning. It turns on the committed hooks, which use
[git-secrets](https://github.com/awslabs/git-secrets#installing-git-secrets) to refuse any commit
that contains a secret.

## Status

Planning. The repositories are being scaffolded, and there is no code yet.

## License

[MIT](LICENSE)
