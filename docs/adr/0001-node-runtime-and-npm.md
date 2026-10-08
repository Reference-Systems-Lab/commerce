# 1. Node.js LTS with a hardened npm

- **Status:** Accepted
- **Date:** 2026-10-08

## Context

Four repositories run JavaScript or TypeScript: the storefront (Nuxt), checkout and admin (Vue with
Vite), and the design system, whose packages and docs every frontend consumes. The platform's
engineering docs build with it too. The backend is Go and is not covered here.

They need one runtime and one package manager, so the design system's packages, its shared
toolchain and every lockfile behave the same everywhere. Nuxt needs Node 22 or newer and recommends
the active LTS; Vite needs 20.19+ or 22.12+. The supply chain is the main risk: npm 11 runs
dependency install scripts by default, and a malicious release is usually caught within days, not
hours. The research is commerce#3 (RQ-1, findings F1-F12), decided as DE-2.

## Decision

- **Runtime:** Node.js LTS in every JavaScript repository: Node 24 now, and Node 26 once it
  becomes LTS on 2026-10-28 (supported to 2029-04-30). `engines` and `engine-strict` enforce it.
- **Package manager:** the npm bundled with that Node, installed with `npm ci`, never `npm install`
  in CI or images. Every repository's `.npmrc` hardens it:
  - `strict-allow-scripts`, with an explicit `allowScripts` policy, so no dependency's install
    script runs unless it is named;
  - `min-release-age=7`, so a release is installable only after a week;
  - `allow-git=none` and `allow-remote=none`, so dependencies come only from the registry;
  - `engine-strict`.
- **Later:** move to npm 12, which blocks lifecycle scripts by default, once Dependabot lists it.

## Alternatives

- **pnpm 10** (the runner-up). It blocks install scripts and has a release-age window by default,
  but it adds Corepack or a separate install (Corepack leaves Node from version 25), and Dependabot
  supports it only to v10.
- **Bun.** No LTS release line, and Dependabot gives it no security updates. Its default trusted list
  ran esbuild's install script in testing.
- **Deno.** A second runtime beside the Node the frameworks expect, for no gain here.
- **npm with its defaults.** It ran esbuild's postinstall and resolved a release 12 hours old in
  testing.

## Consequences

- One runtime, one lockfile format and one set of Dependabot updates across the frontends and the
  design system.
- The protection depends on `.npmrc` being present wherever `npm ci` runs, including Docker build
  contexts. Each repository's scaffold adds it, and each Dockerfile copies it before installing.
- A new dependency that needs an install script is a deliberate `allowScripts` entry, reviewed like
  code.
- Fresh releases wait a week. Taking an urgent security fix sooner is a deliberate exception,
  reviewed in its pull request.
- Moving to Node 26 is one `engines` change and an image update per repository.
