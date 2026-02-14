# 🚀 AGENTS.md — Amplication Repository Guide

Welcome, AI and human agents! This guide explains key workflows, repo structure, development conventions, quality standards, critical rules, and reference examples for navigating and contributing to [amplication/amplication](https://github.com/amplication/amplication), a TypeScript Nx monorepo for backend application generation.

---

## 📘 Project Overview
- **Purpose:** Automated backend/services generation, container management, and workflow orchestration via Nx monorepo.
- **Key Technologies:** TypeScript, Nx, Node.js, Docker, Jest, Prisma, NestJS, React, Apollo GraphQL.
- **Monorepo Structure:** All apps, services, libraries, and enterprise features managed centrally with Nx commands and shared dependencies.

---

## 🗂️ Repository Structure
```
amplication/
├── packages/           # Main apps/services (server, client, CLI, plugin-api, data-service-generator, etc.)
├── libs/               # Shared libs (ui/design-system, schema-registry, util/logging, etc.)
├── ee/                 # Enterprise-only features (EE license)
├── docs/               # Diagrams: ERD.drawio, build-process-artifacts.drawio, system-topolgy.drawio
├── scripts/            # Helper scripts (setup.ts, publish.mjs, coverageMerger.js)
├── tutorials/          # Learning guides, e.g. deploy-to-azure.md, amplication-intro.md
├── .github/            # CI/CD, workflows, repo meta
├── .husky/             # Git hooks (precommit, etc.)
├── .vscode/            # Editor config
├── package.json        # Root deps, scripts
├── tsconfig.base.json  # TypeScript config
├── nx.json             # Nx project config
├── jest.config.ts      # Testing config (Jest)
├── README.md           # Main repo guide
├── CONTRIBUTING.md     # Contribution standards
├── CODE_OF_CONDUCT.md  # Contributor Covenant
└── LICENSE             # Apache 2.0; EE license in ee/
```

---

## 🛠️ Development Guidelines
### Setup
- **Node.js**: ^22.13.0 and **npm**: ^9 required (enforced by `package.json` + `scripts/setup.ts`).
- **Docker**: Must be running for infra scripts and db.
- **Install dependencies:**
  ```bash
  npm install
  npm run setup:dev
  ```
- **Start local dev infra (optional detach):**
  ```bash
  npm run docker:dev -- -d
  npm run db:migrate:deploy
  ```
- **Serve apps/services:**
  ```bash
  npm run serve:<app>
  # Examples:
  npm run serve:server
  npm run serve:client
  npm run serve:dsg
  npm run serve:git
  npm run serve:storage
  npm run serve:plugins
  npm run serve:notification
  ```

### Nx Workflow
- **Per-package/conventional Nx targets:**
  ```bash
  npx nx build <project>
  npx nx lint <project>
  npx nx test <project>
  npx nx serve <project>
  ```
  - Project examples: `amplication-server`, `amplication-client`, `amplication-cli` (from `packages/`).

### Formatting & Quality
- **Check & fix format:**
  ```bash
  npm run format:check
  npm run format:write
  ```
- **Build/test:**
  ```bash
  npm run build
  npm run test:ci
  npx nx test <project>
  ```
- **Coverage merging:**
  - Use `scripts/coverageMerger.js` after Nx tests.

---

## 🧩 Code Patterns & Shared Libraries
- **Shared libraries (under `libs/`):**
  - `ui/design-system`: Storybook stories, e2e tests.
  - `schema-registry`: Data schema models/utilities.
  - `util/logging`: Logging, test patterns.
- **Enterprise features:**
  - Entire `ee/` folder under EE license; separate from Apache 2.0 root.
- **Diagrams & Docs:**
  - `docs/ERD.drawio`, `docs/build-process-artifacts.drawio`, `docs/system-topolgy.drawio`.
- **Tutorials/Guides:**
  - `tutorials/deploy-to-azure.md`, `tutorials/amplication-intro.md`.

---

## ⚡ Common Tasks & Scripts
- **Setup:**
  - `npm run setup:dev` — Bootstrap dev environment.
  - `npm run clean` — Remove build artifacts.
- **Infra:**
  - `npm run docker:dev` — Start containers.
  - `npm run docker:dev:cleanup` — Clean infra.
- **DB:**
  - `npm run db:migrate:deploy` — Run DB migrations.
- **Testing:**
  - `npm run test:ci` — CI test pipeline.
  - `npm run test:custom:actions-migration` — Custom migration tests.
- **Debug:**
  - `npm run debug:dsg` — Debug data-service-generator.
- **Help:**
  - `npm run help` — List all scripts.
- **Pre-commit (Husky):**
  - `npm run prepare`
  - `npm run precommit` (run via Husky pre-commit hook)

---

## 🧬 Quality Standards & Testing
- **Nx Targets:**
  - Each project/app/lib must have `build`, `lint`, `test` Nx targets; see individual READMEs for specifics.
- **UI/Design System:**
  - Must include Storybook stories + e2e tests (`libs/ui/design-system`).
- **Logging/Utils:**
  - Tests & patterns in `libs/util/logging`.
- **Pre-commit:**
  - Husky pre-commit hook runs `npm run precommit`.
- **Node/npm version:**
  - Enforced minimums: Node ^22.13.0, npm ^9.

---

## 🚨 Critical Rules
- **Licensing split:**
  - Apache 2.0 for repo; **EE license** for everything in `ee/`.
- **Branch protection:**
  - `master`, `main`, and `next` are protected — *never commit directly*.
  - Husky hooks enforce branch/protection locally.
- **Contribution standards:**
  - Branch naming: per-feature/topic.
  - Commit format: `<type>(<package>): <subject>` (from `CONTRIBUTING.md`).
  - Pull requests must target `next`.
  - Follow [Contributor Covenant](CODE_OF_CONDUCT.md); contact support@amplication.com for any issues.
- **CI/Release requirements:**
  - Release pipeline expects tests, coverage, build on `master`/`next`.
  - Must support Nx `package:container`, `deploy:*` targets for release.
- **Setup prerequisites:**
  - Node/npm versions, Docker running before setup/dev, TypeScript installed globally.

---

## 🏆 Reference Examples
### Simple
- [packages/amplication-cli](packages/amplication-cli/README.md)
### Complex
- [packages/amplication-server](packages/amplication-server/README.md)
- [packages/amplication-client](packages/amplication-client/README.md)
### Docs
- [docs/ERD.drawio](docs/ERD.drawio), [docs/build-process-artifacts.drawio](docs/build-process-artifacts.drawio), [docs/system-topolgy.drawio](docs/system-topolgy.drawio)
### Scripts
- [scripts/setup.ts](scripts/setup.ts), [scripts/publish.mjs](scripts/publish.mjs), [scripts/coverageMerger.js](scripts/coverageMerger.js)
### Tutorials
- [tutorials/deploy-to-azure.md](tutorials/deploy-to-azure.md)
- [tutorials/amplication-intro.md](tutorials/amplication-intro.md)
### Lib Examples
- [libs/ui/design-system](libs/ui/design-system/README.md)
- [libs/schema-registry](libs/schema-registry/README.md)
- [libs/util/logging](libs/util/logging/README.md)

---

## 📚 Additional Resources
- [README.md](README.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- [.github/workflows/README.md](.github/workflows/README.md)
- [LICENSE](LICENSE)
- [ee/LICENSE](ee/LICENSE)

---

**For any issues, bugs, or contribution queries, open an issue or contact support@amplication.com.**
