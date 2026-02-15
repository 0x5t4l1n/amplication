# AGENTS Guide

## Project Overview
Amplication is an Nx-managed monorepo that powers the open-source Amplication platform. The workspace hosts multiple Node.js/TypeScript applications (NestJS back ends, React front ends, generators, gateways, services) plus shared libraries. Development relies on Nx task orchestration, Prisma for database access, PostgreSQL, GraphQL, Docker-based infrastructure, and a plugin-based code-generation pipeline.

Key entry points and docs:
- [`README.md`](./README.md) – high-level introduction, local setup, serve commands.
- [`package.json`](./package.json) – workspace scripts, engine versions (Node `^22.13.0`, npm `^9`).
- [`nx.json`](./nx.json) – Nx Cloud setup, target defaults, workspace layout (`packages/` for apps, `libs/` for shared code).
- [`scripts/setup.ts`](./scripts/setup.ts) – automated preparation for local development (`npm run setup:dev`).

## Repository Structure
The root contains tooling, documentation, and the Nx workspace folders:

```text
amplication/
├── packages/                 # All applications & services (Nx appsDir)
│   ├── amplication-server/   # NestJS GraphQL API backend
│   ├── amplication-client/   # React front end
│   ├── data-service-generator/ (DSG)
│   ├── amplication-plugin-api/
│   ├── amplication-build-manager/
│   ├── local-data-service-generator-controller/
│   ├── notification-service/
│   └── ...
├── libs/                     # Shared libraries (Nx libsDir)
│   ├── ui/design-system/
│   ├── schema-registry/
│   ├── util/
│   └── ...
├── docs/                     # Architecture diagrams and Draw.io sources
├── ee/                       # Enterprise-specific packages
├── scripts/                  # Automation (e.g., setup.ts, coverage merger)
├── .github/workflows/        # CI/CD pipelines (see `ci.yml`)
├── tutorials/                # End-to-end tutorial assets
├── README.md / CONTRIBUTING.md / CODE_OF_CONDUCT.md
└── nx.json / package.json / jest.config.ts / docker-compose.dev.yml
```

## Development Guidelines
1. **Environment prerequisites** (see `README.md` "Running Amplication"):
   - Install TypeScript globally: `npm install -g typescript`.
   - Use Node/NPM versions defined under `engines` in `package.json`.
   - Ensure Docker Desktop/Engine is running for local infra.
2. **Bootstrap the workspace**:
   ```bash
   npm install
   npm run setup:dev           # runs scripts/setup.ts to validate versions, generate Prisma clients & GraphQL schema, and build packages
   ```
3. **Start infrastructure** using Docker compose file + env var file:
   ```bash
   npm run docker:dev          # foreground logs
   npm run docker:dev -- -d    # detached background mode
   ```
4. **Apply Prisma migrations** across services:
   ```bash
   npm run db:migrate:deploy   # delegates to Nx run-many db:migrate:deploy
   ```
5. **Serve core applications** (defined in `package.json`):
   ```bash
   npm run serve:server        # npx nx serve amplication-server
   npm run serve:client        # npx nx serve amplication-client
   npm run serve:dsg           # runs amplication-build-manager & local DSG controller
   npm run serve:git           # git-sync manager
   npm run serve:plugins       # amplication-plugin-api
   npm run serve:storage       # amplication-storage-gateway
   npm run serve:notification  # notification-service
   ```
6. **Stop/cleanup infra** when needed: `npm run docker:dev:cleanup`.
7. **Use Nx for targeted work** (examples): `npx nx graph`, `npx nx run <project>:lint`, `npx nx affected:test --base=master`.

## Code & Architecture Patterns
- **Nx Workspace Layout**: `packages/` holds apps/services, `libs/` holds shared packages (configured via `workspaceLayout` in `nx.json`). Each project has its own `project.json`, `tsconfig`, and optionally `jest.config.ts`.
- **Target Defaults** (`nx.json`):
  - `serve`, `build`, and `test` depend on generated Prisma clients and upstream `prebuild` steps, ensuring schema consistency.
  - `lint` and `test` read shared configuration inputs (ESLint config, `jest.preset.js`).
- **Setup Automation**: `scripts/setup.ts` enforces Node/npm versions and executes Nx targets (`db:prisma:generate`, `graphql:schema:generate`, `build`).
- **Environment Management**: `.env.docker-compose` and `docker-compose.dev.yml` define local services (PostgreSQL, Kafka, etc.); service-specific READMEs enumerate required env vars (e.g., `packages/amplication-server/README.md`).
- **Shared Libraries**: `libs/ui/design-system` includes Storybook stories and E2E guidance; other libs (schema, util) provide cross-cutting concerns reused by apps.
- **Plugin & Generator Workflow**: `packages/data-service-generator` describes how to generate example input JSON, attach plugins, and run local code generation.

