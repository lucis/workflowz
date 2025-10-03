# 📝 Rich Text Editor & @ Mentions — Technical Guide

> **What is this?** Technical guide for implementing Tiptap-based rich text editors with @ mentions for workflow step inputs. This enables natural language input with autocomplete for tools, steps, and data references.

---

## 🎯 Use Cases

- **Workflow prompts** - Natural language input with @ mentions for tools/steps
- **Step inputs** - Rich text fields with autocomplete for previous step outputs
- **Documentation** - Inline references to tools and data
- **Collaboration** - Tag users, tools, or data points

---

## 📦 Stack

```json
{
  "@tiptap/react": "^2.11.7",
  "@tiptap/starter-kit": "^2.11.7", 
  "@tiptap/extension-mention": "^2.14.0",
  "@tiptap/extension-placeholder": "^2.11.7",
  "@tiptap/suggestion": "^2.14.0",
  "tiptap-markdown": "^0.8.10",
  "tippy.js": "^6.3.7"
}
```

### Why Tiptap?

- ✅ **Headless**: 100% UI control, zero injected CSS
- ✅ **Extensible**: Extension system based on ProseMirror
- ✅ **React-first**: Native hooks, React component node views
- ✅ **Markdown**: Auto serialization/deserialization
- ✅ **Performance**: ProseMirror's virtual DOM is extremely fast

---

## 🏗️ Architecture (3 Layers)

```
┌─────────────────────────────────────┐
│  EditorContent (React)              │  ← UI Layer
├─────────────────────────────────────┤
│  useEditor hook                     │  ← State Management
│  - extensions: [...]                │
│  - content                          │
│  - onUpdate                         │
├─────────────────────────────────────┤
│  ProseMirror (Core)                 │  ← Document Model
│  - Schema                           │
│  - Plugins                          │
│  - State                            │
└─────────────────────────────────────┘
```

---

## 🔧 Extension Stack

```tsx
const extensions: Extensions = [
  StarterKit,                    // Bold, Italic, Lists, etc
  Markdown.configure({           // MD parsing
    html: true,
    transformCopiedText: true,
    transformPastedText: true,
  }),
  Placeholder.configure({        // "Type a message..."
    placeholder: "Type @ to mention...",
  }),
  Mention.extend({               // Custom mention logic
    addNodeView() {
      return ReactNodeViewRenderer(MentionNode);
    },
  }).configure({
    suggestion: suggestionConfig,
  }),
];
```

---

## 💡 Mention System — Deep Dive

### Trigger Character

```tsx
suggestion: {
  char: "@",  // or "/" for commands
}
```

⚠️ **Gotcha #1**: Tiptap accepts only **1 character**. Don't try `char: "@@"` or regex.

---

### Suggestion Pipeline

```
User types "@" → onStart() → ReactRenderer mounts Dropdown
                    ↓
User types "tes"  → onUpdate() → Filters items, updates Dropdown
                    ↓
User press ↓/↑    → onKeyDown() → Navigates via imperative handle
                    ↓
User press Enter  → command() → Inserts Mention Node
                    ↓
             onExit() → Destroys Dropdown and Tippy
```

---

### Items Function (Filtering)

```tsx
items: (props) => {
  const { query } = props;
  
  return tools
    .filter((tool) => 
      tool.name.toLowerCase().includes(query.toLowerCase())
    )
    .map((tool) => ({
      id: tool.id,
      type: "tool",
      label: tool.name,
      // ... metadata
    }))
    .slice(0, 10);  // Performance limit
}
```

⚠️ **Gotcha #2**: `items` can return:
- Simple array: `[{ id, label, ... }]`
- Nested array: `[{ category, children: [...] }]`
- Promise (async): `Promise<Item[]>` — useful for API calls

---

### Render Function (Dropdown)

```tsx
render: () => {
  let component: ReactRenderer | null = null;
  let popup: Instance<Props>[] | null = null;

  return {
    onStart: (props) => {
      component = new ReactRenderer(MyDropdown, {
        props,
        editor: props.editor,
      });

      popup = tippy("body", {
        getReferenceClientRect: props.clientRect,
        appendTo: () => document.body,
        content: component.element,
        showOnCreate: true,
        interactive: true,
        trigger: "manual",
        placement: "top-start",
      });
    },

    onUpdate(props) {
      component?.updateProps(props);
      popup?.[0]?.setProps({
        getReferenceClientRect: props.clientRect,
      });
    },

    onKeyDown(props) {
      if (props.event.key === "Escape") {
        popup?.[0]?.hide();
        return true;
      }
      return component?.ref?.onKeyDown(props);
    },

    onExit() {
      popup?.[0]?.destroy();
      component?.destroy();
    },
  };
}
```

