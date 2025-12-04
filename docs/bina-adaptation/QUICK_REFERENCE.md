# Quick Reference Cheat Sheet
# Fincept Terminal → Bina Analytics Control Console

**Purpose:** Single-page quick reference for common tasks and patterns  
**Version:** 1.0  
**Date:** December 2024

---

## Common File Paths

| Path | Purpose |
|------|---------|
| `src/main.tsx` | React entry point, wraps App with AuthProvider |
| `src/App.tsx` | Main app component, handles screen routing |
| `src/App.css` | Global styles and CSS variables |
| `index.html` | HTML entry point, sets page title |
| **Components** | |
| `src/components/dashboard/DashboardScreen.tsx` | Main dashboard container with tabs |
| `src/components/tabs/` | All feature tabs (30+ files) |
| `src/components/ui/` | shadcn/ui components (40+ primitives) |
| `src/components/auth/` | Authentication screens |
| `src/components/common/` | Shared components (Header, Footer, etc.) |
| **Contexts** | |
| `src/contexts/AuthContext.tsx` | Authentication state management |
| `src/contexts/ThemeContext.tsx` | Theme and appearance settings |
| `src/contexts/DataSourceContext.tsx` | Data source registry |
| `src/contexts/NavigationContext.tsx` | Navigation state |
| **Services** | |
| `src/services/supabase/` | Supabase integration (Bina) |
| `src/services/llmApi.ts` | LLM provider integration |
| `src/services/workflowService.ts` | Workflow management |
| `src/services/mcpManager.ts` | MCP server management |
| `src/services/dataSourceRegistry.ts` | Data source registry |
| **Configuration** | |
| `package.json` | npm dependencies and scripts |
| `vite.config.ts` | Vite build configuration |
| `tsconfig.json` | TypeScript configuration |
| `src-tauri/tauri.conf.json` | Tauri app configuration |
| `src-tauri/Cargo.toml` | Rust dependencies |
| `.env` | Environment variables (not committed) |
| **Tauri Backend** | |
| `src-tauri/src/lib.rs` | Rust backend entry point |
| `src-tauri/src/commands/` | Tauri command handlers |
| `src-tauri/resources/scripts/` | Bundled Python scripts |

---

## Key Commands

```bash
# Development
npm run dev                    # Start Vite dev server (port 1420)
npm run tauri dev              # Start Tauri app with hot reload

# Building
npm run build                  # Build frontend (TypeScript check + Vite build)
npm run tauri build            # Build desktop app installers
npm run preview                # Preview production build

# Utilities
tsc --noEmit                   # Type check without building
npm run bump-version           # Increment version number

# Tauri CLI
npx tauri info                 # System information
npx tauri icon icon.png        # Generate icons from PNG

# Cleaning
rm -rf dist node_modules src-tauri/target
npm install                    # Fresh install

# Dependencies
npm install <package>          # Add dependency
npm uninstall <package>        # Remove dependency
npm update                     # Update all dependencies
npx depcheck                   # Find unused dependencies

# Rust (in src-tauri/)
cargo build                    # Build Rust backend
cargo check                    # Type check without building
cargo clean                    # Clean build artifacts
cargo add <crate>              # Add Rust dependency
```

---

## Important Code Patterns

### Add a New Tab

**1. Create component file:**
```typescript
// src/components/tabs/MyNewTab.tsx
import React from 'react';

const MyNewTab: React.FC = () => {
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold text-white">My New Feature</h1>
      {/* Tab content */}
    </div>
  );
};

export default MyNewTab;
```

**2. Register in DashboardScreen:**
```typescript
// src/components/dashboard/DashboardScreen.tsx

// Import
import MyNewTab from '@/components/tabs/MyNewTab';

// Add tab trigger (inside TabsList)
<TabsTrigger value="my-new-tab">
  <Icon className="w-4 h-4 mr-2" />
  My Feature
</TabsTrigger>

// Add tab content (after other TabsContent)
<TabsContent value="my-new-tab">
  <MyNewTab />
</TabsContent>
```

