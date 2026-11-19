# Arca Workers

> **Cloudflare Workers monorepo hosting backend microservices, sync relays, and edge APIs for the Arca Health platform.**

Arca (Automated Records & Clinical Archive) is an offline-first, privacy-focused health tracking companion and longitudinal medical records platform. This repository hosts all Cloudflare Workers services and shared packages that power Arca's cloud infrastructure and edge operations.

---

## Repository Structure

```
workers/
├── .github/
│   └── workflows/
│       └── ci.yml               # Automated verification and typechecking pipeline
├── packages/                    # Shared TypeScript libraries, schemas, and configurations
│   └── .gitkeep
├── workers/                     # Cloudflare Worker microservices
│   └── .gitkeep
├── .editorconfig                # Workspace formatting rules
├── .gitignore                   # Monorepo git ignore patterns
├── .nvmrc                       # Pinned Node.js version (22)
├── package.json                 # Monorepo root scripts & dev dependencies
├── pnpm-lock.yaml               # Pnpm lockfile
├── pnpm-workspace.yaml          # Monorepo workspace definitions & build authorizations
├── tsconfig.base.json           # Shared base TypeScript configuration for workers & packages
├── tsconfig.json                # Root TypeScript configuration
└── README.md                    # Project documentation
```

- **`workers/*`**: Standalone Cloudflare Worker services (e.g. edge API gateways, sync/vault relays, webhooks, auth verification).
- **`packages/*`**: Shared internal libraries, TypeScript typings, validation schemas, and common utility modules consumed by workers.

---

## Prerequisites

- **Node.js**: `^20.0.0` or `^22.0.0` (managed via `.nvmrc`)
- **pnpm**: `^9.0.0` or `^10.0.0+` (`packageManager: pnpm@12.21.0`)
- **Cloudflare Account**: With access to Workers and Wrangler CLI (wrangler is bundled locally in `devDependencies`)

---

## Getting Started

1. **Clone the repository:**
   ```bash
   git clone git@github.com:arcahealth/workers.git
   cd workers
   ```

2. **Use the correct Node.js version:**
   ```bash
   nvm use
   ```

3. **Install workspace dependencies:**
   ```bash
   pnpm install
   ```

---

## Workspace Commands

The monorepo provides root-level scripts that execute across all packages and workers:

| Command | Description |
| :--- | :--- |
| `pnpm dev` | Run all workers in local development mode concurrently |
| `pnpm build` | Build all workers and shared packages |
| `pnpm typecheck` | Run TypeScript type checks across all projects |
| `pnpm lint` | Run code quality linters across all projects |
| `pnpm test` | Run test suites across all projects |
| `pnpm wrangler <cmd>` | Run Cloudflare Wrangler CLI from workspace root |

To run commands within a specific worker or package:

```bash
# Run dev for a specific worker
pnpm --filter <worker-name> dev

# Add a dependency to a specific package
pnpm --filter <worker-name> add <package>
```

---

## Creating a New Worker

To add a new Cloudflare Worker to the monorepo:

1. **Create the worker directory:**
   ```bash
   mkdir -p workers/<worker-name>/src
   ```

2. **Add `package.json` in `workers/<worker-name>/package.json`:**
   ```json
   {
     "name": "@arcahealth/<worker-name>",
     "version": "0.1.0",
     "private": true,
     "type": "module",
     "scripts": {
       "dev": "wrangler dev",
       "deploy": "wrangler deploy",
       "typecheck": "tsc --noEmit"
     }
   }
   ```

3. **Add `tsconfig.json` in `workers/<worker-name>/tsconfig.json`:**
   ```json
   {
     "extends": "../../tsconfig.base.json",
     "compilerOptions": {
       "outDir": "./dist",
       "rootDir": "./src"
     },
     "include": ["src/**/*"]
   }
   ```

4. **Add Wrangler configuration (`wrangler.jsonc` or `wrangler.toml`):**
   ```jsonc
   {
     "$schema": "node_modules/wrangler/config-schema.json",
     "name": "<worker-name>",
     "main": "src/index.ts",
     "compatibility_date": "2025-02-14",
     "compatibility_flags": ["nodejs_compat"]
   }
   ```

5. **Create entry point `src/index.ts`:**
   ```typescript
   export default {
     async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
       return new Response("OK", { status: 200 });
     },
   };
   ```

6. **Register dependencies:**
   ```bash
   pnpm install
   ```

---

## Environment Variables & Secrets

- **Local Development**: Place non-committed environment variables in a `.dev.vars` file inside the respective worker folder:
  ```env
  API_SECRET=example-local-secret
  ```
- **Production Secrets**: Use Wrangler to securely provision encrypted secrets in Cloudflare:
  ```bash
  pnpm --filter <worker-name> exec wrangler secret put <SECRET_KEY>
  ```

---

## Continuous Integration

Every push and pull request to `main` is validated via [GitHub Actions](.github/workflows/ci.yml) against:
- Workspace lockfile integrity (`--frozen-lockfile`)
- Monorepo-wide TypeScript typechecking
- Automated test suites
