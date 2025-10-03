### Deco Workflow Builder — Blueprint (Minimal)

> **New to Deco?** It's a platform for building MCP (Model Context Protocol) apps that run on Cloudflare Workers. Think: serverless functions + AI integrations + type-safe RPC, all deployed to the edge in one command.

### Tech Stack
- **Frontend**: React 19 + Vite + TanStack (Router/Query) + Tailwind CSS
- **Backend**: Cloudflare Workers + Deco Runtime (MCP server)
- **Database**: SQLite (Durable Objects) + Drizzle ORM
- **Execution**: QuickJS sandbox (`DECO_TOOL_RUN_TOOL`) for dynamic code
- **AI**: `AI_GATEWAY.AI_GENERATE_OBJECT` (structured JSON generation)

### Flow
1) User prompt → AI generates a `WorkflowStep` (code + schemas + default input)
2) Resolve `@refs` in step.input (from previous steps/global input)
   - **CRITICAL**: @refs must use EXACT step IDs (e.g., `@step_1759490947550_abc.output.field`)
   - Never shorten to `@step1` or `@step-1` (these IDs don't exist!)
3) Execute via `DECO_TOOL_RUN_TOOL`
4) Save result → available to next steps via their exact ID
5) UI shows result with scroll, copy JSON button, syntax highlighting

### Execution Engine Architecture
- **Isomorphic engine** - mesmo código roda no client (UX) e server (RPC)
- **Sub-JSON strategy** - step outputs salvos como arquivos separados (`executions/run-id/step-id.json`)
- Apenas referências mantidas em memória para evitar bloat
- Formato de referência: `{ ref: "stepId", path: "output.field" }` ao invés de dados completos

### Minimal Data Model
```
WorkflowStep {
  id, name, description,
  code, inputSchema, outputSchema,
  input, result?, output?, error?, status?,
  inputView?, outputView?,
}
```

### State Management (Frontend)
- **Zustand store** (`view/src/store/workflow.ts`) - single source of truth for workflow state
- States como `steps`, `currentStepIndex`, `editingCode`, `isPlaying` devem estar no Zustand
- Apenas estados efêmeros/locais (hover, dropdown open/close) podem usar `useState`
- Zustand facilita: compartilhamento entre componentes, persistência localStorage, single source of truth
- Hooks como `updateStepInput(stepId, field, value)` para atualizar dados de form inputs

### Integrations Access (CRITICAL)
- ✅ Always bracket notation: `ctx.env['i:workspace-management'].DATABASES_RUN_SQL({ sql })`
- ❌ Never `ctx.env.SELF.*`

### Verified Tools (safe to use now)
- `ctx.env['i:workspace-management'].AI_GENERATE_OBJECT` - Structured AI generation
- `ctx.env['i:workspace-management'].DATABASES_RUN_SQL` - SQL queries
- `ctx.env['i:workspace-management'].KNOWLEDGE_BASE_SEARCH` - Document search
- Pure JS logic (no tool calls) - Math, string operations, data transforms

**Integration access**: Use bracket notation `ctx.env['i:workspace-management']` inside step code.

Note: Full tool access may be restricted; ask Gimenes/Candeia to enable if needed. Until then, keep to the verified set above.

### Commands (IMPORTANT: Run Manually!)
```bash
# Development (run manually, don't auto-start)
npm run dev

# Generate types for external integrations
npm run gen

# Generate self-types (your own tools/workflows)
# 1. Start dev server first
# 2. Copy dev URL from terminal (e.g., https://localhost-xxxx.deco.host)
# 3. Run:
DECO_SELF_URL=<your-dev-url>/mcp npm run gen:self

# Build (use bun for reliability)
bun run build

# Deploy
npm run deploy
```

**Note**: Use `bun` for install/build (faster and more reliable than npm).

### wrangler.toml Essentials
- `main = "server/main.ts"`
- `compatibility_date = "2025-06-17"`
- Assets bound as `ASSETS` serving `dist/client` with SPA fallback
- `[deco.bindings]`: `AI_GATEWAY`, `DATABASE`, `TOOLS` (used by steps)

### Porting to Another App
- Expose your domain APIs as MCP tools (one endpoint → one tool)
- Keep step code stateless; access via `ctx.env['your-integration']`
- Start with: `RUN_WORKFLOW_STEP`, `GENERATE_STEP`, `DISCOVER_WORKSPACE_TOOLS`

### Testing (CLI)
```bash
cd /Users/candeia/deco/app
node scripts/test-tool.js GENERATE_STEP '{"objective":"Query the database to count all todos"}'
node scripts/test-tool.js RUN_WORKFLOW_STEP '{"step":{...}}'
```

Minimal, production-ready pattern: generate → resolve @refs → execute → chain.