---

### Create a New API Call (Supabase)

**Pattern for fetching data:**
```typescript
// src/services/supabase/myNewService.ts
import { supabase } from './supabaseClient';

export interface MyData {
  id: string;
  name: string;
  created_at: string;
}

export async function fetchMyData(): Promise<MyData[]> {
  const { data, error } = await supabase
    .from('my_table')
    .select('*')
    .order('created_at', { ascending: false });
  
  if (error) throw error;
  return data;
}

export async function createMyData(name: string): Promise<string> {
  const { data, error } = await supabase
    .from('my_table')
    .insert({ name })
    .select()
    .single();
  
  if (error) throw error;
  return data.id;
}

export async function updateMyData(id: string, updates: Partial<MyData>): Promise<void> {
  const { error } = await supabase
    .from('my_table')
    .update(updates)
    .eq('id', id);
  
  if (error) throw error;
}

export async function deleteMyData(id: string): Promise<void> {
  const { error } = await supabase
    .from('my_table')
    .delete()
    .eq('id', id);
  
  if (error) throw error;
}

// Real-time subscription
export function subscribeToMyData(callback: (data: MyData) => void) {
  return supabase
    .channel('my_data_channel')
    .on('postgres_changes', {
      event: '*',
      schema: 'public',
      table: 'my_table'
    }, (payload) => {
      callback(payload.new as MyData);
    })
    .subscribe();
}
```

**Usage in component:**
```typescript
import { fetchMyData, subscribeToMyData } from '@/services/supabase/myNewService';

const MyComponent = () => {
  const [data, setData] = useState<MyData[]>([]);

  useEffect(() => {
    // Initial fetch
    fetchMyData().then(setData);
    
    // Subscribe to updates
    const subscription = subscribeToMyData((newData) => {
      setData(prev => [newData, ...prev]);
    });
    
    return () => subscription.unsubscribe();
  }, []);

  return (/* ... */);
};
```

---

### Add a New Component (shadcn/ui)

**Install from shadcn:**
```bash
npx shadcn-ui@latest add <component-name>
```

**Example:**
```bash
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
```

**Use in component:**
```typescript
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

<Dialog open={isOpen} onOpenChange={setIsOpen}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>My Dialog</DialogTitle>
    </DialogHeader>
    <div>Dialog content</div>
  </DialogContent>
</Dialog>
```

---

### Modify State in Context

**Pattern for using contexts:**
```typescript
// In component
import { useAuth } from '@/contexts/AuthContext';

const MyComponent = () => {
  const { session, user, signOut } = useAuth();

  if (!session) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <p>Welcome {user?.email}</p>
      <button onClick={signOut}>Sign Out</button>
    </div>
  );
};
```

**Available contexts:**
- `useAuth()` - Authentication state
- `useTheme()` - Theme settings
- `useNavigation()` - Navigation state
- `useDataSource()` - Data source registry

---

### Call Tauri Command (Rust Backend)

**Define command in Rust:**
```rust
// src-tauri/src/lib.rs or src-tauri/src/commands/my_command.rs

#[tauri::command]
fn my_command(arg1: String, arg2: i32) -> Result<String, String> {
    println!("Called my_command with: {} {}", arg1, arg2);
    Ok("Success".to_string())
}

// Register in lib.rs
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![my_command])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

**Call from frontend:**
```typescript
import { invoke } from '@tauri-apps/api/core';