---

## 🎈 Tippy.js — Positioning

```tsx
tippy("body", {
  getReferenceClientRect: props.clientRect,  // Function! Not object
  appendTo: () => document.body,             // Portal pattern
  placement: "top-start",                    // top, bottom, left, right + -start/-end
  maxWidth: 400,                             // Prevent overflow
  interactive: true,                         // Allow hover/click
  trigger: "manual",                         // You control show/hide
})
```

⚠️ **Gotcha #3**: `getReferenceClientRect` must be a **function**, not an object.  
Tiptap passes `props.clientRect` which is already a function.

⚠️ **Gotcha #4**: Always `appendTo: document.body` to avoid z-index issues.

---

## ⌨️ Dropdown Component — Keyboard Navigation

### useImperativeHandle Pattern

```tsx
export default forwardRef<
  { onKeyDown: (props: SuggestionProps) => boolean },
  DropdownProps
>(function MyDropdown({ items, command }, ref) {
  const [selectedIndex, setSelectedIndex] = useState(0);

  useImperativeHandle(ref, () => ({
    onKeyDown: (props) => {
      const event = props.event;

      if (event?.key === "ArrowUp") {
        event.preventDefault();
        setSelectedIndex((prev) => 
          (prev - 1 + items.length) % items.length
        );
        return true;  // Handled
      }

      if (event?.key === "ArrowDown") {
        event.preventDefault();
        setSelectedIndex((prev) => (prev + 1) % items.length);
        return true;
      }

      if (event?.key === "Enter") {
        event.preventDefault();
        selectItem(selectedIndex);
        return true;
      }

      return false;  // Not handled
    },
  }));

  const selectItem = (index: number) => {
    const item = items[index];
    if (item) {
      command({ id: item.id, label: item.label });
    }
  };

  return (
    <div className="dropdown">
      {items.map((item, index) => (
        <button
          key={item.id}
          onClick={() => selectItem(index)}
          className={selectedIndex === index ? "selected" : ""}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
});
```

⚠️ **Gotcha #5**: Always `event.preventDefault()` if you handle the event.  
⚠️ **Gotcha #6**: Return `true` = handled, `false` = bubbles to editor.

---

### Scroll Into View

```tsx
const itemRefs = useRef<(HTMLButtonElement | null)[]>([]);

const scrollSelectedIntoView = (index: number) => {
  itemRefs.current[index]?.scrollIntoView({
    behavior: "smooth",
    block: "nearest",    // No unnecessary scroll
    inline: "nearest",
  });
};

useEffect(() => {
  scrollSelectedIntoView(selectedIndex);
}, [selectedIndex]);
```

⚠️ **Gotcha #7**: `block: "nearest"` prevents scroll when item is already visible.

---

## 🎨 Mention Node — Custom Rendering

### Node View (React Component)

```tsx
export default function MentionNode({
  node,
  extension,
  editor,
  getPos,
}: ReactNodeViewProps<HTMLSpanElement>) {
  const { id, label, type } = node.attrs;

  return (
    <NodeViewWrapper 
      as="span" 
      data-id={id} 
      data-type={type}
    >
      <Badge className="bg-accent text-accent-foreground">
        <AtSign className="w-3 h-3" />
        <span>{label}</span>
      </Badge>
    </NodeViewWrapper>
  );
}
```

### Registering Node View

```tsx
Mention.extend({
  addNodeView() {
    return ReactNodeViewRenderer(MentionNode);
  },
  
  addAttributes() {
    return {
      type: { default: "mention" },
      id: { default: "" },
      label: { default: "" },
      // Custom attributes here
    };
  },
})
```

---

## 🔍 Extracting Mentions

### From Editor Content

```tsx
const extractMentions = (editor: Editor) => {
  const mentions: Array<{ id: string; label: string; type: string }> = [];
  
  editor.state.doc.descendants((node) => {
    if (node.type.name === "mention") {
      mentions.push({
        id: node.attrs.id,
        label: node.attrs.label,
        type: node.attrs.type,
      });
    }
  });
  
  return mentions;
};
```

### On Content Change

```tsx
const editor = useEditor({
  extensions,
  content: initialContent,
  onUpdate: ({ editor }) => {
    const mentions = extractMentions(editor);
    console.log("Current mentions:", mentions);
    
    // Update Zustand store
    updatePromptMentions(mentions);
  },
});
```

---

## 🎯 Integration with Workflow Builder

### Prompt Input with Tool Mentions

