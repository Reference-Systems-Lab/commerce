# 2. One container baseline for every service

- **Status:** Accepted
- **Date:** 2026-10-08

## Context

Every application ships as a container: the Go backend, the Nuxt storefront, and the checkout and
admin single-page apps. The platform runs them beside its own infrastructure containers. Without a
shared baseline each repository would choose its own base image, user and hardening, and the weakest
choice would set the system's security. The platform runs the images CI publishes, so the baseline is
set once, in each image.

Node.js "was not designed to run as PID 1": it doesn't forward signals or reap child processes. A
shell-form command doesn't pass signals either. The research is commerce#3 (RQ-4, findings F44-F52),
decided as DE-5.

## Decision

- **Images.** Multi-stage builds on Debian trixie-slim, with a final stage on distroless `nonroot`:
  `nodejs` for the storefront, `static` for the Go backend. The static single-page apps (checkout and
  admin) are served by `nginx-unprivileged` on its `alpine-slim` line. Every base image is pinned by
  digest.
- **Every service, without exception** (the platform's infrastructure included, as in its ADR 0001):
  - a numeric non-root user, set with `USER` in the image;
  - an exec-form `CMD`, and a `HEALTHCHECK` that runs without a shell;
  - `init: true`, so a real init process forwards signals and reaps children;
  - a read-only root filesystem, with a `/tmp` tmpfs for what must be written;
  - `cap_drop: ALL` and `no-new-privileges`.
- **Fallback:** Docker Hardened Images, if distroless stops fitting.

## Alternatives

- **The official `node:*-slim` images.** About 25 MB larger than distroless, and they ship a shell
  and a package manager the service never needs.
- **Alpine.** musl instead of the glibc the build stage uses, and the Go image's Alpine variant is not
  officially supported. It stays only for nginx-unprivileged, which serves files and runs no
  application code.
- **Chainguard.** The free tier offers only `:latest`; versioned tags are paid, and `:latest` isn't
  an LTS.
- **Docker Hardened Images** as the default. Free, with SBOMs and SLSA provenance, but they are
  pulled from `dhi.io` after a login, which every CI job and developer would need.

## Consequences

- An attacker who gets code running in a service finds no shell, no package manager, no
  capabilities and no writable filesystem outside `/tmp` and its volumes.
- `docker stop` is fast and clean: with `init`, a distroless Node container stopped in 0.53 s with
  exit 143; without it, 6.87 s and a kill (137).
- Debugging a distroless container needs its `:debug` tag, which has a shell, used locally only.
- Each service writes only where it declares a tmpfs or a volume, so a new write path shows up as a
  read-only error during development rather than in production.
- Health checks run as exec-form commands the image already contains, because there is no shell or
  curl to call. Each application's ADR says how its own check works.