const result = await invoke<string>('my_command', {
  arg1: 'hello',
  arg2: 42
});
console.log(result);  // "Success"
```

**Error handling:**
```typescript
try {
  const result = await invoke('my_command', { arg1: 'test', arg2: 10 });
  console.log('Success:', result);
} catch (error) {
  console.error('Tauri command failed:', error);
}
```

---

### Add Environment Variable

**1. Add to `.env`:**
```bash
VITE_MY_NEW_VAR=my_value
```

**2. Use in code:**
```typescript
const myVar = import.meta.env.VITE_MY_NEW_VAR;
console.log(myVar);  // "my_value"
```

**3. Add TypeScript type (optional):**
```typescript
// src/vite-env.d.ts
interface ImportMetaEnv {
  readonly VITE_MY_NEW_VAR: string;
}
```

**Note:** Must prefix with `VITE_` for Vite to expose to frontend!

**Restart dev server after adding env vars:**
```bash
# Stop dev server (Ctrl+C)
npm run dev
```

---

## UI Component Examples

### Button Variants

```tsx
import { Button } from '@/components/ui/button';

<Button>Default</Button>
<Button variant="destructive">Destructive</Button>
<Button variant="outline">Outline</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>

<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
<Button size="icon"><Icon /></Button>
```

---

### Card with Header

```tsx
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from '@/components/ui/card';

<Card>
  <CardHeader>
    <CardTitle>Card Title</CardTitle>
    <CardDescription>Card description</CardDescription>
  </CardHeader>
  <CardContent>
    <p>Card content goes here</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

---

### Dialog (Modal)

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog';

<Dialog>
  <DialogTrigger asChild>
    <Button>Open Dialog</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Dialog Title</DialogTitle>
    </DialogHeader>
    <div>Dialog content</div>
  </DialogContent>
</Dialog>
```

---

### Form Input

```tsx
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';

<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input
    id="email"
    type="email"
    placeholder="you@example.com"
    value={email}
    onChange={(e) => setEmail(e.target.value)}
  />
</div>
```

---

### Toast Notification

```tsx
import { toast } from 'sonner';

// Success
toast.success('Task created successfully');

// Error
toast.error('Failed to create task');

// Info
toast.info('Task is running');

// Loading
const toastId = toast.loading('Creating task...');
// Later:
toast.success('Task created', { id: toastId });
```

---

### Loading State

```tsx
const MyComponent = () => {
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(data => {
      setData(data);
      setLoading(false);
    });
  }, []);

  if (loading) {
    return (
      <div className="flex items-center justify-center h-full">
        <div className="animate-spin rounded-full h-8 w-8 border-2 border-blue-500 border-t-transparent" />
      </div>
    );
  }

  return <div>{/* Render data */}</div>;
};
```

---

## Styling Patterns

### Tailwind Classes

```tsx
// Layout
<div className="flex items-center justify-between">
<div className="grid grid-cols-3 gap-4">
<div className="space-y-4">  {/* Vertical spacing */}
<div className="space-x-2">  {/* Horizontal spacing */}

// Colors (Bina theme)
<div className="bg-zinc-900 text-white">  {/* Dark background */}
<div className="text-zinc-400">  {/* Muted text */}
<div className="border-zinc-800">  {/* Border */}
<div className="text-blue-500">  {/* Bina accent */}

// Typography
<h1 className="text-2xl font-bold">
<p className="text-sm text-zinc-500">
<span className="font-mono text-xs">  {/* Code-style */}

// Spacing
<div className="p-6">  {/* Padding all sides */}
<div className="px-4 py-2">  {/* Padding horizontal/vertical */}
<div className="mb-4">  {/* Margin bottom */}

// Hover effects
<button className="hover:bg-zinc-800 transition-colors">

// Responsive
<div className="hidden md:block">  {/* Hidden on mobile */}
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```

---

### CSS Variables (from App.css)

```css
/* Use in custom styles */
.my-element {
  color: var(--color-bina-accent);
  background: var(--color-background);
  border: 1px solid var(--color-border);
}
```

---

## Debugging Tips

### Console Logging

```typescript
// Structured logging
console.log('[MyComponent] Loading data...');
console.log('[MyComponent] Data:', data);
console.error('[MyComponent] Error:', error);

// Conditional logging
if (import.meta.env.DEV) {
  console.log('Debug:', debugData);
}
```

---

### React DevTools

1. Open browser at `http://localhost:1420`
2. Press F12
3. Go to Components or Profiler tab
4. Inspect component state/props

