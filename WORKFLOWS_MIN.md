### Workflows — Minimal Usage

> **What are workflows?** Multi-step processes where each step can use data from previous steps. Think: "Step 1 fetches todos → Step 2 counts them → Step 3 generates summary".

### Core Tools (these 3 make everything work)
- `RUN_WORKFLOW_STEP`: resolves `@refs` then executes step code via `DECO_TOOL_RUN_TOOL`.
- `GENERATE_STEP`: AI builds a self-contained `WorkflowStep`.
- `DISCOVER_WORKSPACE_TOOLS`: static catalog (verified tools).

### @refs (CRITICAL: Use Exact IDs!)
- Use in step.input to consume data from previous steps
- **MUST use EXACT step ID** (long generated IDs like `step_1759490947550_abc`)
- Examples:
  - `@step_1759490947550_p9ugp5tmn.output.result` ✅
  - `@input.userId` ✅ (for global input)
- **NEVER shorten**:
  - `@step-1.result` ❌ (this ID doesn't exist!)
  - `@step1.output` ❌ (this ID doesn't exist!)

### Minimal patterns
```ts
// Generate step
const gen = await client.GENERATE_STEP({
  objective: 'List all todos',
  previousSteps: steps.map(s => ({ id: s.id, name: s.name, outputSchema: s.outputSchema })),
});

// Run step with resolved refs
const previousStepResults = Object.fromEntries(
  steps.filter(s => s.result?.success).map(s => [s.id, s.result.output])
);
const run = await client.RUN_WORKFLOW_STEP({
  step: gen.step,
  previousStepResults,
  globalInput: { userId: 'u-1' },
});
```

### Step code (inside DECO_TOOL_RUN_TOOL)
- Must be ES module string with:
  - `export default async function (input, ctx) { ... }`
  - Use bracket notation: `ctx.env['i:workspace-management'].AI_GENERATE_OBJECT(...)`
  - Wrap with try/catch, return matches `outputSchema`

### Testing (CLI)
```bash
cd /Users/candeia/deco/app
node scripts/test-tool.js GENERATE_STEP '{"objective":"Count todos in DB"}'
node scripts/test-tool.js RUN_WORKFLOW_STEP '{"step":{...}}'
```


