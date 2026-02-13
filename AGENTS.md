# AGENTS.md

## 📦 Project Overview

Amplication is an Nx-powered monorepo designed for generating backend services using custom templates. Major technologies include TypeScript, Node.js, Nx, Jest, Docker/docker-compose, GitHub Actions, React, NestJS, Prisma, Postgres, Redis, and Kafka. The codebase supports modular backend, client, and enterprise components under unified project management.

## 🗂 Repository Structure

- `packages/` — Individual applications (each with its own README)
- `libs/` — Shared libraries for cross-app use
- `ee/` — Enterprise-exclusive components (separate license; commit restrictions apply)
- `docs/` — Documentation content
- `tutorials/` — Tutorials and guides
- `scripts/` — Setup/build scripts
- `.github/`, `.devcontainer/`, `.husky/`, `.vscode/` — Tooling/configuration folders
- Root configs: `package.json`, `nx.json`, `docker-compose.dev.yml`, etc.

## ⚙️ Development Guidelines & Workflow

### Setup Instructions

> **Agents must verify path existence before operations. Use Nx commands where applicable.**

1. **Clone & Install:**
   ```bash
   git clone https://github.com/amplication/amplication.git && cd amplication && npm install
   ```
2. **Initial Setup:**
   ```bash
   npm run setup:dev
   ```
3. **Run Infrastructure:**
   ```bash
   npm run docker:dev         # Starts Docker containers interactively
   npm run docker:dev -- -d   # Starts Docker containers in detached mode
   ```
4. **Apply Database Migrations:**
   ```bash
   npm run db:migrate:deploy
   ```
5. **Serve Applications (in separate terminals):**
   ```bash
   npm run serve:server
   npm run serve:client
   npm run serve:dsg
   npm run serve:git
   npm run serve:plugins
   ```

### Prerequisites
- Install TypeScript globally: `npm install -g typescript`
- Ensure Node.js and npm versions per `package.json`
- Docker must be running

### Contribution Workflow
- **Branch from** `next` (never `main`)
- **Open PRs against** `next`
- **Use forks** (never direct commits)
- **Coordinate major changes via** issues/Discord
- **Community tasks:** Watch for `open to community` or `good first issue` labels
- **Commit format:** `<type>(<package>): <subject>` (types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`; packages e.g. `server`, `client`, `data-service-gen`)
- **Read** [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) **before contributing**
- **Do NOT commit changes from** `ee/` **to unrestricted branches or PRs**

## 🪄 Code Patterns

- **Simple:** DI pattern in `libs/util/logging` (`ILogger`)
- **Complex:** Application setup, Nx targets, Docker packaging in `packages/amplication-server`
- **Common:** Every module includes `README.md`, `project.json`, `jest.config.ts` following architectural conventions

## 🧪 Quality Standards

- **Testing:** Nx targets and local `jest.config.ts` files per module; unit and integration testing expected
- **Linting:** Follow Nx and repo patterns; config present per module
- **Coverage/Build:** All apps and libraries are subject to Nx-driven coverage, linting, build enforcement

## 🚨 Critical Rules

- Agents must verify directory existence and use Nx/applicable commands
- Do **not** commit enterprise (`ee/`) code unless properly licensed and targeted
- Follow branch/PR/completion guidelines strictly
- Use prescribed commit message format
- Review [CONTRIBUTING.md](CONTRIBUTING.md) **and** [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) **before making changes**

## 🏗️ Common Tasks

- **Clone, install, and setup:** See commands above
- **Run core infrastructure:** `npm run docker:dev` / `npm run docker:dev -- -d`
- **Apply database migrations:** `npm run db:migrate:deploy`
- **Serve applications:** `npm run serve:<app>`
- **Check per-application README for advanced instructions**
- **Coordinate via issues and Discord for major efforts**

## 📖 Reference Examples

- `libs/util/logging`: Dependency Injection (`ILogger`) pattern
- `packages/amplication-server`: Application setup, Nx targets, Docker config
- Module structure conventions: `README.md`, `project.json`, `jest.config.ts` in each module

## 📚 Additional Resources

- [README.md](README.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- [`tutorials/` Directory](tutorials/)
- [`docs/` Directory](docs/)

---

*This AGENTS.md is intended as an operational and architectural guide for automation agents and contributors. Follow project conventions, verify paths, and always consult referenced resources before making changes.*