---

### Check Environment Variables

```typescript
console.log('Environment:', import.meta.env.MODE);  // 'development' or 'production'
console.log('Supabase URL:', import.meta.env.VITE_SUPABASE_URL);
console.log('All env vars:', import.meta.env);
```

---

### Tauri Console

**Rust `println!()` output appears in terminal running `npm run tauri dev`**

```rust
println!("Debug: my_command called with arg: {}", arg);
```

---

## Common Errors & Solutions

### Error: "Module not found: '@/components/X'"

**Solution:** Check `tsconfig.json` has path alias:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

---

### Error: "invoke is not defined"

**Solution:** Tauri APIs only work in Tauri app, not browser.

**Workaround for dev:**
```typescript
const isTauri = typeof window !== 'undefined' && (window as any).__TAURI__;

if (isTauri) {
  // Use Tauri API
  const result = await invoke('my_command');
} else {
  // Browser fallback
  console.warn('Tauri API not available in browser');
}
```

---

### Error: "Supabase error: Invalid API key"

**Check:**
1. `.env` has correct `VITE_SUPABASE_ANON_KEY`
2. Restart dev server after changing `.env`
3. Verify key in Supabase dashboard

---

### Error: TypeScript type errors on build

**Quick fix (temporary):**
```typescript
// Add @ts-ignore above problematic line
// @ts-ignore
const result = someFunction();
```

**Better fix:** Define proper types
```typescript
interface MyType {
  id: string;
  name: string;
}
const result: MyType = someFunction();
```

---

## Performance Tips

### Lazy Load Tabs

```typescript
import { lazy, Suspense } from 'react';

const MyHeavyTab = lazy(() => import('@/components/tabs/MyHeavyTab'));

<TabsContent value="my-heavy-tab">
  <Suspense fallback={<LoadingSpinner />}>
    <MyHeavyTab />
  </Suspense>
</TabsContent>
```

---

### Memoize Expensive Calculations

```typescript
import { useMemo } from 'react';

const MyComponent = ({ data }) => {
  const processedData = useMemo(() => {
    return data.map(item => heavyProcessing(item));
  }, [data]);

  return <div>{/* Use processedData */}</div>;
};
```

---

### Debounce Search Input

```typescript
import { useState, useEffect } from 'react';

const SearchComponent = () => {
  const [search, setSearch] = useState('');
  const [debouncedSearch, setDebouncedSearch] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedSearch(search);
    }, 500);
    
    return () => clearTimeout(timer);
  }, [search]);

  useEffect(() => {
    if (debouncedSearch) {
      performSearch(debouncedSearch);
    }
  }, [debouncedSearch]);

  return <Input value={search} onChange={(e) => setSearch(e.target.value)} />;
};
```

---

## Git Workflow

```bash
# Create feature branch
git checkout -b feature/my-new-feature

# Make changes, then:
git add .
git commit -m "Add my new feature"

# Push to GitHub
git push origin feature/my-new-feature

# Create PR on GitHub

# After merge, update local main
git checkout main
git pull origin main
git branch -d feature/my-new-feature
```

---

## Useful VS Code Extensions

- **rust-analyzer** - Rust language support
- **Tauri** - Tauri-specific tools
- **ES7+ React/Redux/React-Native snippets** - React snippets
- **Tailwind CSS IntelliSense** - Tailwind autocomplete
- **Prettier** - Code formatting
- **ESLint** - Linting

---

## Related Documentation

- [ARCHITECTURE_ANALYSIS.md](./ARCHITECTURE_ANALYSIS.md) - Deep dive into codebase structure
- [INTEGRATION_POINTS.md](./INTEGRATION_POINTS.md) - Backend integration guide
- [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md) - Step-by-step customization
- [DEV_WORKFLOW.md](./DEV_WORKFLOW.md) - Development setup and workflows
- [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) - Phased implementation plan

---

**Document Version:** 1.0  
**Last Updated:** December 2024  
**Tip:** Print this page and keep it handy during development!
