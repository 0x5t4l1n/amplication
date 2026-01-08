# 🤖 Amplication Agents Guide

## 📋 Title & Project Overview
- Amplication is a large Nx-managed TypeScript monorepo that hosts multiple deployable services, generators, and shared libraries in a single workspace (see `README.md`).
- Core stack: Node.js 22.x and npm 9.x (per `package.json` engines), Nx 16, TypeScript, NestJS, React, Prisma with PostgreSQL, GraphQL, Jest, Docker Compose, Kafka/Redis, Husky + lint-staged, and Nx Cloud caching.
- The repository powers live templates, service catalog automation, and AI-assisted generation across backend and frontend domains.

## 🗂️ Repository Layout
```
./
├── packages/            # Deployable apps & services (NestJS backends, React client, generators)
├── libs/                # Shared utilities, schema helpers, UI design system
├── ee/                  # Enterprise-only services (e.g., git-sync-manager) under a separate license
├── docs/                # Product & architecture documentation assets
├── tutorials/           # Step-by-step guides/blog-style walkthroughs
├── scripts/             # Workspace automation (setup, coverage, publishing)
├── .github/             # GitHub Actions, templates, community health files
└── Root configs         # `project.json`, `tsconfig.*`, `jest` configs, Dockerfiles per package
```
- Common project structure uses Nx `project.json`, `tsconfig.*`, Jest configs, Dockerfiles, and `.env` samples per package (`analyze-repo` summary, verified in package directories).
- TypeScript path aliases in `tsconfig.base.json` expose shared modules such as `@amplication/ui/design-system`, `@amplication/util/*`, and `@amplication/prisma-db`, enabling consistent imports across packages.

## 🛠️ Tooling & Environment Setup
### Prerequisites
- Install **Node.js** (must satisfy `^22.13.0`) and **npm** (`^9.0.0`) per `package.json` `engines`.
- Ensure **Docker** and **Git** are available (`README.md`).

### Local Workflow (from `README.md`)
1. **Clone & install**
   ```bash
git clone https://github.com/amplication/amplication.git
cd amplication
npm install
   ```
2. **Bootstrap workspace**
   ```bash
npm run setup:dev
   ```
3. **Provision infrastructure** (foreground logs or detached):
   ```bash
npm run docker:dev
# or
npm run docker:dev -- -d
   ```
4. **Apply database migrations**
   ```bash
npm run db:migrate:deploy
   ```
5. **Start services** using the `serve:*` scripts (`package.json`), e.g. `npm run serve:server`, `npm run serve:client`, `npm run serve:dsg`, `npm run serve:git`, `npm run serve:plugins`, `npm run serve:storage`, `npm run serve:notification`.

> 💡 **Codespaces**: The README highlights a ready-to-use GitHub Codespaces option under the "Running Amplication" section for agents who need a managed environment.

### Infrastructure Details (`docker-compose.dev.yml`)
- Postgres 12 (`db`) with credentials from `.env.docker-compose`.
- Redis cache (`redis`).
- Kafka stack: Zookeeper, Kafka broker with dual listeners, and Kafka UI (`kafka-ui`).
- Shared volumes for Postgres and Redis data persistence.

## ⚙️ Automation & Scripts
- `npm run setup:dev` executes `scripts/setup.ts`, which:
  - Validates Node/npm versions against `package.json`.
  - Optionally clears the Nx cache (`npx nx clear-cache` when invoked with `--clean`).
  - Runs `npx nx run-many --target db:prisma:generate` to ensure Prisma clients are fresh.
  - Runs `npx nx run-many --target graphql:schema:generate` to produce schemas before builds.
  - Builds all packages via `npx nx run-many --target build`, surfacing progress with `ora` spinners.
- Nx target defaults in `nx.json` enforce dependencies: e.g., `build` depends on `prebuild`, local and upstream builds, and Prisma generation; `serve` and `test` similarly depend on Prisma generation and prebuild hooks. `package:container` targets chain `prebuild → build → postbuild`.
- Selected npm scripts (`package.json`):
  - `setup:dev`, `docker:dev`, `docker:dev:cleanup`, `db:migrate:deploy`.
  - `serve:*` scripts for individual services (server, client, data-service-generator, git-sync-manager, plugin API, storage gateway, notification service).
  - `build`, `test:ci`, `format:check`, `format:write`, `clean`, `precommit`.
  - `prepare` installs Husky hooks.

