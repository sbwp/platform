# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

The suite-wide `../CLAUDE.md` and the design in `../README.md` also apply. This repo is not scaffolded yet, so there are no commands. Add them here once they exist.

## What this repo is
The `platform` repo: the `runtime`, `build`, and `web` npm packages, Terraform modules, reusable GitHub Actions workflows, and `local-dev/` config. Nothing here is deployed. Releasing means publishing, and apps adopt new versions through their own PRs.

## Rules
- **Build only what an app needs.** Start with what Budget needs, and add more only when a second app needs the same thing.
- **This repo stores no secrets.** No AWS credentials and no npm token. Publishing uses npm trusted publishing (GitHub OIDC) from the release workflow. Never add an `NPM_TOKEN` secret as a workaround.
- **Keep the packages separate.** `runtime` must never depend on React or esbuild, and `web` must never depend on backend code.
- **One shared version.** Changes to `build`'s routes JSON output and the Terraform modules that read it must ship in the same release. A breaking change to anything apps depend on (manifest API, handler contract, routes JSON, module inputs, Node major version) needs a `feat!:` PR title.
- **Types are part of the API.** The generic types behind `defineApp`, the handler contract, and `createClient` need type-level tests (`expectTypeOf`) alongside the runtime tests.
- **Security-sensitive code:**
    - The wrapper fails closed: 401 when a non-public route receives no claims, 403 when required scopes are missing.
    - JWT verification exists only in the local dev server, never in `runtime`.
    - The fake sign-in issuer must never be reachable from deployed code.
- **The SSM parameter names read by the modules are a contract with `platform-infra`.** Changing one takes coordinated PRs: infra first, then the module.
- **The platform owns the Node major version.** The esbuild target, the Lambda runtime, and `engines.node` must agree.