## Quality & Validation Standards
- **Formatting & Linting**:
  - `npm run format:check` / `format:write` call `npx nx format:* --all`.
  - `npx nx run <project>:lint` for per-project linting; workspace lint via `npx nx workspace-lint`.
  - Husky + lint-staged (`package.json`) format staged files pre-commit.
- **Testing**:
  - `npm run test:ci` executes `nx run-many --target=test --all --parallel --coverage`, merges coverage (`scripts/coverageMerger.js`), and generates reports via `lcov-viewer`.
  - Individual targets: `npx nx test amplication-server`, `npx nx e2e data-service-generator`, etc.
- **Builds**:
  - `npm run build` (`nx run-many --target=build`) followed by optional `postbuild` targets.
- **CI Pipeline** (`.github/workflows/ci.yml`):
  - Triggered on PRs, pushes to `master`/`next`, and manual dispatch.
  - Steps: checkout, `nrwl/nx-set-shas`, install deps (`npm ci`), workspace lint, format check, Nx lint/build/postbuild, and Nx tests with coverage. Additional workflows (`nx.template.yml`, release templates) run after CI on main branches.
- **Caching**: Nx Cloud caching is enabled with access token configured in `nx.json` and CI environment variables.

## Critical Rules & Constraints
- Respect Node/npm engine requirements (`package.json` `engines`). `scripts/setup.ts` will fail if versions mismatch.
- Execute Nx targets via `npx nx ...` or the npm scripts—they encapsulate required dependency steps (Prisma generation, prebuild) defined in `nx.json`.
- Sensitive env values (JWT secrets, OAuth keys) must come from `.env`/secret managers; never commit them. Service READMEs (e.g., `packages/amplication-server/README.md`) specify required vars.
- Use Docker compose config and `.env.docker-compose` for local infra parity. Clean up with `npm run docker:dev:cleanup` before switching branches to avoid stale containers.
- Nx `affected` commands compare against `master` (`affected.defaultBase`), so ensure the base branch is up-to-date when running affected builds/tests locally.

## Common Tasks for Agents
| Task | Command(s) | Notes |
| --- | --- | --- |
| Install & prepare workspace | `npm install && npm run setup:dev` | Validates engines, generates Prisma clients & GraphQL schema, builds packages. |
| Start infra services | `npm run docker:dev` or `npm run docker:dev -- -d` | Uses `docker-compose.dev.yml` + `.env.docker-compose`. |
| Apply DB migrations | `npm run db:migrate:deploy` | Runs `nx run-many --target=db:migrate:deploy` for all relevant apps. |
| Run core services | `npm run serve:server`, `npm run serve:client`, `npm run serve:dsg`, etc. | Ensure server + client are both running for full UI experience. |
| Generate local service code | `npx nx generate-example-input-json data-service-generator`, `npx nx generate-local-code data-service-generator` | Refer to `packages/data-service-generator/README.md` for plugin pointers. |
| Debug DSG | `npm run debug:dsg` | Launches generator in debug mode to inspect plugins. |
| Run storybook/UI tests | See `libs/ui/design-system/README.md` and related Nx targets (`nx g @nx/react:component-story`, `nx e2e ui-design-system-e2e`). |
| Clean Nx cache | `npm run clean` (`nx clear-cache`) | Useful when changing branches or updating dependencies. |

## Reference Examples
- **Server (packages/amplication-server/)**: NestJS GraphQL API. Useful targets: `npx nx serve amplication-server`, `nx test`, `nx lint`, `nx build`. README lists required env vars (PostgreSQL URL, Kafka, JWT secrets, etc.).
- **Client (packages/amplication-client/)**: React UI bootstrapped with CRA, served via `npx nx serve amplication-client` (http://localhost:3001). Includes standard Nx test/lint/build targets.
- **Data Service Generator (packages/data-service-generator/)**: Generates source code & integrates plugins. Commands for example input generation, local code generation, and DSG E2E tests (`npx nx e2e data-service-generator`).
- **Notification Service (packages/notification-service/)**: Event-driven notification runner. Served via `npm run serve:notification` and adheres to contributing guidelines in the root `CONTRIBUTING.md`.
- **UI Design System (libs/ui/design-system/)**: Shared component library with Storybook; README explains generating stories and ensuring corresponding e2e coverage.

## Additional Resources
- [README](./README.md) – setup, scripts, tutorials.
- [CONTRIBUTING](./CONTRIBUTING.md) – contribution process, coding standards.
- [CODE_OF_CONDUCT](./CODE_OF_CONDUCT.md) – community guidelines.
- [Docs directory](./docs/) – architecture diagrams (`system-topolgy.drawio`, ERDs, build process visuals).
- [GitHub Actions workflows](./.github/workflows/ci.yml) – CI/CD logic, Nx caching, release orchestration.
- [Tutorials](./tutorials/) – guided samples referenced in README.
- [Discord](https://amplication.com/discord) and [docs site](https://docs.amplication.com) for real-time help and platform documentation.
