# 🤖 Agents Guide for the Amplication Monorepo

## 1. Project Overview
Amplication is a large-scale Nx-managed TypeScript monorepo that powers backend services, a React client, automation tooling, and enterprise overlays. The stack centers on NestJS + GraphQL + Prisma + PostgreSQL for services, React + MUI for the UI, an oclif-based CLI, and Kafka/Redis/SendGrid/Novu/OpenTelemetry integrations. Development relies on Docker Compose for infra, Nx for orchestration, and strict formatting/testing pipelines enforced through Husky, ESLint, Prettier, and Nx Cloud caching.

## 2. Repository Structure & Key Directories
| Directory | Purpose | Notes |
| --- | --- | --- |
| `packages/` | Deployable apps/services (server, client, CLI, GPT gateway, notification service, storage gateway, build manager, DSG, etc.) | Each package has its own `project.json`, README, and Nx targets. |
| `libs/` | Shared Nx libraries (schema registry, UI design system, utilities) | Consume via Nx import paths; update READMEs when APIs change. |
| `ee/` | Enterprise Edition overlays mirroring OSS structure | **Never modify without explicit scope**; uses a different license. |
| `docs/` | Architecture diagrams (`*.drawio`) and shared documentation assets | Keep diagrams updated when flows change; store source `.drawio` files here. |
| `tutorials/` | Blog/marketing-style Markdown posts and walkthroughs | Use for narrative tutorials, announcements, and marketing drafts. |
| `.github/` | CI/CD workflows (CI, release, security, DSG pipelines) | Keep workflows aligned with Nx targets; do not hardcode secrets. |
| `.devcontainer/`, `.husky/`, `.vscode/`, root configs (`nx.json`, `tsconfig.base.json`, `jest.config.ts`, `docker-compose.dev.yml`, `.env.docker-compose`) | Tooling, editor, and environment configuration | Update alongside tooling changes; respect lint/format settings. |
| `scripts/` | Automation utilities (`setup.ts`, `publish.mjs`, `coverageMerger.js`) | Follow existing patterns when adding automation. |

> ℹ️ Each package/library typically documents environment variables or usage details in its local README. Always consult and update those files when changing behavior.

## 3. Tooling & Environment Setup
- **Prerequisites:** Node.js **22.13**, npm bundled with that Node release, Nx **16.9.1** (workspace-managed), Docker Desktop/Engine, Git, and optional global TypeScript (`npm install -g typescript`).
- **Install & bootstrap:**
  ```bash
  npm install
  npm run setup:dev
  ```
  `npm run setup:dev` invokes the canonical automation script `scripts/setup.ts` to install dependencies, build packages, and keep the Nx graph metadata consistent.
- **Infrastructure services:**
  ```bash
  npm run docker:dev        # foreground logs
  npm run docker:dev -- -d  # daemonized
  npm run db:migrate:deploy # keep Prisma schema & DB aligned
  ```
- **Workspace conventions:** Use Nx-managed commands (`nx <target> <project>` or npm scripts that proxy Nx). Never invoke `tsc`, `jest`, etc., directly unless a project-specific README explicitly requires it.

## 4. Development Workflow & Commands
All local activity should go through Nx commands or the npm scripts that proxy them—avoid calling underlying tooling (Jest, Prisma, etc.) directly unless a package README explicitly requires it.
1. **Serve applications:**
   ```bash
   nx serve amplication-server
   nx serve amplication-client
   nx serve amplication-cli
   ```
   Use the identifiers defined in `project.json`. Multiple services can run concurrently; ensure infra (`docker:dev`) is up for server-dependent apps.
2. **Test suites:**
   ```bash
   nx test amplication-server
   nx affected --target=test --base=origin/main --head=HEAD
   nx affected --target=lint --base=origin/main --head=HEAD
   nx affected --target=build --base=origin/main --head=HEAD
   ```
   Targeted Nx commands keep CI parity. Jest config is centralized through `jest.preset.js`.
3. **Lint & format:**
   ```bash
   nx lint <project>
   nx format:write --projects <project>
   ```
   Husky/lint-staged run these automatically; run locally before opening a PR.
4. **Coverage aggregation:** `scripts/coverageMerger.js` combines package reports during CI. When altering test output paths, update this script.
5. **Automation references:** `scripts/setup.ts` shows the expected Node-based automation style (CLI args, logging, error handling).

## 5. Code & Testing Patterns
- **Backend:** NestJS modules with GraphQL resolvers, Prisma clients, Kafka/Redis integrations. Tests favor Jest with dependency-injected modules.
- **Frontend:** React + MUI, story-driven components in `libs/ui/design-system`. Hooks/components rely on Nx library boundaries.
- **CLI:** `packages/amplication-cli` leverages oclif; commands are documented in its README. Follow its command registration pattern when extending.
- **Shared libraries:** Use Nx generators to create libraries; ensure public APIs are exported via each lib’s `index.ts` and documented in README or Storybook when UI-related.
- **Testing:**
  - Unit: `nx test <project>` (Jest).
  - Integration/e2e: defined per project; consult package README.
  - Affected runs: `nx affected --target=test|lint|build` to mirror CI behavior.

