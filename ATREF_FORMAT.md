# 📎 @refs Format Guide (Critical!)

## 🎯 The Problem We Solved

Users were getting `"Step not found"` errors because @refs used shortened IDs that don't exist.

---

## 🎯 Recent Fix (Oct 3, 2025)

**What was broken**: UI inserted `@step1.output`, AI generated `@step1.output`, but actual ID was `step_1759490947550_abc` → @ref failed!

**What was fixed**:
1. ✅ UI now inserts exact IDs: `@step_1759490947550_abc.output.poem`
2. ✅ AI now receives exact IDs in context and uses them
3. ✅ resolve-refs.ts now strips `.output.` prefix correctly

**Status**: @refs work end-to-end now! 🎉

---

## ✅ Correct @ref Format

### Rule 1: Use EXACT Step IDs
Step IDs are generated with timestamps:
```
step_1759490947550_p9ugp5tmn
```

**You MUST use the full ID** in @refs:
```json
{
  "input": {
    "data": "@step_1759490947550_p9ugp5tmn.output.result"
  }
}
```

### Rule 2: Always Include Path
Don't just reference the step, reference the field:
```json
// ✅ GOOD - includes path
"@step_1759490947550_abc.output.result"
"@step_1759490947550_abc.output.todos[0].title"

// ⚠️ OK but passes whole object
"@step_1759490947550_abc.output"

// ❌ BAD - missing path
"@step_1759490947550_abc"
```

### Rule 3: Never Shorten IDs
```json
// ❌ THESE DON'T WORK
"@step1.output"
"@step-1.output"  
"@previous.output"
"@first.output"

// ✅ THIS WORKS
"@step_1759490947550_p9ugp5tmn.output.result"
```

---

## 📋 How to Get the Correct ID

### Method 1: From Export
1. Click "Export" button in UI
2. window.prompt() shows JSON
3. Copy step ID from there:
   ```json
   {
     "steps": [
       {
         "id": "step_1759490947550_p9ugp5tmn",  // ← Copy this!
         ...
       }
     ]
   }
   ```

### Method 2: From Console
After generating a step, check browser console:
```
✅ Step generated successfully: step_1759490947550_p9ugp5tmn
```

### Method 3: From UI (Future)
We can add a "Copy @ref" button next to each step that generates the correct reference.

---

## 🤖 How AI Generates Correct @refs

When you create Step 2 referencing Step 1, the AI receives:

```
Previous steps available (use @refs with EXACT IDs below):
- ID: step_1759490947550_p9ugp5tmn
  Name: Generate Hello World
  Reference as: @step_1759490947550_p9ugp5tmn.output
  Output schema: {"type":"object","properties":{"result":{"type":"string"}}}
```

AI is instructed:
```
CORRECT @ref format:
- Use FULL step ID: "@step_1759490947550_p9ugp5tmn.output.result"
- NEVER shorten to: "@step1" or "@step-1" (these won't work!)
- Always include path to field: ".output.result"
```

Result: AI generates correct @ref ✅

---

## 🧪 Testing @refs

### Test: Simple 2-step workflow
```bash
# Step 1: Generate data
node scripts/test-tool.js RUN_WORKFLOW_STEP '{
  "step": {
    "id": "step_abc123",
    "code": "export default async () => ({ result: \"Hello\" })",
    "inputSchema": {},
    "outputSchema": {},
    "input": {}
  }
}'

# Step 2: Use data with @ref
node scripts/test-tool.js RUN_WORKFLOW_STEP '{
  "step": {
    "id": "step_def456",
    "code": "export default async (input) => ({ reversed: input.text.split(\"\").reverse().join(\"\") })",
    "inputSchema": {},
    "outputSchema": {},
    "input": { "text": "@step_abc123.output.result" }
  },
  "previousStepResults": {
    "step_abc123": { "output": { "result": "Hello" } }
  }
}'
```

Expected:
```json
{
  "success": true,
  "output": { "reversed": "olleH" },
  "resolvedInput": { "text": "Hello" }
}
```

---

## 📖 Common Mistakes

### Mistake 1: Using Position Instead of ID
```json
"@step-0.output"  // ❌ Position not supported yet
```

**Fix**: Use exact ID shown in export or console.

### Mistake 2: Using `@ref:` Prefix
```json
"@ref:step_abc.output"  // ❌ Wrong prefix
```

**Fix**: Remove `ref:` prefix, just use `@step_abc.output`

### Mistake 3: Missing Path
```json
"@step_abc123"  // ⚠️ Passes entire step object (usually not what you want)
```

**Fix**: Add path: `"@step_abc123.output.result"`

### Mistake 4: Wrong Field Path
```json
"@step_abc123.result"  // ❌ Missing .output
```

**Fix**: Always include `.output`: `"@step_abc123.output.result"`

---

## 🎓 Summary

- ✅ Use EXACT step IDs (long generated ones)
- ✅ Always include `.output` in path
- ✅ Reference specific fields: `.output.result`
- ❌ Never shorten IDs
- ❌ Never use `@ref:` prefix
- ❌ Never use positional refs (not supported)

**When in doubt**: Export workflow → see exact IDs → copy/paste into @ref

---

Ready to build workflows that work! 🚀