```tsx
function WorkflowPromptInput() {
  const { tools } = useDiscoverTools();
  
  const editor = useEditor({
    extensions: [
      StarterKit,
      Markdown,
      Mention.configure({
        suggestion: {
          char: "@",
          items: ({ query }) => {
            return tools
              .filter(t => t.name.toLowerCase().includes(query.toLowerCase()))
              .map(t => ({ id: t.id, label: t.name, type: "tool" }))
              .slice(0, 10);
          },
          render: () => createSuggestionRenderer(MentionDropdown),
        },
      }),
    ],
  });

  return (
    <div>
      <EditorContent editor={editor} />
      <Button onClick={() => {
        const mentions = extractMentions(editor);
        generateStep({ 
          objective: editor.getText(),
          mentionedTools: mentions.filter(m => m.type === "tool")
        });
      }}>
        Generate Step
      </Button>
    </div>
  );
}
```

### Step Input with Previous Steps

```tsx
function StepInputEditor({ stepId }: { stepId: string }) {
  const { steps } = useWorkflowStore();
  const previousSteps = steps.filter(s => s.id !== stepId);
  
  const editor = useEditor({
    extensions: [
      StarterKit,
      Mention.configure({
        suggestion: {
          char: "@",
          items: ({ query }) => {
            // Show previous steps as mention options
            return previousSteps
              .filter(s => s.name.toLowerCase().includes(query.toLowerCase()))
              .map(s => ({
                id: `@${s.id}.output`,
                label: s.name,
                type: "step",
              }))
              .slice(0, 10);
          },
          render: () => createSuggestionRenderer(StepMentionDropdown),
        },
      }),
    ],
    onUpdate: ({ editor }) => {
      // Update step input in Zustand
      const content = editor.getHTML();
      updateStepInput(stepId, "prompt", content);
    },
  });

  return <EditorContent editor={editor} />;
}
```

---

## 🐛 Common Gotchas & Solutions

### 1. Tippy not showing
**Problem**: Dropdown doesn't appear  
**Solution**: Check `appendTo: () => document.body` and `trigger: "manual"`

### 2. Keyboard navigation broken
**Problem**: Arrow keys don't work  
**Solution**: Ensure `useImperativeHandle` returns `{ onKeyDown }` and returns `true` when handled

### 3. Mentions not styled
**Problem**: @ mentions look like plain text  
**Solution**: Use `ReactNodeViewRenderer` with custom component in `addNodeView()`

### 4. Multiple @ characters in trigger
**Problem**: Want to use `@@` as trigger  
**Solution**: Not supported! Use single character only, e.g., `/` for commands

### 5. Dropdown position wrong
**Problem**: Dropdown appears in wrong place  
**Solution**: Use `props.clientRect` directly (it's already a function)

### 6. Memory leaks
**Problem**: Editors not cleaning up  
**Solution**: Always call `editor.destroy()` in `useEffect` cleanup

---

## 🎨 Styling Tips

### Editor Container

```tsx
<div className="editor-container">
  <EditorContent editor={editor} className="prose" />
</div>
```

```css
.editor-container {
  border: 1px solid hsl(var(--border));
  border-radius: 0.5rem;
  padding: 1rem;
  min-height: 100px;
}

.ProseMirror {
  outline: none;
}

.ProseMirror p.is-editor-empty:first-child::before {
  color: hsl(var(--muted-foreground));
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}
```

### Mention Badge

```tsx
<Badge 
  variant="secondary" 
  className="mx-0.5 cursor-pointer hover:bg-accent"
>
  <AtSign className="w-3 h-3 mr-1" />
  {label}
</Badge>
```

---

## 🚀 Performance Tips

1. **Limit suggestion items**: `.slice(0, 10)` to show max 10 results
2. **Debounce filtering**: Use `useDebouncedValue` for async item fetching
3. **Memoize extension config**: Create extensions array outside component
4. **Virtual scrolling**: For large dropdown lists (100+ items)
5. **Lazy load node views**: Only render visible mentions

---

## 📚 References

- **Tiptap Docs**: https://tiptap.dev/docs
- **ProseMirror**: https://prosemirror.net/
- **Tippy.js**: https://atomiks.github.io/tippyjs/
- **React NodeView**: https://tiptap.dev/docs/editor/extensions/custom-extensions/node-views/react

---

## ✅ Quick Checklist

- [ ] Install all required packages
- [ ] Create `useEditor` hook with extensions
- [ ] Configure Mention extension with suggestion
- [ ] Implement dropdown component with keyboard navigation
- [ ] Create custom mention node view
- [ ] Add Tippy.js positioning
- [ ] Extract mentions on content change
- [ ] Integrate with Zustand store
- [ ] Test keyboard navigation (↑↓ Enter Escape)
- [ ] Style mentions and dropdown
- [ ] Handle cleanup (`editor.destroy()`)

---

**Built with** ❤️ **for natural language workflow editing**

