# AGENTS Guide

This document summarizes the conventions and expectations for contributors and automation agents working inside the Amplication monorepo.

## 1. Project Overview

Amplication is an Nx-managed monorepo that delivers AI-assisted tooling for generating production-ready backend services. It combines Node.js, TypeScript, Nx task orchestration, NestJS APIs, React frontends, Prisma-based data access, GraphQL APIs, Jest testing, Docker packaging, and companion services that integrate with AWS, GCP, and other infrastructure providers. The repository mixes open-source components (Apache 2.0) with Enterprise Edition packages housed under `ee/`.

## 2. Repository Structure

| Path | Purpose |
| --- | --- |
| `packages/` | Primary applications and services (e.g., `amplication-server`, `amplication-client`, `amplication-build-manager`, `gpt-gateway`, generators, notification service). Each package typically defines its own Nx targets and README.
| `libs/` | Shared libraries such as `schema-registry`, `ui`, and `util` that encapsulate reusable NestJS modules, React components, and utility helpers.
| `ee/` | Enterprise Edition features with their own licensing (see `ee/LICENSE`).
| `.github/workflows/` | CI/CD templates and workflow docs (`README.md`) describing lint/test/build/release pipelines and Nx-driven rules for container packaging and deployments.
| `docs/`, `tutorials/` | End-user documentation, product guides, and sample walkthroughs.
| `scripts/` | Repository-wide tooling (setup scripts, automation utilities).
| Root files (`README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `LICENSE`, `package.json`, `nx.json`) | Entry-point documentation, governance, and workspace configuration.

## 3. Development Guidelines

- **Environment**: Use the Node/npm versions declared in `package.json` and install TypeScript globally if needed. Docker must be running for infrastructure dependencies.
- **Bootstrap**: After cloning, run `npm install` then `npm run setup:dev` to prepare Nx projects and shared tooling. Local infrastructure launches via `npm run docker:dev` (foreground or background with `-d`).
- **Database**: Apply Prisma migrations with `npm run db:migrate:deploy`. Data-access code is generated and validated through Prisma schema files embedded in relevant packages.
- **Serving apps**: Start services via scripts such as `npm run serve:server`, `serve:client`, `serve:dsg`, `serve:git`, and `serve:plugins`. Running the React client requires the server (and sometimes specialized services) to be active simultaneously.
- **Coding standards**: Follow rules in `CONTRIBUTING.md` and respect the code of conduct. Prefer TypeScript strictness, leverage Nx dependency graphs, and keep features encapsulated within libraries when possible.

## 4. Code Patterns to Follow

- **NestJS services** (`packages/amplication-server`, `packages/gpt-gateway`, etc.) rely on modular architecture, GraphQL resolvers, and Prisma repositories. New modules should expose DTOs, guards, and services consistent with existing patterns.
- **React client** (`packages/amplication-client`) uses modern React (hooks, context providers) with design assets from `libs/ui`. Maintain separation between presentation and data-fetching layers, and co-locate tests with components.
- **Generators** (`packages/data-service-generator*`) compose code via templates and Nx executors; respect existing plugin contracts and CLI arguments.
- **Shared libs** such as `libs/schema-registry` demonstrate how to package reusable NestJS features or utilities; use them as references when extracting logic.
- **Infrastructure**: Nx targets often include `build`, `test`, `lint`, `package:container`, `deploy:container`, and custom `db:*` commands. Ensure new projects define the appropriate targets in `project.json`.

## 5. Quality Standards

- **Linting**: Run `npx nx lint <project>` or `npm run lint` to respect workspace ESLint rules (.eslintrc.json/.eslintignore).
- **Testing**: Jest is the standard test runner. Execute `npx nx test <project>` or `npm run test` for affected apps. Keep tests deterministic and co-located.
- **Builds**: Use `npx nx build <project>` or package-specific scripts before submitting changes; CI re-builds affected projects automatically.
- **Database tasks**: `npm run db:migrate:deploy` and related `db:*` Nx targets (e.g., `db:migrate:dev`, `db:studio`) must succeed locally prior to opening a PR.
- **CI compliance**: `.github/workflows/README.md` documents how the `continuous integration` workflow lint/tests/builds everything and how release templates depend on `package:container` / `deploy:container` targets. Failing to define these targets correctly will break deployments.

## 6. Critical Rules

1. Do not modify Enterprise Edition code (`ee/`) without confirming licensing requirements.
2. PRs touching workflow templates must target `master` to take effect (see `.github/workflows/README.md`).
3. Keep secrets and credentials out of source control; prefer GitHub Actions secrets or environment files excluded via `.gitignore`.
4. Run the full Nx target suite (lint/test/build) for affected projects before requesting review.
5. Follow contribution steps in `CONTRIBUTING.md`, including issue templates and commit hygiene.
6. Ensure client/server versions remain compatible—when updating APIs, update corresponding React hooks/components.

## 7. Common Tasks

| Task | Command / Notes |
| --- | --- |
| Install dependencies | `npm install` |
| One-time workspace setup | `npm run setup:dev` |
| Start infrastructure (logs) | `npm run docker:dev` |
| Start infrastructure (detached) | `npm run docker:dev -- -d` |
| Run Prisma migrations | `npm run db:migrate:deploy` |
| Serve server | `npm run serve:server` |
| Serve client | `npm run serve:client` (requires server running) |
| Serve generators | `npm run serve:dsg` and related scripts |
| Run Nx lint | `npx nx lint <project>` |
| Run Nx tests | `npx nx test <project>` |
| Run Nx build | `npx nx build <project>` |
| Package container | `npx nx run <project>:package:container` (project must define target) |
| Deploy container/static | `npx nx run <project>:deploy:container` or `deploy:static` |

## 8. Reference Examples

- **`libs/schema-registry/`** – Minimal shared NestJS module that illustrates how to expose reusable schemas and dependency-injection tokens.
- **`packages/amplication-server/`** – Core NestJS backend with GraphQL resolvers, Prisma integration, and Nx targets for build/test/deploy; follow its module layout when adding backend capabilities.
- **`packages/amplication-client/`** – React frontend consuming the generated APIs using hooks/components sourced from `libs/ui`.
- **`packages/amplication-build-manager/`** – Service that coordinates builds and release workflows; showcases background job processing and container packaging targets.
- **`packages/gpt-gateway/`** – AI gateway service illustrating how Amplication integrates with external AI providers and how to structure standalone services inside the monorepo.

## 9. Additional Resources

- **Root `README.md`** – Product overview, setup instructions, and key scripts.
- **`CONTRIBUTING.md`** – Contribution workflow, coding standards, and governance.
- **`CODE_OF_CONDUCT.md`** – Community expectations.
- **`.github/workflows/README.md`** – Detailed CI/CD and Nx target requirements.
- **Online docs** – https://docs.amplication.com and tutorials under `tutorials/` for practical guides.
- **Community links** – Discord, blog, and YouTube resources highlighted in `README.md`.

Adhering to these guidelines ensures changes align with existing architecture, pass CI checks, and remain consistent with Amplication’s engineering standards.