## 🧱 Development & Code Patterns
- **Backend** (`packages/amplication-server`, `packages/notification-service`, etc.) is built with NestJS, GraphQL, Prisma/PostgreSQL, Kafka integrations, and shared logging/tracing utilities (`CONTRIBUTING.md`, `packages/*/project.json`).
- **Frontend** (`packages/amplication-client`) is a React + Apollo Client app backed by the shared `libs/ui/design-system` components.
- **Shared libraries** in `libs/` cover schema registries, UI, logging, Kafka, billing roles, code-gen helpers, etc., accessible via path aliases (`tsconfig.base.json`).
- **Testing** uses Nx-managed Jest projects: `jest.config.ts` delegates to `getJestProjects()` so every package/library contributes its own configuration.
- **Build & deploy** pipelines leverage Nx webpack/node builds per project (`packages/amplication-server/project.json`), container packaging via `package:container`, and workspace-wide `nx run-many` operations.
- **Local infra** is standardized through `docker-compose.dev.yml`, ensuring Postgres, Redis, and Kafka are consistent across services.
- **Enterprise** code lives under `ee/` with the Amplication Enterprise Edition license (see `README.md` License section and `ee/LICENSE`).

## ✅ Quality & Compliance
- Husky pre-commit hook (`.husky/pre-commit`) blocks commits directly on `master`, `main`, or `next` and then runs `npm run precommit`.
- `npm run precommit` maps to `lint-staged --relative`, which formats staged files via `nx format:write --files <paths>` (`package.json` `lint-staged`).
- Commit messages follow `<type>(<package>): <subject>` and branches should use prefixes like `feat/{ISSUE}-{slug}` or `fix/{ISSUE}-{slug}` (`CONTRIBUTING.md`). Pull requests must target the `next` branch and follow the same title convention.
- CI status is published via `.github/workflows/ci.yml` (badge in `README.md`), and Nx Cloud is configured as the default task runner with cacheable operations (`nx.json`).

## 🧰 Common Tasks for Agents
- **Run a specific service**
  ```bash
npx nx serve amplication-server
npx nx serve amplication-client
  ```
- **Run unit tests**
  ```bash
npx nx test amplication-server
npx nx test amplication-client
  ```
- **Generate GraphQL schema/models** (per `packages/amplication-server/project.json`)
  ```bash
npx nx run amplication-server:graphql:schema:generate
npx nx run amplication-server:graphql:models:generate
  ```
- **Regenerate all Prisma clients** (as in `scripts/setup.ts`)
  ```bash
npx nx run-many --target db:prisma:generate --output-style stream
  ```
- **Bring up local infrastructure**
  ```bash
npm run docker:dev        # foreground
npm run docker:dev -- -d  # detached
  ```
- **Apply workspace migrations**
  ```bash
npm run db:migrate:deploy
  ```
- **Format staged files or specific paths**
  ```bash
npx nx format:write --files libs/ui/design-system/src/lib/**
  ```
- **Re-run full setup with cache clear**
  ```bash
DEBUG=true npm run setup:dev -- --clean
  ```

## 🧭 Reference Examples
- **Simple packages**: `packages/amplication-cli` (minimal Node CLI) and `libs/util/logging` for focused utilities.
- **Complex services**: `packages/amplication-server` (NestJS GraphQL API), `packages/amplication-client` (React UI), `packages/gpt-gateway` (AI service built on NestJS + OpenAI integrations).
- **Documentation**: `tutorials/deploy-to-azure.md` demonstrates the writing style used for deployment guides.

## 📚 Additional Resources
- [Root README](README.md) – onboarding, prerequisites, and serve scripts.
- [CONTRIBUTING guide](CONTRIBUTING.md) – branch, commit, and PR conventions.
- [Docs directory](docs/) – product and architecture references.
- [Tutorials](tutorials/) – hands-on guides such as `tutorials/deploy-to-azure.md`.
- [docker-compose.dev.yml](docker-compose.dev.yml) & [.env.docker-compose](.env.docker-compose) – infrastructure definitions and environment variables.
- [scripts/setup.ts](scripts/setup.ts) – authoritative automation flow for workspace preparation.
