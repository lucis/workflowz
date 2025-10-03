### Custom Views (Input + Output) and Testing — Minimal

> **What are custom views?** Instead of showing raw JSON, you can generate pretty UIs: tables with badges, forms with dropdowns, cards with charts. Think: transforming `{"todos": [...]}` into a sortable table.

**Purpose**: Optional UI helpers for step input (forms) and output (display).  
**Status**: ✅ Implemented with error-proof base64 data injection (Oct 3, 2025).

### Recent Fix (Oct 3)
**Problem**: Custom views showed "N/A" instead of actual data (timing issue + string injection risks)

**Solution**: Base64 encoding for safe data injection
- Data encoded to base64 → injected as plain string → decoded in iframe
- Error-proof: try/catch with fallback to `{}`
- No string concatenation issues (quotes, newlines, unicode all safe)

### Where it lives
- Types: `shared/types/views.ts` (JSON ViewDefinition) and `shared/types/workflows.ts` (step.inputView/outputView)
- Tools: `server/tools/views.ts` (GENERATE_VIEW, VALIDATE_VIEW, GET_VIEW_EXAMPLES)
- Renderer: `view/src/components/ViewRenderer.tsx`

### Two approaches
1) JSON ViewDefinition (preferred, safe)
   - Output components: container, card, heading, text, code, badge, table, list, divider, tableList(hardcoded)
   - Input components: input, select, textarea, checkbox, file-upload (onChange hooks)
2) QuickJS HTML generation (optional)
   - Use `DECO_TOOL_RUN_TOOL` to execute code that returns `{ html }`
   - Render HTML in frontend (prefer iframe/sanitization). Use only if needed.

### Explicit data flow (MUITO IMPORTANTE)
**Output View:**
1. Run step → result saved in `step.result`
2. Generate view (AI) or handcraft JSON → save to `step.outputView`
3. `<ViewRenderer definition={step.outputView} data={step.result.output} />`
4. Renderer lê `step.outputView` + `step.result.output` e renderiza

**Input View:**
1. Have `step.inputSchema`
2. Generate input view (AI) → save to `step.inputView`
3. `<ViewRenderer definition={step.inputView} isForm formValues={step.input} onChange={(name,val)=>updateStepInput(stepId,name,val)} />`
4. onChange atualiza Zustand store: `updateStepInput(stepId, fieldName, value)`
5. Execute step uses updated `step.input` from Zustand

**Conexão ViewDefinition → Dados → Zustand:**
- ViewDefinition é a estrutura da UI (JSON schema-like)
- Dados vêm de `step.result.output` (output) ou `step.input` (input)
- onChange de input views atualiza Zustand via `updateStepInput()`
- Este fluxo deve estar EXPLÍCITO na documentação de Custom Views

### Prompts control (server)
- Keep prompts compartmentalized and imported by the tool
  - Components catalog
  - Design system
  - Output guidelines (tables, badges, data paths)
  - Input guidelines (name required, submit button, types)

### Testing with CLI
```bash
cd /Users/candeia/deco/app

# Generate a view
node scripts/test-tool.js GENERATE_VIEW '{"purpose":"Todos table with badges","viewType":"output","dataSchema":{}}'

# Render HTML view via step (QuickJS approach)
node scripts/test-tool.js RUN_WORKFLOW_STEP '{"step":{"id":"view","name":"View","code":"export default async function(input){return { html: `<div>${input.value}</div>`}}","inputSchema":{},"outputSchema":{},"input":{"value":"Test"}},"previousStepResults":{},"globalInput":{}}'
```

Note on safety: If using HTML approach, sanitize or render in iframe; keep it simple. JSON approach is preferred.


