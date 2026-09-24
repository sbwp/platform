# platform
Shared code for the apps suite: npm packages, Terraform modules, reusable CI workflows, and local dev config. Apps pin one platform version and upgrade on their own schedule.

> **Status:** not scaffolded yet. The design lives in the suite README (`../README.md`), which is the source of truth until this repo has code.

## What's here (planned)

| Part | Purpose |
|---|---|
| `packages/runtime` | Backend library: wraps route handlers, validates requests with zod, builds the `User` from token claims, and provides `ctx.db`, `ctx.secrets`, and `ctx.log`. It never exposes AWS to app code. |
| `packages/build` | CLI: reads an app's routes manifest, bundles one Lambda per route with esbuild, writes the routes JSON, and runs the local dev environment (`dev`), including the local API server and fake sign-in. |
| `packages/web` | Frontend package: auth client (`useUser()`, `<RequireAuth>`), typed API client (`createClient<typeof routes>()`), test helpers, and the base Mantine theme. |
| `modules/` | Terraform modules apps use by pinned git tag, e.g. `app-api`, `static-site`, `app-auth`. |
| `.github/workflows/` | Reusable workflows for app PR checks and the build → dev → prod pipeline. |
| `local-dev/` | Caddy config and the fixed port pair for each app. |

The packages are published publicly to npmjs.com as `@sbwp/runtime`, `@sbwp/build`, and `@sbwp/web`. Each is a separate package so Lambda bundles never include React and the esbuild tooling never ships at runtime.

## Versioning and releases
- **One version for everything.** The packages, modules, and workflows share a single version and a single git tag (e.g. `v1.4.0`), because `build`'s output and the Terraform modules' input have to match.
- **release-please** keeps a release PR open that updates versions and changelogs. Merging it tags the release and publishes to npm. Versions are never edited by hand.
- **Trusted publishing:** each package names this repo's release workflow as its trusted publisher on npmjs.com, so CI publishes with GitHub's short-lived OIDC token and adds provenance. No npm token is stored. Each package's first version is published manually, since trusted publishing can only be configured once a package exists. Every package sets `publishConfig.access: "public"`.
- PR titles follow Conventional Commits. While the version is 0.x, breaking changes bump only the minor version.

## Security
This repo stores no secrets: no AWS credentials and no npm token. The reusable workflows run in each calling app's context with that app's own OIDC role.

## Development
Planned standard scripts: `pnpm test`, `pnpm lint`, `pnpm build`. Tool versions are pinned in `mise.toml`.