## 6. Quality & Compliance Safeguards
- **Formatting & Linting:** ESLint + Prettier enforced via Husky pre-commit hooks (`.husky/`). Local or CI failures must be fixed before contributing.
- **Secrets:** `.env.docker-compose` documents required env vars; **never commit credentials**. Use placeholders and reference 1Password/secret managers where applicable.
- **CI/CD:** `.github/workflows/*.yml` orchestrate tests, releases, security scans. Align new targets or scripts with existing workflows.
- **Licensing:** OSS code is Apache 2.0. `ee/` code follows the Amplication Enterprise license; changes require explicit approval.
- **Caching:** Nx Cloud caching is enabled; respect target inputs/outputs to avoid cache poisoning.

## 7. Critical Rules for Agents
1. **Never commit secrets or personally identifiable information.** Use sample values (`<PLACEHOLDER>`) in docs.
2. **Respect OSS vs EE boundaries.** Only touch `ee/` when the task explicitly scopes it.
3. **Use Nx targets & scripts.** Do not bypass Nx (e.g., running raw `jest`) unless a package README mandates it.
4. **Update documentation alongside code.** Especially README env tables and diagrams under `docs/` or `tutorials/`.
5. **Do not push commits directly.** All contributions flow through PRs handled after agent work concludes.
6. **Preserve automation patterns.** Mirror `scripts/setup.ts` conventions for new or updated tooling and refresh any documentation referencing the automation.
7. **Run formatting/testing locally before handing off.** Mirrors Husky + CI expectations.

## 8. Common Tasks for Agents
### A. Initial Local Setup
1. `npm install`
2. `npm run setup:dev`
3. `npm run docker:dev` (optionally `-- -d`)
4. `npm run db:migrate:deploy`
5. Start required apps via `nx serve <project>`

### B. Running a Service + Client
1. Ensure Docker infra is running.
2. In one terminal: `nx serve amplication-server`
3. In another: `nx serve amplication-client`
4. For feature-specific work, start additional apps (e.g., `nx serve amplication-data-service-generator`).

### C. Executing Tests & Coverage
1. Targeted tests: `nx test amplication-server`
2. Workspace-wide affected tests: `nx affected --target=test --base=origin/main`
3. Merge coverage (CI or local): `node scripts/coverageMerger.js` after generating coverage outputs.

### D. Updating Dependencies
1. Verify scope (OSS vs EE).
2. Update `package.json` / lockfile via `npm install <pkg>@<version>`.
3. Run `nx affected --target=lint,test,build` to validate.
4. Document notable changes in relevant READMEs.

### E. Adding or Updating Documentation
1. Edit Markdown (e.g., `docs/`, `tutorials/`, package README) with clear instructions.
2. Include code fences for commands, tables for env vars.
3. Reference authoritative sources (e.g., `packages/amplication-server/README.md`).
4. Re-run `nx format:write --files <path>` if needed.

### F. Introducing Automation
1. Review `scripts/setup.ts` for patterns (argument parsing, logging).
2. Place new scripts in `scripts/` and wire through `package.json` or Nx targets.
3. Update documentation referencing the new command.

## 9. Reference Examples & Templates
- `scripts/setup.ts` – canonical automation script structure (logging, error handling, workspace orchestration).
- `packages/amplication-cli/README.md` – CLI documentation & command modeling for oclif-based tooling.
- `packages/amplication-server/README.md` – environment variable table, service targets, and NestJS conventions.
- `packages/gpt-gateway/README.md` – gateway-specific env matrix and OpenAI workflow guidance for GPT-powered services.
- `packages/notification-service/README.md` – event-driven notification runner overview; reference when touching Novu/SendGrid/Kafka flows.
- `packages/amplication-client/` – React + MUI project layout, Nx `project.json` usage.
- `libs/ui/design-system/` – shared component patterns, Storybook-ready exports.
- `docs/ERD.drawio` – authoritative entity-relationship diagram source; update via Draw.io when data models shift.
- `tutorials/deploy-to-azure.md` – reference for simple documentation/blog-style contributions.
- `tutorials/social_media_post.md` – lightweight template for marketing or announcement posts.

Use these files when adding new commands, services, or documentation to ensure consistency.

## 10. Additional Resources & Pointers
- [Root README](./README.md) – onboarding overview, prerequisites, scripts.
- [Contributing Guide](./CONTRIBUTING.md) & [Code of Conduct](./CODE_OF_CONDUCT.md) – collaboration standards.
- [Docs Portal](https://docs.amplication.com) – detailed user-facing documentation.
- [Discord](https://amplication.com/discord) – coordination with maintainers.
- Nx documentation: <https://nx.dev> – for generators, caching, target configuration.

> ✅ **Reminder:** Always verify current instructions from package-level READMEs before making changes, and ensure any modifications are mirrored in the relevant documentation. Keeping this guide and related docs updated is part of every task.
