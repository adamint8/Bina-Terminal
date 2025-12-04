# Development Workflow Guide
# Fincept Terminal → Bina Analytics Control Console

**Purpose:** Practical development information for working with the codebase  
**Version:** 1.0  
**Date:** December 2024

---

## Table of Contents
1. [Setup Instructions](#setup-instructions)
2. [Build Process](#build-process)
3. [Project Configuration](#project-configuration)
4. [Development Best Practices](#development-best-practices)
5. [Testing Approach](#testing-approach)
6. [Debugging](#debugging)
7. [Common Issues](#common-issues)

---

## Setup Instructions

### Prerequisites

#### **System Requirements**

**Operating Systems:**
- Windows 10/11 (64-bit)
- macOS 10.15+ (Catalina or later)
- Linux (Ubuntu 20.04+, Fedora 35+, or similar)

**Required Software:**
- **Node.js** 18.x or 20.x (LTS recommended)
- **Rust** 1.70+ (for Tauri)
- **Git** 2.30+

**Optional:**
- **Python** 3.11+ (if modifying analytics scripts)
- **VS Code** (recommended IDE with Tauri extension)

#### **Install Node.js**

```bash
# Using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
nvm install 20
nvm use 20

# Or download from nodejs.org
# https://nodejs.org/en/download/
```

**Verify:**
```bash
node --version  # Should be v20.x.x
npm --version   # Should be 10.x.x
```

#### **Install Rust**

```bash
# Unix/Linux/macOS
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Windows
# Download from https://rustup.rs/
```

**Verify:**
```bash
rustc --version  # Should be 1.70+
cargo --version  # Should be 1.70+
```

#### **Install System Dependencies**

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install -y \
  libwebkit2gtk-4.0-dev \
  build-essential \
  curl \
  wget \
  libssl-dev \
  libgtk-3-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev
```

**macOS:**
```bash
xcode-select --install
brew install pkg-config
```

**Windows:**
- Install [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
- Install [WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/) (usually pre-installed on Windows 11)

---

### Clone and Install

#### **1. Clone Repository**

```bash
git clone https://github.com/adamint8/Bina-Terminal.git
cd Bina-Terminal/fincept-terminal-desktop
```

#### **2. Install Dependencies**

```bash
npm install
```

**This will install:**
- 90+ npm packages (~500MB in `node_modules/`)
- TypeScript, React, Vite, Tauri CLI
- UI libraries (Radix UI, Tailwind, Lucide)
- LLM libraries (LangChain, Ollama)
- Chart libraries (Recharts, ReactFlow)

**Expected output:**
```
added 1234 packages in 45s
```

#### **3. Environment Variables**

**Create `.env` file** in `fincept-terminal-desktop/`:

```bash
# .env
# Bina Analytics Backend
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# n8n Integration
VITE_N8N_WEBHOOK_URL=https://your-n8n.com/webhook
VITE_N8N_API_URL=https://your-n8n.com/api/v1
VITE_N8N_API_KEY=your-api-key

# LLM Providers (optional)
VITE_OPENAI_API_KEY=sk-...
VITE_ANTHROPIC_API_KEY=sk-ant-...
VITE_GOOGLE_API_KEY=...
VITE_GROQ_API_KEY=gsk_...

# Ollama (local LLM)
VITE_OLLAMA_BASE_URL=http://localhost:11434

# Data Sources (optional)
VITE_ALPHA_VANTAGE_API_KEY=...
VITE_POLYGON_API_KEY=...
```

**Notes:**
- Prefix all environment variables with `VITE_` for Vite to expose them to frontend
- Never commit `.env` to version control
- Add `.env` to `.gitignore` (already done)

#### **4. Verify Setup**

```bash
# Check if Tauri CLI is available
npm run tauri --version
# Should output: tauri-cli 2.x.x

# Check if Rust is set up correctly
cd src-tauri
cargo --version
cd ..
```

---

## Build Process

### Development Build

#### **Start Development Server**

**Terminal 1:** Start Vite dev server (frontend)
```bash
npm run dev
```

**Expected output:**
```
  VITE v7.0.4  ready in 1234 ms

  ➜  Local:   http://localhost:1420/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**Terminal 2 (Optional):** Start Tauri dev (desktop app)
```bash
npm run tauri dev
```

**Expected output:**
```
    Finished dev [unoptimized + debuginfo] target(s) in 45.67s
    Running `target/debug/FinceptTerminal.exe`
```

**This will:**
1. Compile Rust backend (~3-5 minutes first time, ~10s incremental)
2. Launch desktop window with WebView
3. Connect to Vite dev server at `localhost:1420`
4. Enable hot module replacement (HMR)

**Development Mode Features:**
- ✅ Hot reload for React components
- ✅ Fast refresh without losing state
- ✅ Source maps for debugging
- ✅ Detailed error messages
- ✅ React DevTools support

#### **Development Without Tauri**

**For faster frontend-only development:**

```bash
npm run dev
```

**Then open browser:**
```
http://localhost:1420
```

**Limitations:**
- No Tauri API access (invoke commands, file system, etc.)
- No Rust backend
- No Python script execution
- Good for UI-only work

---

### Production Build

#### **Build Frontend**

```bash
npm run build
```

**This runs:**
1. `tsc` - TypeScript type checking
2. `vite build` - Production bundle creation

**Expected output:**
```
vite v7.0.4 building for production...
✓ 1234 modules transformed.
dist/index.html                   5.67 kB │ gzip: 2.34 kB
dist/assets/index-abc123.js     567.89 kB │ gzip: 189.01 kB
dist/assets/index-xyz789.css     45.67 kB │ gzip: 12.34 kB

✓ built in 45.67s
```

**Output location:** `dist/` directory

#### **Build Desktop App**

```bash
npm run tauri build
```

**This will:**
1. Run `npm run build` (if not already built)
2. Compile Rust in release mode (~5-10 minutes)
3. Bundle assets (Python scripts, icons, etc.)
4. Create installers for your platform

**Expected output:**
```
    Finished release [optimized] target(s) in 7m 34s
    Bundling FinceptTerminal.exe (/src-tauri/target/release)
    
    Finished 2 bundles at:
    - /src-tauri/target/release/bundle/nsis/FinceptTerminal_3.0.11_x64-setup.exe
    - /src-tauri/target/release/bundle/msi/FinceptTerminal_3.0.11_x64_en-US.msi
```

**Build artifacts:**
- **Windows:** `.exe` (NSIS installer), `.msi` (Windows Installer)
- **macOS:** `.dmg`, `.app`
- **Linux:** `.AppImage`, `.deb`, `.rpm`

**Locations:**
- Windows: `src-tauri/target/release/bundle/nsis/`
- macOS: `src-tauri/target/release/bundle/dmg/`
- Linux: `src-tauri/target/release/bundle/appimage/`

---

### Build Optimization

#### **Reduce Bundle Size**

**1. Analyze bundle:**
```bash
npm run build
npx vite-bundle-visualizer
```

**2. Code splitting** is already configured in `vite.config.ts`:
```typescript
manualChunks: {
  'react-vendor': ['react', 'react-dom', 'react-router-dom'],
  'chart-vendor': ['recharts', 'lightweight-charts'],
  'ui-vendor': ['@radix-ui/react-*'],
  'flow-vendor': ['reactflow'],
  'tauri-vendor': ['@tauri-apps/*'],
}
```

**3. Remove unused dependencies:**
```bash
npx depcheck
npm uninstall <unused-package>
```

#### **Faster Rust Compilation**

**File:** `src-tauri/Cargo.toml`

**Add to `[profile.dev]`:**
```toml
[profile.dev]
opt-level = 1  # Light optimization for faster builds
```

**Use `cargo-watch` for auto-rebuild:**
```bash
cargo install cargo-watch
cd src-tauri
cargo watch -x build
```

---

## Project Configuration

### Key Configuration Files

#### **1. package.json**

**Location:** `fincept-terminal-desktop/package.json`

**Important sections:**

```json
{
  "name": "fincept-terminal-desktop",
  "version": "3.0.11",
  "type": "module",  // ES modules
  "scripts": {
    "dev": "vite",  // Frontend dev server
    "build": "tsc && vite build",  // Production build
    "preview": "vite preview",  // Preview production build
    "tauri": "tauri",  // Tauri CLI
    "bump-version": "node scripts/bump-version.js"  // Version increment
  },
  "dependencies": {
    // 90+ packages (see package.json for full list)
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2",  // Tauri build tools
    "typescript": "~5.8.3",
    "vite": "^7.0.4"
  }
}
```

**Custom npm scripts to add:**

```json
{
  "scripts": {
    "dev": "vite",
    "dev:tauri": "tauri dev",  // ADD: Explicit Tauri dev
    "build": "tsc && vite build",
    "build:tauri": "tauri build",  // ADD: Explicit Tauri build
    "preview": "vite preview",
    "tauri": "tauri",
    "type-check": "tsc --noEmit",  // ADD: Type check only
    "lint": "eslint src --ext ts,tsx",  // ADD: Linting (if ESLint configured)
    "format": "prettier --write src/**/*.{ts,tsx}",  // ADD: Code formatting
    "clean": "rm -rf dist node_modules src-tauri/target",  // ADD: Clean build
    "bump-version": "node scripts/bump-version.js"
  }
}
```

#### **2. vite.config.ts**

**Location:** `fincept-terminal-desktop/vite.config.ts`

**Current configuration:**
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import path from 'path'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),  // Path alias
    },
  },
  server: {
    port: 1420,
    strictPort: true,  // Fail if port is unavailable
    proxy: {
      '/api': {  // Proxy API requests to backend
        target: 'https://finceptbackend.share.zrok.io',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      }
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: { /* code splitting */ }
      }
    },
    chunkSizeWarningLimit: 1000,  // Suppress warnings for large chunks
  },
})
```

**To modify for Bina:**

```typescript
export default defineConfig({
  // ... existing config
  server: {
    port: 1420,
    strictPort: true,
    proxy: {
      '/api': {
        target: process.env.VITE_SUPABASE_URL || 'http://localhost:54321',  // CHANGE
        changeOrigin: true,
      }
    }
  },
})
```

#### **3. tsconfig.json**

**Location:** `fincept-terminal-desktop/tsconfig.json`

**Current configuration:**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,  // Strict type checking
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]  // Path alias for imports
    },
    "skipLibCheck": true,
    "moduleResolution": "bundler"
  },
  "include": ["src"],
  "exclude": ["src/services/duckdbService.ts"]
}
```

**Key settings:**
- `strict: true` - Enables all strict type-checking options
- `baseUrl` + `paths` - Allows `import { X } from '@/components/X'`
- `exclude` - Exclude files with type errors (temporary workaround)

#### **4. tauri.conf.json**

**Location:** `fincept-terminal-desktop/src-tauri/tauri.conf.json`

**Critical sections:**

```json
{
  "productName": "FinceptTerminal",  // CHANGE for Bina
  "version": "3.0.11",  // CHANGE for Bina
  "identifier": "com.fincept.terminal",  // CHANGE for Bina (UNIQUE)
  "build": {
    "beforeDevCommand": "npm run dev",  // Command before dev
    "devUrl": "http://localhost:1420",  // Dev server URL
    "beforeBuildCommand": "npm run build",  // Command before build
    "frontendDist": "../dist"  // Frontend build output
  },
  "app": {
    "windows": [{
      "title": "FinceptTerminal",  // CHANGE
      "width": 1400,
      "height": 900,
      "minWidth": 1024,
      "minHeight": 768,
      "resizable": true,
      "fullscreen": false,
      "center": true
    }]
  },
  "bundle": {
    "icon": ["icons/32x32.png", "icons/icon.ico", "icons/icon.icns"],
    "resources": ["resources/scripts"],  // Bundled Python scripts
    "externalBin": ["binaries/python-interpreter"]  // Python interpreter
  },
  "plugins": {
    "updater": {
      "endpoints": [
        "https://github.com/Fincept-Corporation/FinceptTerminal/releases/latest/download/latest.json"
      ]  // CHANGE for Bina
    }
  }
}
```

**For Bina, change:**
- `productName` → `"BinaAnalytics"`
- `identifier` → `"com.bina.analytics"`
- `version` → `"1.0.0"`
- `updater.endpoints` → Bina GitHub releases URL

#### **5. components.json**

**Location:** `fincept-terminal-desktop/components.json`

**shadcn/ui configuration:**
```json
{
  "style": "default",
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "src/App.css",
    "baseColor": "zinc",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

**To customize theme colors**, edit `src/App.css` CSS variables.

---

### Tauri-Specific Configuration

#### **Capabilities (Permissions)**

**Location:** `src-tauri/capabilities/default.json`

**Purpose:** Define which Tauri APIs frontend can access

```json
{
  "identifier": "default",
  "windows": ["main"],
  "permissions": [
    "http:default",
    "shell:default",
    "sql:default",
    "dialog:default",
    "process:default",
    "updater:default",
    "opener:default"
  ]
}
```

**Add new permission:**
```json
{
  "permissions": [
    "core:default",
    "http:default",
    "fs:read-all",  // ADD: File system read
    "fs:write-all"  // ADD: File system write
  ]
}
```

**Note:** Tauri 2.0 uses fine-grained permissions for security.

#### **Cargo.toml (Rust Dependencies)**

**Location:** `src-tauri/Cargo.toml`

**Current dependencies:**
```toml
[dependencies]
tauri = { version = "2", features = ["..."] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
reqwest = { version = "0.11", features = ["json"] }
# ... more Rust crates
```

**To add new Rust dependency:**
```bash
cd src-tauri
cargo add <crate-name>
```

**Example:**
```bash
cargo add uuid --features v4,serde
```

---

## Development Best Practices

### Code Organization

#### **File Naming Conventions**

- **Components:** PascalCase (e.g., `DashboardScreen.tsx`)
- **Services:** camelCase (e.g., `authApi.tsx`, `workflowService.ts`)
- **Hooks:** camelCase with `use` prefix (e.g., `useAuth.ts`)
- **Types:** PascalCase (e.g., `types/Workflow.ts`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `APP_VERSION`)

#### **Import Order**

```typescript
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { invoke } from '@tauri-apps/api/core';

// 2. UI components
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

// 3. Local components
import DashboardTab from '@/components/tabs/DashboardTab';

// 4. Contexts/Hooks
import { useAuth } from '@/contexts/AuthContext';
import { useTheme } from '@/contexts/ThemeContext';

// 5. Services
import { binaAgentService } from '@/services/binaAgentService';

// 6. Types
import type { AgentTask } from '@/types';

// 7. Assets/Styles
import './styles.css';
```

#### **Component Structure**

```typescript
// Imports
import React, { useState, useEffect } from 'react';

// Types
interface MyComponentProps {
  title: string;
  onSave: (data: any) => void;
}

// Component
const MyComponent: React.FC<MyComponentProps> = ({ title, onSave }) => {
  // State
  const [data, setData] = useState<any>(null);
  const [loading, setLoading] = useState(false);

  // Effects
  useEffect(() => {
    loadData();
  }, []);

  // Handlers
  const loadData = async () => {
    setLoading(true);
    try {
      // Logic
    } catch (error) {
      console.error('Failed to load data:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleSave = () => {
    onSave(data);
  };

  // Render
  return (
    <div>
      {loading ? <Spinner /> : <Content data={data} onSave={handleSave} />}
    </div>
  );
};

// Export
export default MyComponent;
```

### TypeScript Best Practices

#### **Use Explicit Types**

```typescript
// ✅ Good
const [user, setUser] = useState<User | null>(null);
const fetchData = async (): Promise<Data[]> => { /* ... */ };

// ❌ Avoid
const [user, setUser] = useState(null);  // Type is 'null'
const fetchData = async () => { /* ... */ };  // Type is 'Promise<any>'
```

#### **Define Interfaces**

```typescript
// Define interfaces for props and state
interface TaskCardProps {
  task: AgentTask;
  onView: (id: string) => void;
  onDelete: (id: string) => void;
}

// Use interfaces consistently
const TaskCard: React.FC<TaskCardProps> = ({ task, onView, onDelete }) => {
  // ...
};
```

#### **Avoid `any`**

```typescript
// ❌ Bad
const data: any = await fetchData();

// ✅ Good
interface ApiResponse {
  success: boolean;
  data: Task[];
}
const response: ApiResponse = await fetchData();
```

---

## Testing Approach

### Current State

**The Fincept Terminal codebase does NOT have a comprehensive test suite.**

**Testing Strategy for Bina:**

1. **Manual Testing** (primary approach for now)
2. **Add tests incrementally** for critical features
3. **Focus on integration tests** over unit tests

---

### Manual Testing Checklist

#### **Before Each Commit:**

- [ ] Run `npm run build` - Ensure no TypeScript errors
- [ ] Test affected features in dev mode
- [ ] Check browser console for errors
- [ ] Verify UI renders correctly

#### **Before Each Release:**

- [ ] Test all major workflows end-to-end
- [ ] Test on all target platforms (Windows, macOS, Linux)
- [ ] Verify Supabase integration
- [ ] Test n8n workflow triggers
- [ ] Check error handling (offline mode, API failures)

---

### Adding Tests (Optional)

#### **Setup Testing Framework**

```bash
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom
```

**Create test config:** `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

**Create setup file:** `src/test/setup.ts`

```typescript
import '@testing-library/jest-dom';
```

**Example test:** `src/components/tabs/ResearchTasksTab.test.tsx`

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import ResearchTasksTab from './ResearchTasksTab';

// Mock services
vi.mock('@/services/binaAgentService', () => ({
  binaAgentService: {
    listTasks: vi.fn(() => Promise.resolve([])),
    subscribeToTasks: vi.fn(() => ({ unsubscribe: vi.fn() })),
  },
}));

describe('ResearchTasksTab', () => {
  it('renders without crashing', async () => {
    render(<ResearchTasksTab />);
    await waitFor(() => {
      expect(screen.getByText('Research Tasks')).toBeInTheDocument();
    });
  });

  it('displays loading state initially', () => {
    render(<ResearchTasksTab />);
    expect(screen.getByRole('status')).toBeInTheDocument();  // Spinner
  });
});
```

**Run tests:**
```bash
npm run test
```

---

## Debugging

### Frontend Debugging

#### **Browser DevTools**

1. **Open app in browser:** `http://localhost:1420`
2. **Open DevTools:** F12 or Ctrl+Shift+I
3. **Use Console tab:** View `console.log()` output and errors
4. **Use Network tab:** Monitor API requests
5. **Use React DevTools:** Install extension for component inspection

#### **VS Code Debugging**

**Create `.vscode/launch.json`:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://localhost:1420",
      "webRoot": "${workspaceFolder}/src"
    }
  ]
}
```

**Set breakpoints** in `.tsx` files and press F5.

---

### Tauri/Rust Debugging

#### **Console Output**

**Tauri logs output** to terminal running `npm run tauri dev`.

**Add debug prints in Rust:**

```rust
// src-tauri/src/lib.rs
#[tauri::command]
fn my_command() -> Result<String, String> {
    println!("Debug: my_command called");  // Prints to terminal
    Ok("Success".to_string())
}
```

#### **Rust Debugger (VS Code)**

**Install extension:** `rust-analyzer`

**Create `.vscode/launch.json`:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Tauri Development Debug",
      "cargo": {
        "args": [
          "build",
          "--manifest-path=./src-tauri/Cargo.toml",
          "--no-default-features"
        ]
      },
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

**Set breakpoints** in `.rs` files and press F5.

---

### Common Debugging Scenarios

#### **"Module not found" Error**

**Error:** `Cannot find module '@/components/X'`

**Solution:** Check `tsconfig.json` has correct path alias:
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

#### **"invoke is not defined" Error**

**Error:** `invoke is not defined` in browser console

**Solution:** Tauri APIs only work in Tauri WebView, not browser.

**Workaround:** Add mock for browser development:
```typescript
import { invoke } from '@tauri-apps/api/core';

// Mock invoke for browser
const mockInvoke = (cmd: string, args?: any) => {
  console.warn(`invoke('${cmd}') called in browser mode`);
  return Promise.resolve({});
};

export const safeInvoke = typeof window !== 'undefined' && (window as any).__TAURI__
  ? invoke
  : mockInvoke;
```

#### **Supabase Connection Error**

**Error:** `Failed to connect to Supabase`

**Check:**
1. `.env` has correct `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`
2. Restart dev server after adding env variables
3. Check Supabase project is running
4. Verify network connectivity

---

## Common Issues

### Build Issues

#### **Issue: `npm run build` fails with TypeScript errors**

**Solution:**
1. Fix TypeScript errors shown in output
2. If errors are in external libraries, add to `tsconfig.json` exclude:
   ```json
   {
     "exclude": ["src/services/problematic-file.ts"]
   }
   ```
3. As last resort, disable strict mode temporarily:
   ```json
   {
     "compilerOptions": {
       "strict": false
     }
   }
   ```

#### **Issue: Rust compilation fails**

**Error:** `error: linking with 'cc' failed`

**Solution (Linux):**
```bash
sudo apt install build-essential
```

**Error:** `error: toolchain 'stable-x86_64-unknown-linux-gnu' is not installed`

**Solution:**
```bash
rustup install stable
rustup default stable
```

#### **Issue: Out of memory during build**

**Error:** `JavaScript heap out of memory`

**Solution:**
```bash
export NODE_OPTIONS="--max-old-space-size=8192"
npm run build
```

---

### Runtime Issues

#### **Issue: App window is blank**

**Possible causes:**
1. Vite dev server not running
2. Wrong port in `tauri.conf.json`
3. Frontend build failed

**Solution:**
1. Check terminal for Vite errors
2. Verify `devUrl` in `tauri.conf.json` matches Vite port
3. Delete `dist/` and rebuild: `npm run build`

#### **Issue: Hot reload not working**

**Solution:**
1. Restart Vite dev server
2. Check firewall not blocking port 1420
3. Disable browser extensions interfering with WebSocket

---

### Dependency Issues

#### **Issue: `npm install` fails**

**Error:** `Cannot resolve dependency tree`

**Solution:**
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and lock file
rm -rf node_modules package-lock.json

# Reinstall
npm install
```

**Error:** `EACCES: permission denied`

**Solution (Linux/macOS):**
```bash
sudo chown -R $USER ~/.npm
npm install
```

---

## Quick Commands Reference

```bash
# Development
npm run dev                    # Start Vite dev server
npm run tauri dev              # Start Tauri app (includes Vite)

# Building
npm run build                  # Build frontend only
npm run tauri build            # Build desktop app (production)

# Utilities
npm run type-check             # TypeScript type checking (add script)
npm run preview                # Preview production build
npm run bump-version           # Increment version number

# Cleaning
rm -rf dist node_modules src-tauri/target  # Full clean
npm install                    # Reinstall dependencies

# Tauri CLI
npx tauri info                 # System information
npx tauri dev                  # Alternative dev command
npx tauri build                # Alternative build command
npx tauri icon path/to/icon.png  # Generate icons

# Rust
cd src-tauri && cargo build    # Build Rust backend only
cd src-tauri && cargo check    # Check Rust code without building
cd src-tauri && cargo clean    # Clean Rust build artifacts
```

---

## Next Steps

- Review [ARCHITECTURE_ANALYSIS.md](./ARCHITECTURE_ANALYSIS.md) for codebase understanding
- See [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md) for UI modifications
- Follow [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) for implementation plan
- Check [QUICK_REFERENCE.md](./QUICK_REFERENCE.md) for fast lookup

---

**Document Version:** 1.0  
**Last Updated:** December 2024
