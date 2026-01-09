# AGENTS.md

## Project Overview
Amplication is an Nx-managed TypeScript monorepo that bundles every deployable surface—backend services, the React client, CLIs, gateways, and automation—into a single workspace. The platform automates service scaffolding with Node.js, NestJS, Prisma, GraphQL, and React while relying on Nx for orchestration, caching, and target coordination. Local development mirrors production by running Docker Compose–based infrastructure (`docker-compose.dev.yml`) alongside the core apps.

## Repository Structure
- `packages/` – Application packages with their own `project.json` targets. Key examples include `amplication-server` (NestJS GraphQL API), `amplication-client` (React/Apollo UI), `amplication-cli`, `data-service-generator`, `gpt-gateway`, `notification-service`, and storage/gateway utilities. Each subdirectory contains a README plus Nx targets for serve/build/test/lint.
- `libs/` – Shared modules (UI widgets, schema helpers, utilities) imported by multiple packages to prevent duplication.
- `ee/` – Enterprise-only code licensed under `ee/LICENSE`. Treat as read-only unless explicitly authorized.
- `docs/` & `tutorials/` – Published documentation and hands-on guides referenced by the main README.
- `.github/workflows/` – CI/CD automations (e.g., `ci.yml`) enforcing the Nx targets in pipelines.
- `scripts/` – Workspace-level helpers such as `coverageMerger.js` (aggregates Jest coverage), `publish.mjs`, and setup tooling.
- Root config: `nx.json`, `tsconfig.base.json`, `jest.config.ts`, `docker-compose.dev.yml`, `.eslintrc.json`, and other Nx/TypeScript/Jest linters establish global defaults.

## Development Guidelines
1. **Install prerequisites** – Node.js (see `package.json` `engines`), npm, Docker, Git, plus `npm install -g typescript` as called out in `README.md`.
2. **Clone & install**
   ```bash
   git clone https://github.com/amplication/amplication.git
   cd amplication
   npm install
   ```
3. **Bootstrap the workspace** – Run `npm run setup:dev` once; it wires dependencies, builds packages, and syncs generators.
4. **Provision infra** – `npm run docker:dev` (foreground logs) or `npm run docker:dev -- -d` (detached) to start PostgreSQL, Kafka, and ancillary services defined in `docker-compose.dev.yml`.
5. **Database migrations** – Apply schema updates with `npm run db:migrate:deploy` before serving apps.
6. **Serve applications** – Use npm scripts or their Nx equivalents:
   ```bash
   npm run serve:server     # wraps npx nx serve amplication-server
   npm run serve:client     # React client (requires server running)
   npm run serve:dsg        # data-service-generator
   npm run serve:git        # git-sync-manager
   npm run serve:plugins    # plugin API
   ```
7. **Follow package-level targets** – Each `packages/<name>/project.json` exposes standard Nx commands (`serve`, `test`, `lint`, `build`). Example from `packages/amplication-server/README.md`:
   ```bash
   npx nx serve amplication-server
   npx nx test amplication-server
   npx nx lint amplication-server
   npx nx build amplication-server
   ```
8. **Tear down responsibly** – Stop Docker services when finished to avoid port conflicts and data drift.

## Code Patterns
- **Backend services** (`packages/amplication-server`, gateways, notification-service) are NestJS apps exposing GraphQL APIs backed by Prisma/PostgreSQL. Kafka topics, JWT secrets, and storage roots are configurable through environment variables listed in each package README.
- **Frontend** (`packages/amplication-client`) is a React + Apollo application consuming the GraphQL API. Styling preferences (SCSS) and ESLint defaults inherit from workspace generators (`nx.json` references `@nx/react`).
- **Generators & automation** (`data-service-generator`, `generator-blueprints`, `local-data-service-generator-controller`) rely on Nx targets to emit service templates, keeping org standards codified.
- **Nx target defaults** (from `nx.json`) enforce dependency ordering—e.g., `serve` depends on `db:prisma:generate` and `^prebuild`, `build` depends on `prebuild` and Prisma generation, and `test` depends on upstream `prebuild`. Respect these chains to prevent stale artifacts.

