# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a [Backstage](https://backstage.io) Internal Developer Platform (IDP) demo running version 1.48.0. It is a monorepo managed with Yarn 4.4.1 (node-modules linker), consisting of two packages and a plugins directory for custom extensions.

## Commands

```bash
# Development
yarn install          # Install dependencies (auto-runs on devcontainer create)
yarn start            # Start frontend (port 3000) and backend (port 7007) concurrently
yarn workspace app start      # Frontend only
yarn workspace backend start  # Backend only

# Building
yarn build:all        # Build all packages
yarn build:backend    # Build backend only
yarn build-image      # Build Docker image

# Testing
yarn test             # Run unit tests across monorepo
yarn test:all         # Run with coverage
yarn test:e2e         # Run Playwright E2E tests

# Code quality
yarn lint             # Lint changes since origin/main
yarn lint:all         # Lint all files
yarn tsc              # TypeScript type check
yarn tsc:full         # Full TS check (skipLibCheck: false)
yarn fix              # Auto-fix lint issues
yarn prettier:check   # Check formatting

# Scaffolding
yarn new              # Create new plugin or package
yarn clean            # Remove build artifacts
```

## Architecture

### Monorepo Structure

- **`packages/app/`** — React 18 frontend. Entry point: `src/index.tsx`. Main routing in `src/App.tsx`, sidebar in `src/components/Root/Root.tsx`, entity details in `src/components/catalog/EntityPage.tsx`.
- **`packages/backend/`** — Node.js backend. Entry point: `src/index.ts`. Uses `createBackend()` with modular plugin loading via dynamic imports.
- **`plugins/`** — Custom plugin directory (currently empty; place new plugins here).
- **`examples/`** — Sample catalog entities, org structure, and scaffolder templates loaded at startup.

### Configuration

- **`app-config.yaml`** — Development config. SQLite in-memory DB, GitHub integration for org `solthoth`, guest auth enabled, TechDocs with local builder/publisher.
- **`app-config.production.yaml`** — Production overrides. PostgreSQL via env vars (`POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_USER`, `POSTGRES_PASSWORD`), binds to `0.0.0.0:7007`.
- **`app-config.local.yaml`** (gitignored) — Local developer overrides.

### Backend Plugins Loaded

The backend loads these Backstage plugins: app, auth (guest provider), catalog (with GitHub + GitHub Org providers), scaffolder (with GitHub actions + notifications), techdocs, search (with PostgreSQL module, catalog + techdocs indexers), kubernetes, notifications, signals, and permission.

### Frontend Routes

`/catalog` (default), `/docs`, `/create` (scaffolder), `/api-docs`, `/search`, `/catalog-graph`, `/settings`, `/notifications`, `/catalog/:namespace/:kind/:name`, `/docs/:namespace/:kind/:name/*`.

### GitHub Integration

The app auto-discovers repositories with `catalog-info.yaml` files and organization members from GitHub org `solthoth`. Requires `GITHUB_TOKEN` environment variable (mapped from `SOLTHOTH_GITHUB_TOKEN` in devcontainer).

### Database

- **Development:** better-sqlite3 (in-memory, no setup needed)
- **Production:** PostgreSQL (required for search indexing at scale)

## Key Conventions

- Backstage CLI (`@backstage/cli`) manages TypeScript, ESLint, and build tooling — avoid overriding its configs directly.
- New plugins go in `packages/` (for core extensions) or `plugins/` (for standalone plugins); use `yarn new` to scaffold them.
- Entity catalog data files follow Backstage YAML schema with `apiVersion: backstage.io/v1alpha1` or `v1beta1`.
- The `examples/` directory entities are loaded via `catalog.locations` in `app-config.yaml` and are only for demo purposes.
