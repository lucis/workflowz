### Server, Tools, and wrangler (Minimal)

> **What is this?** The backend of your Deco app. It's a Cloudflare Worker that serves both your MCP server (at `/mcp`) and your React frontend (at `/`).

### Server entry (`server/main.ts`)
- Uses `withRuntime<Env>` to configure everything:
  - `tools`: aggregated from `server/tools/index.ts`
  - `workflows`: from `server/workflows/index.ts`
  - `views`: from `server/views.ts`
  - `fetch`: `env.ASSETS.fetch(req)` (serves frontend)
- OAuth scopes requested: `AI_GENERATE`, `AI_GENERATE_OBJECT`, `DATABASES_RUN_SQL`, `DECO_TOOL_RUN_TOOL`.

### wrangler.toml (key points)
- `main = "server/main.ts"`
- `compatibility_date = "2025-06-17"`
- Assets binding:
  - `[assets] directory = "./dist/client"`, `binding = "ASSETS"`, `not_found_handling = "single-page-application"`, `run_worker_first = true`
- `[deco]`:
  - `workspace = "/candeia/default"`
  - `enable_workflows = true`
  - `local = false`
- Bindings present and required by steps:
  - `AI_GATEWAY`, `DATABASE`, `TOOLS`

### Domain tools (where to add new ones)
- `server/tools/` organized by domain
- Aggregate in `server/tools/index.ts`

Verified working tools to target now (keep to these unless platform access is expanded):
- `ctx.env['i:workspace-management'].AI_GENERATE_OBJECT({...})`
- `ctx.env['i:workspace-management'].DATABASES_RUN_SQL({ sql, params? })`
- `ctx.env['i:workspace-management'].KNOWLEDGE_BASE_SEARCH({ query, topK })`
- Pure JS execution (no tool calls)

Note on platform access
- Some workspace-wide tools may be restricted. Coordinate with Gimenes/Candeia to enable broader access. Until then, generate step code that only relies on the verified set.

### Commands (must-run manually where applicable)
```bash
# Install dependencies (use bun, not npm!)
bun install

# Dev server (run manually, never auto-start)
npm run dev

# Generate types for external integrations (AI_GATEWAY, DATABASE, TOOLS)
npm run gen

# Generate self-types (your own tools/workflows → typed RPC in frontend)
# 1. Start dev server: npm run dev
# 2. Copy dev URL from logs: https://localhost-xxxx.deco.host
# 3. Add /mcp to URL and run:
DECO_SELF_URL=https://localhost-xxxx.deco.host/mcp npm run gen:self

# Build (use bun for clean builds)
bun run build

# Deploy to production
npm run deploy
```

**Troubleshooting**:
- If `npm run build` fails with EPIPE: Use `bun run build` instead
- If cache issues: `rm -rf node_modules dist .vite && bun install`

### Tool patterns (Mastra)
- Always one integration call per tool step; complex logic in workflows `.map/.branch/.parallel`.
- Zod input/output schemas enforced at tool boundary.