## Quality Standards
- **Linting** – Run `nx lint <project>` or the repo-wide script `npm run lint` before submitting changes. ESLint rules come from `.eslintrc.json` and each project’s overrides.
- **Formatting** – Use `nx format:write` / `nx format:check` so that Nx enforces consistent Prettier output.
- **Testing** – Execute `nx test <project>` for unit coverage and `nx test:e2e`/`nx e2e <project>` where defined. Target defaults ensure Prisma artifacts and prebuild steps run automatically.
- **Coverage reporting** – Collect package-level `.lcov` outputs and run `node scripts/coverageMerger.js` to merge results for CI parity.
- **Caching & CI parity** – Nx Cloud (`tasksRunnerOptions.default.runner`) caches `prebuild`, `build`, `test`, `lint`, etc. Avoid bypassing Nx commands so cache keys stay valid.

## Critical Rules
- **Do not modify `ee/`** without explicit enterprise approval; its license is separate and more restrictive.
- **Always run `npm run setup:dev` after pulling new dependencies** to keep generators in sync.
- **Keep Docker infra running while serving apps**; the client requires the server, which in turn expects PostgreSQL/Kafka from `docker-compose.dev.yml`.
- **Use Nx scripts (`npx nx ...` or existing npm wrappers)** instead of calling underlying toolchains directly; this preserves target dependencies, caching, and environment variables.
- **Protect secrets** – Never hardcode environment secrets; rely on `.env` files or secret managers referenced in package READMEs.
- **Respect branch base** – Nx’s affected graph assumes `master` as the default base; rebase frequently to reduce affected surface and flaky caches.

## Common Tasks
| Goal | Command(s) | Notes |
| --- | --- | --- |
| Spin up full stack locally | `npm run docker:dev` → `npm run serve:server` + `npm run serve:client` | Client depends on server + infra; start both for UI work.
| Run backend tests | `npx nx test amplication-server` | Prisma generation & prebuild run automatically via target defaults.
| Apply database migrations | `npm run db:migrate:deploy` | Required after schema edits or before first serve.
| Check lint + formatting | `npm run lint` and `nx format:check` | Run prior to PRs to satisfy CI.
| Merge coverage | `node scripts/coverageMerger.js` | Use after running multiple Jest targets locally.
| Generate data-service templates | `npm run serve:dsg` (or `npx nx serve data-service-generator`) | Requires docker infra plus server when integrating.

## Reference Examples
- **Backend blueprint** – `packages/amplication-server/README.md` documents GraphQL endpoints, environment variables, and Nx targets for the NestJS API.
- **Client workflow** – `packages/amplication-client/` (React/Apollo) follows the same Nx target pattern; inspect its `project.json` for exact scripts when adjusting UI logic.
- **Gateway services** – `packages/gpt-gateway/` and `packages/amplication-storage-gateway/` illustrate how to expose additional services with consistent serve/test/lint targets.
- **Generators** – `packages/data-service-generator/` and `packages/generator-blueprints/` show how to encode org-specific scaffolding inside Nx applications.

## Additional Resources
- [`README.md`](./README.md) – End-to-end onboarding guide, prerequisites, and npm script catalog.
- [`docs/`](./docs) & [`tutorials/`](./tutorials) – Architecture diagrams and guided builds (React/Angular to-do apps, etc.).
- [`CONTRIBUTING.md`](./CONTRIBUTING.md) & [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) – Contribution workflow and behavior expectations.
- [`docker-compose.dev.yml`](./docker-compose.dev.yml) – Source of all required local services.
- [`nx.json`](./nx.json) – Authoritative definition of target dependencies, cacheable operations, and generator presets.
