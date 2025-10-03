# 🚀 Deco Workflow Builder — One-Pagers

> **Never heard of Deco?** Think: Cloudflare Workers + MCP + AI-powered tool orchestration.  
> **This template**: Visual workflow builder where AI generates executable code steps.

## 🌐 Try the Live Demo

**Test the concept app**: [candeia.deco.page](https://candeia.deco.page)

⚠️ **Alpha Status**: This is an early preview of the Deco workflow system. The platform is still in active development.

📝 **Note**: Source code is currently closed-source as we're finishing up the core platform. This documentation provides architectural patterns and concepts you can implement in your own projects.

---

## 🎁 What This Makes Easy

Before this template existed, building workflow automation required:
- ❌ Writing backend APIs manually
- ❌ Creating frontend forms for every tool
- ❌ Managing state for multi-step processes
- ❌ Connecting AI generation to execution
- ❌ Building cross-step data passing

**With this template:**
- ✅ Type-safe RPC (frontend ↔ backend) auto-generated
- ✅ AI generates executable steps from natural language
- ✅ @refs system for automatic data flow between steps
- ✅ Zustand state management pre-configured
- ✅ UI components for workflow visualization
- ✅ One-command deployment to edge network

**In other words**: This template is like **Zapier's visual builder** meets **GitHub Actions** meets **AI code generation**, all running on the edge.

---

## 📖 Quick Navigation

If you're **implementing this in your app** or **learning the architecture**, start here:

| Doc | What's Inside | Read Time |
|-----|---------------|-----------|
| [**BLUEPRINT.md**](BLUEPRINT.md) | High-level architecture, data flow, commands, wrangler config | 5 min |
| [**SERVER_AND_TOOLS.md**](SERVER_AND_TOOLS.md) | MCP server setup, tools organization, verified integrations | 4 min |
| [**WORKFLOWS_MIN.md**](WORKFLOWS_MIN.md) | @refs system, step execution, AI generation patterns | 3 min |
| [**ATREF_FORMAT.md**](ATREF_FORMAT.md) | ⚠️ CRITICAL: How to use @refs correctly (exact IDs!) | 3 min |
| [**FRONTEND_UX.md**](FRONTEND_UX.md) | Zustand state management, TanStack Query, UI patterns | 5 min |
| [**RICH_TEXT_EDITOR.md**](RICH_TEXT_EDITOR.md) | Tiptap rich text editor with @ mentions implementation | 7 min |
| [**CUSTOM_VIEWS_AND_TESTING.md**](CUSTOM_VIEWS_AND_TESTING.md) | Custom input/output views, testing with CLI | 4 min |
| [**WHY_THIS_TEMPLATE.md**](WHY_THIS_TEMPLATE.md) | What problems this solves, before/after comparison | 6 min |
| [**REFACTOR_SUMMARY.md**](REFACTOR_SUMMARY.md) | Recent refactor: prompts extracted, @refs fixed | 3 min |
| [**CHANGELOG.md**](CHANGELOG.md) | What changed on Oct 3, 2025 (latest updates) | 4 min |
| [**FINAL_STATUS.md**](FINAL_STATUS.md) | Production readiness, metrics, known issues | 5 min |

**Total:** ~50 minutes to understand the complete system.

💡 **Quick path**: Read BLUEPRINT → ATREF_FORMAT → CHANGELOG → Start building! (15 min)  
💡 **With rich text**: Add RICH_TEXT_EDITOR for @ mentions guide (22 min total)

---

## 🆕 Latest Updates (Oct 3, 2025)

### ✅ Recent Improvements
1. **@refs Fixed** - Now use exact step IDs (e.g., `@step_1759490947550_abc.output.field`)
2. **Prompts Refactored** - Separated into `server/prompts/workflow-generation.ts` for easy editing
3. **Export Improved** - Uses `window.prompt()` for easy JSON copy
4. **Lint Errors Reduced** - 204 → 60 errors (-70%)
5. **Build with Bun** - More reliable than npm (`bun install`, `bun run build`)
6. **Views Added** - Workflowz view registered in MCP server
7. **Rich Text Editor Guide** - Complete Tiptap + @ mentions implementation guide added

### 📋 What Works Now
- ✅ AI generates steps with correct @refs (tested!)
- ✅ Step execution resolves @refs correctly
- ✅ UI "Insert @ref" buttons use exact IDs
- ✅ Export shows full JSON in copyable prompt
- ✅ Build works with workers-runtime 0.21.1 (via bun)

---

## 🎯 What This System Does

**In one sentence:**  
AI generates executable workflow steps (code + schemas) → system resolves cross-step data references (@refs) → executes in QuickJS sandbox → chains results.

**Core loop:**
```
User: "Count todos in database"
  ↓ AI generates WorkflowStep with code
  ↓ User executes step
  ↓ Result: { count: 42 }
  ↓ Available as @step-1.count for next steps
```

---

## 🏗️ Architecture at a Glance

```
Frontend (React)          Backend (Cloudflare Workers)      Execution
     ↓                            ↓                              ↓
User prompt ────────→ GENERATE_STEP (AI) ──→ WorkflowStep JSON
     ↓                            ↓                              ↓
Show step preview    ←────────────┘                              │
     ↓                                                            │
Click "Run Step"     ────────────────────────────────────────────→
     ↓                                                            ↓
                                                          RUN_WORKFLOW_STEP
                                                                  ↓
                                                          1. Resolve @refs
                                                          2. Execute code (QuickJS)
                                                          3. Return result
     ↓                                                            │
Display result       ←───────────────────────────────────────────┘
```

---

## ⚡ Quick Start (4 Steps)

### 1. **Setup & Install**
```bash
bun install              # Use bun (faster than npm)
npm run configure        # Set app name/workspace (one-time)
```

### 2. **Start Development**
```bash
npm run dev              # Start server (manual command)
# Wait for: "Listening on http://localhost:8787"
```

### 3. **Generate Types** (Optional but recommended)
```bash
# In a new terminal:
npm run gen              # External integrations types

# Then for self-types:
# 1. Copy dev URL from Terminal 1 (e.g., https://localhost-xxxx.deco.host)
# 2. Add /mcp to the end
# 3. Run:
DECO_SELF_URL=https://localhost-xxxx.deco.host/mcp npm run gen:self
```

### 4. **Open & Test**
- Browser: `http://localhost:8787`
- Enter prompt: "Generate a hello world message"
- Click: "⚡ Generate Step with AI"
- Wait ~10s for AI to generate step
- Step appears in toolbar → Click it
- Click: "Execute Step"
- See result with poem/message! 🎉

**Create multi-step workflow**:
- Click on output field (e.g., "poem")
- Sees "📌 Insert @ref" → Click it
- Prompt updates with `@step_xxx.output.poem`
- Add description: "and reverse it"
- Generate Step 2 → AI uses the @ref!
- Execute → @ref resolves automatically ✨

---

## 🧩 Key Concepts (30 seconds each)

### @refs (Cross-Step References)
```javascript
// Step 1 output: { todos: [...] }
// Step 2 input:  { data: "@step-1.todos" }  ← resolved automatically
```

### WorkflowStep (Minimal Model)
```typescript
{
  id: "step-1",
  name: "Count Todos",
  code: "export default async function(input, ctx) { ... }",
  inputSchema: { /* JSON Schema */ },
  outputSchema: { /* JSON Schema */ },
  input: { /* default values or @refs */ },
  result?: { output, error, logs, duration }
}
```

### Integration Access (CRITICAL)
```javascript
// ✅ ALWAYS use bracket notation
ctx.env['i:workspace-management'].DATABASES_RUN_SQL({ sql })

// ❌ NEVER use dot notation
ctx.env.SELF.SOME_TOOL()  // Will fail!
```

---

## 🔍 What Makes This Portable?

This is a **blueprint, not just an app**. To port to your domain:

1. **Expose your APIs as MCP tools** (one endpoint = one tool)
2. **Keep step code stateless** (all data via `ctx.env['your-integration']`)
3. **Implement 3 core tools**:
   - `RUN_WORKFLOW_STEP` (execute + resolve @refs)
   - `GENERATE_STEP` (AI generates steps)
   - `DISCOVER_WORKSPACE_TOOLS` (catalog your tools)

The same UX patterns work across any domain (contracts, data pipelines, customer support, etc.).

---

## 🧪 Testing (No UI Needed)

```bash
cd /Users/candeia/deco/app

# Generate a step
node scripts/test-tool.js GENERATE_STEP '{"objective":"List todos"}'

# Execute a step
node scripts/test-tool.js RUN_WORKFLOW_STEP '{"step":{...}}'

# Test verified tools
node scripts/test-tool.js RUN_WORKFLOW_STEP '{
  "step": {
    "code": "export default async (input,ctx) => {
      const r = await ctx.env[\"i:workspace-management\"].DATABASES_RUN_SQL({sql:\"SELECT COUNT(*) as c FROM todos\"});
      return {count: r.result[0].results[0].c};
    }",
    "inputSchema": {}, "outputSchema": {}, "input": {}
  }
}'
```

---

## 📚 When to Read Each Doc

| Your Goal | Start With |
|-----------|------------|
| Understand overall system | **BLUEPRINT.md** |
| Set up server + wrangler | **SERVER_AND_TOOLS.md** |
| Implement step execution | **WORKFLOWS_MIN.md** |
| Build the frontend | **FRONTEND_UX.md** |
| Add rich text with @ mentions | **RICH_TEXT_EDITOR.md** |
| Add custom UI views | **CUSTOM_VIEWS_AND_TESTING.md** |
| Port to your domain | All docs, in order (~50 min) |

---

## 🚨 Current Limitations & Known Issues

### Limitations
- **Tool access restricted**: Only 4 verified tools work now (AI, DB, KB, pure JS). Ask platform team (Gimenes/Candeia) to enable broader access.
- **No persistence**: Workflows not saved to DB yet (localStorage only).
- **Custom views**: Implemented but not fully tested (TODO).

### Known Issues
- **AI errors sometimes**: "Provider returned error" can happen if:
  - Rate limit reached (wait 1 minute)
  - Claude API temporarily down
  - Authentication issues (check credentials)
- **Build cache**: If build fails, use `bun run build` instead of `npm run build`

### Troubleshooting
```bash
# Test if server works
node scripts/test-tool.js LIST_TODOS '{}'

# Test if AI works
node scripts/test-tool.js RUN_WORKFLOW_STEP '{...step with AI call...}'
```

See documentation for complete status.

---

## 🎓 Learning Path

**Beginner** (never seen this before):
1. Read BLUEPRINT.md
2. Run Quick Start
3. Try test-tool.js examples

**Intermediate** (want to extend):
1. All 5 one-pagers (20 min)
2. Study `examples/working-steps.json`
3. Check `server/tools/workflows.ts`

**Advanced** (porting to your app):
1. All one-pagers
2. Review `server/main.ts` + `wrangler.toml`
3. Read `shared/types/workflows.ts`
4. Implement your domain's tools

---

## 💬 Questions?

- **Architecture questions**: See documentation in this directory
- **Testing help**: See `examples/working-steps.json` if available

---

## ⭐ Quick Stats

- **Lines of Code**: ~15k (server + frontend)
- **Core Abstractions**: 3 (WorkflowStep, @refs, RUN_WORKFLOW_STEP)
- **Build Time**: <1s (Vite)
- **Execution Speed**: 50-500ms per step (depends on integration)
- **Type Safety**: 100% (zero `any`, full Zod validation)

---

**Built with:** React 19, Vite, TanStack (Router/Query), Tailwind, Cloudflare Workers, Deco Runtime, SQLite (Durable Objects), Drizzle ORM.

**License:** MIT (adapt freely!)

---

Made with ❤️ for the MCP ecosystem. Happy building! 🚀

