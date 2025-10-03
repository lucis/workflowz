### Frontend Architecture & UX Patterns

> **What is this?** The React app that users interact with. It talks to the backend via type-safe RPC calls and manages workflow state with Zustand.

### Stack
- **React 19** + **Vite** + **TanStack Router/Query** + **Tailwind CSS**
- **Zustand** for state management (workflow state, UI state)
- **shadcn/ui** components for consistent design
- **Lucide React** icons

### State Management Pattern
```typescript
// ✅ Use Zustand for shared/business logic state
const useWorkflowStore = create<WorkflowState>((set) => ({
  steps: [],
  currentStepIndex: 0,
  isPlaying: false,
  editingCode: false,
  
  // Actions
  addStep: (step) => set((state) => ({ 
    steps: [...state.steps, step] 
  })),
  updateStepInput: (stepId, field, value) => set((state) => ({
    steps: state.steps.map(s => 
      s.id === stepId 
        ? { ...s, input: { ...s.input, [field]: value } }
        : s
    )
  })),
  setCurrentStep: (index) => set({ currentStepIndex: index }),
}));

// ❌ Avoid useState for shared state
// ✅ useState only for purely local/ephemeral UI state (hover, dropdown)
```

### RPC Pattern (TanStack Query)
```typescript
// ✅ Wrap all RPC calls with TanStack Query hooks
export const useGenerateStep = () => {
  return useMutation({
    mutationFn: (input) => client.GENERATE_STEP(input),
    onSuccess: (data) => {
      useWorkflowStore.getState().addStep(data.step);
    },
  });
};

// ❌ Never call client.TOOL() directly in components
```

### UI Execution Result Display
When step executes, show result with:
- **Status badge** (Success ✓ / Failed ✗) with color coding (green/red)
- **Output/Error display** in syntax-highlighted `<pre>` block with proper JSON formatting
- **Copy JSON button** with visual feedback (changes to "✓ Copied!" for 2s, button turns green)
- **Scroll container** (`maxHeight: 400px, overflowY: auto`) for large outputs
- **Duration** (ms), **logs** (if any), and other execution metadata
- **Transition animations** for smooth UX

Example structure:
```tsx
<div className="execution-result">
  <div className="header">
    <h3>Execution Result</h3>
    <Badge status={step.status} />
    <Button onClick={copyJSON}>Copy JSON</Button>
  </div>
  <div className="content" style={{ maxHeight: '400px', overflowY: 'auto' }}>
    <pre>{JSON.stringify(step.output, null, 2)}</pre>
  </div>
</div>
```

### Navigation (TanStack Router)
```typescript
// ✅ Use Link for internal navigation (type-safe)
<Link to="/workflow/$id" params={{ id: workflow.id }}>
  View Workflow
</Link>

// ✅ Use useNavigate for programmatic navigation
const navigate = useNavigate();
navigate({ to: "/workflow/$id", params: { id } });

// ❌ Never use <a href> for internal routes
```

### Performance Patterns
1. **Memoize expensive computations**
   ```typescript
   const processedSteps = useMemo(() => 
     steps.map(processStep), 
     [steps]
   );
   ```

2. **Debounce rapid updates** (e.g., search, text input)
   ```typescript
   const [debouncedQuery] = useDebouncedValue(query, 150);
   ```

3. **Virtualize long lists** (if > 50 items)
   ```typescript
   import { useVirtualizer } from '@tanstack/react-virtual';
   ```

### File Organization
```
view/src/
├── components/        # Reusable UI components
│   ├── ui/           # shadcn/ui components
│   ├── ViewRenderer.tsx
│   └── WorkflowLayout.tsx
├── hooks/            # TanStack Query hooks for RPC
│   ├── useGenerateStep.ts
│   ├── useRunStep.ts
│   └── useWorkflows.ts
├── lib/              # Utilities
│   ├── rpc.ts        # RPC client
│   └── utils.ts      # cn(), etc.
├── routes/           # TanStack Router routes
│   ├── home.tsx
│   └── workflows.tsx
├── store/            # Zustand stores
│   └── workflow.ts
└── types/            # TypeScript types
    └── workflow.ts
```

### Key Features Implemented
- ✅ **Input onChange** - Controlled inputs that save automatically to Zustand
- ✅ **Custom Views** - Toggle between JSON and custom view rendering
- ✅ **Edit Code Modal** - Modal with Escape to close, save/cancel buttons
- ✅ **Export/Import** - JSON clipboard + file download/upload
- ✅ **Click to Reference** - Interactive execution monitor to insert @refs
- ✅ **Step Player** - Bottom bar with play/pause controls

### Common Pitfalls
❌ **Don't:** Use useState for workflow/step state  
✅ **Do:** Use Zustand store

❌ **Don't:** Call `client.TOOL()` directly in components  
✅ **Do:** Wrap with TanStack Query hooks

❌ **Don't:** Use `<a href>` for internal navigation  
✅ **Do:** Use TanStack Router `<Link>` or `useNavigate()`

❌ **Don't:** Use `defaultValue` for inputs (uncontrolled)  
✅ **Do:** Use `value + onChange` (controlled components)

❌ **Don't:** Forget to show execution results in UI  
✅ **Do:** Display output/error with copy button and scroll

❌ **Don't:** Re-render entire component tree on small changes  
✅ **Do:** Use Zustand selectors and React.memo strategically

