# Architecture Analysis Document
# Fincept Terminal → Bina Analytics Control Console

**Version:** 3.0.11  
**Analysis Date:** December 2024  
**Purpose:** Complete architectural analysis for adapting Fincept Terminal to Bina Analytics Control Console

---

## Table of Contents
1. [Project Structure Overview](#project-structure-overview)
2. [Technology Stack Breakdown](#technology-stack-breakdown)
3. [Key Components Inventory](#key-components-inventory)
4. [Data Flow Patterns](#data-flow-patterns)
5. [State Management](#state-management)
6. [Build System & Configuration](#build-system--configuration)

---

## Project Structure Overview

### Directory Tree
```
fincept-terminal-desktop/
├── src/                          # Frontend React source code
│   ├── main.tsx                  # React entry point, wraps App with AuthProvider
│   ├── App.tsx                   # Main app component with routing logic
│   ├── App.css                   # Global styles
│   ├── components/               # React components organized by feature
│   │   ├── auth/                 # Authentication screens (Login, Register, etc.)
│   │   ├── dashboard/            # Main dashboard screen container
│   │   ├── tabs/                 # Feature tabs (30+ tabs for different features)
│   │   │   ├── ChatTab.tsx       # AI chat interface
│   │   │   ├── NodeEditorTab.tsx # Workflow visual editor
│   │   │   ├── MarketsTab.tsx    # Market data display
│   │   │   ├── AnalyticsTab.tsx  # Analytics features
│   │   │   ├── data-sources/     # Data source management
│   │   │   ├── node-editor/      # Node editor components
│   │   │   ├── mcp/              # MCP (Model Context Protocol) integration
│   │   │   └── ...               # 20+ other feature tabs
│   │   ├── common/               # Shared components
│   │   ├── payment/              # Payment flow screens
│   │   ├── settings/             # Settings panels
│   │   └── ui/                   # shadcn/ui components (40+ UI primitives)
│   ├── contexts/                 # React Context providers for state
│   │   ├── AuthContext.tsx       # Authentication & session management
│   │   ├── ThemeContext.tsx      # Theme/appearance settings
│   │   ├── NavigationContext.tsx # Navigation state
│   │   ├── DataSourceContext.tsx # Data source registry
│   │   └── ProviderContext.tsx   # Provider configurations
│   ├── services/                 # Business logic & API services
│   │   ├── authApi.tsx           # Backend authentication API
│   │   ├── llmApi.ts             # LLM provider integrations
│   │   ├── workflowService.ts    # Workflow management
│   │   ├── mcpManager.ts         # MCP server management
│   │   ├── nodeExecutionManager.ts # Node execution orchestration
│   │   ├── dataSourceRegistry.ts # Data source registry
│   │   └── ...                   # 30+ specialized services
│   ├── hooks/                    # Custom React hooks
│   ├── types/                    # TypeScript type definitions
│   ├── lib/                      # Utility functions
│   ├── constants/                # App constants (version, etc.)
│   └── stockBrokers/             # Stock broker integrations
│
├── src-tauri/                    # Tauri backend (Rust + Python)
│   ├── src/                      # Rust source code
│   │   ├── lib.rs                # Main Rust entry point
│   │   └── commands/             # Tauri command handlers
│   │       ├── ai_agents.rs      # AI agent execution
│   │       ├── analytics.rs      # Analytics computation
│   │       ├── market_data.rs    # Market data fetching
│   │       └── ...               # 15+ command modules
│   ├── resources/                # Bundled resources
│   │   └── scripts/              # Python scripts
│   │       ├── Analytics/        # CFA-level analytics (Python)
│   │       └── agents/           # AI agent implementations
│   ├── tauri.conf.json           # Tauri configuration
│   ├── Cargo.toml                # Rust dependencies
│   └── capabilities/             # Tauri permissions
│
├── public/                       # Static assets
├── package.json                  # Node.js dependencies & scripts
├── vite.config.ts                # Vite build configuration
├── tsconfig.json                 # TypeScript configuration
├── tailwind.config.js            # Tailwind CSS configuration (implied)
└── components.json               # shadcn/ui configuration

```

### Entry Points

#### 1. **Frontend Entry Point** (`src/main.tsx`)
```typescript
// Initializes React app with AuthProvider wrapper
ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <AuthProvider>
      <App />
    </AuthProvider>
  </React.StrictMode>,
)
```

#### 2. **App Component** (`src/App.tsx`)
- **Purpose:** Main router and screen controller
- **Responsibilities:**
  - Screen-based routing (not React Router, custom screen state)
  - Authentication flow management
  - Payment flow orchestration
  - Provider wrapping for authenticated views
- **Screens:** `login | register | forgotPassword | help | pricing | paymentProcessing | paymentSuccess | dashboard`

**Key Logic:**
- Unauthenticated → Shows auth screens with Header/Footer
- Authenticated + Free Plan → Shows pricing screen
- Authenticated + Subscription/Guest → Shows dashboard
- Dashboard renders with full provider tree

#### 3. **Dashboard Container** (`src/components/dashboard/DashboardScreen.tsx`)
- **Purpose:** Main application shell after authentication
- **Contains:** 
  - Tab navigation system (30+ tabs)
  - Menu bar (File, Data, Intelligence, Tools, Help)
  - Auto-updater integration
  - Full-screen toggle
  - User profile dropdown

#### 4. **Tauri Backend Entry** (`src-tauri/src/lib.rs`)
- Registers Rust commands exposed to frontend via Tauri IPC
- Initializes plugins (HTTP, Shell, SQL, Dialog, Process, Updater)
- Sets up Python interpreter for analytics/agents

---

## Technology Stack Breakdown

### Frontend Technologies

#### **Core Framework**
- **React 19.1.0** (latest)
  - Hooks-based architecture (no class components)
  - React.StrictMode enabled
  - Functional components with TypeScript

#### **TypeScript Configuration**
- **Version:** 5.8.3
- **Target:** ES2020
- **Module:** ESNext
- **JSX:** react-jsx
- **Path Alias:** `@/*` → `./src/*`
- **Strict Mode:** Enabled
- **Excluded:** `src/services/duckdbService.ts` (not in use)

#### **Build System**
- **Vite 7.0.4**
  - Dev server on port 1420
  - Hot module replacement (HMR)
  - Code splitting with manual chunks:
    - `react-vendor`: React core
    - `chart-vendor`: Recharts, Lightweight Charts
    - `ui-vendor`: Radix UI components
    - `flow-vendor`: ReactFlow
    - `tauri-vendor`: Tauri plugins
  - Proxy to backend: `/api` → `https://finceptbackend.share.zrok.io`

#### **UI Framework & Styling**
- **shadcn/ui** (Radix UI primitives)
  - 40+ pre-built accessible components
  - Components in `src/components/ui/`
  - Customizable via `components.json`
- **Tailwind CSS 4.1.13**
  - Utility-first CSS
  - Custom theme via Tailwind config
  - Dark mode support (via `next-themes`)
- **Lucide React** (icons)
  - Modern icon library
  - Tree-shakeable

#### **Routing**
- **Custom Screen-Based Routing** (no React Router for top-level)
  - Uses state-based screen switching in `App.tsx`
  - Screens: auth → pricing → dashboard
  - Dashboard uses Radix UI Tabs for internal navigation

#### **State Management**
- **React Context API** (primary)
  - `AuthContext`: User session, API keys, subscription
  - `ThemeContext`: Dark/light mode, terminal theme
  - `NavigationContext`: Screen navigation
  - `DataSourceContext`: Data source configurations
  - `ProviderContext`: Provider settings (backtesting, etc.)
- **Local Component State** (useState, useReducer)
- **LocalStorage** (persistence)
  - Workflows stored in `localStorage`
  - User preferences cached

#### **Data Fetching**
- **Tauri HTTP Plugin** (`@tauri-apps/plugin-http`)
  - Replaces `fetch` for cross-origin requests
  - Bypasses CORS via native HTTP
- **Axios 1.12.2** (supplemental)
- **WebSocket** (`ws` library)
  - Real-time data streams (market data, etc.)

---

### Backend/Desktop Technologies

#### **Tauri 2.0**
- **Purpose:** Desktop app framework (Electron alternative)
- **Architecture:**
  - **Frontend:** Web technologies (React/Vite) in WebView
  - **Backend:** Rust for system access, IPC, and performance
- **Plugins Used:**
  - `@tauri-apps/plugin-http`: Native HTTP requests
  - `@tauri-apps/plugin-shell`: Execute system commands
  - `@tauri-apps/plugin-sql`: SQLite database
  - `@tauri-apps/plugin-dialog`: File/folder dialogs
  - `@tauri-apps/plugin-process`: Process management
  - `@tauri-apps/plugin-updater`: Auto-update functionality
  - `@tauri-apps/plugin-opener`: Open URLs in browser

#### **Rust Backend** (1.70+)
- **Entry Point:** `src-tauri/src/lib.rs`
- **Command Modules** (in `src-tauri/src/commands/`):
  - `ai_agents.rs`: Execute Python agents
  - `analytics.rs`: Run Python analytics scripts
  - `market_data.rs`: Fetch market data
  - `alphavantage.rs`, `yfinance.rs`, etc.: Data source integrations
  - `bis.rs`, `ecb.rs`, `imf.rs`: Economic data APIs
- **Python Integration:**
  - Bundled Python 3.11+ interpreter
  - Located in `src-tauri/resources/python-windows/`
  - Scripts in `src-tauri/resources/scripts/`

#### **Python Scripts**
- **Analytics** (`resources/scripts/Analytics/`):
  - Portfolio optimization (skfolio)
  - DCF models
  - Risk metrics (VaR, Sharpe, etc.)
  - Options pricing
- **AI Agents** (`resources/scripts/agents/`):
  - LangChain-based agents
  - Investor personas (Buffett, Dalio, etc.)
  - MCP tools integration

---

### Key Libraries

#### **Visualization**
- **Recharts 2.15.4**: Declarative charts (bar, line, area, pie)
- **Lightweight Charts 5.0.9**: TradingView-style candlestick/financial charts
- **ReactFlow 11.11.4**: Visual workflow editor (node-based UI)

#### **AI/LLM Integration**
- **LangChain 0.3.34**: LLM orchestration framework
  - `@langchain/community`: Community integrations
  - `@langchain/ollama`: Ollama (local LLM) support
- **Ollama 0.6.0**: Local LLM client
- **LLM Providers Supported:**
  - OpenAI (GPT-3.5, GPT-4)
  - Anthropic (Claude)
  - Google (Gemini)
  - Groq
  - Ollama (local models)

#### **Drag & Drop**
- **@dnd-kit 6.3.1**: Modern drag-and-drop toolkit
  - Core, Sortable, Utilities

#### **Data Transformation**
- **JSONPath Plus**: JSON querying
- **JSONata 2.1.0**: JSON transformation language
- **JMESPath 0.16.0**: JSON query language
- **Cheerio 1.1.2**: HTML parsing (for web scraping)

#### **Date Handling**
- **date-fns 4.1.0**: Modern date utility library

#### **Markdown**
- **react-markdown 10.1.0**: Render markdown in React
- **remark-gfm 4.0.1**: GitHub Flavored Markdown support

#### **Authentication**
- **OTPAuth 9.4.1**: TOTP/HOTP generation
- **otplib 12.0.1**: One-time password library

---

## Key Components Inventory

### Authentication Flow Components (`src/components/auth/`)

| Component | Purpose | Key Features |
|-----------|---------|-------------|
| `LoginScreen.tsx` | User login | Email/password, Guest mode, MFA support |
| `RegisterScreen.tsx` | Account registration | Email verification, password strength |
| `ForgotPasswordScreen.tsx` | Password reset | Email-based reset flow |
| `HelpScreen.tsx` | Help/support | FAQ, documentation links |
| `PricingScreen.tsx` | Subscription plans | Free/Pro/Enterprise tiers, Stripe integration |

### Payment Flow (`src/components/payment/`)

| Component | Purpose |
|-----------|---------|
| `PaymentOverlay.tsx` | In-app payment window container |
| `InAppPaymentWindow.tsx` | Embedded Stripe checkout |
| `PaymentProcessingScreen.tsx` | Payment status polling |
| `PaymentSuccessScreen.tsx` | Success confirmation & confetti |

### Dashboard & Navigation (`src/components/dashboard/`)

**DashboardScreen.tsx:**
- Tab navigation system (Radix UI Tabs)
- 30+ tabs organized by category
- Menu bar with dropdowns (File, Data, Intelligence, Tools, Help)
- User profile dropdown
- Auto-updater integration
- Full-screen mode toggle

**Tab Categories:**
1. **Core:** Dashboard, Markets, News, Watchlist
2. **Analysis:** Analytics, Backtesting, Equity Research, Screener
3. **Data:** Data Sources, Data Mapping, DBnomics, Economics
4. **Intelligence:** Geopolitics, Maritime, Chat (AI)
5. **Workflows:** Node Editor, Code Editor, MCP
6. **Trading:** Trading, Portfolio, Fyers
7. **Management:** Settings, Profile, Support Tickets, Forum

### Feature Tabs (Selected Deep Dive)

#### **ChatTab.tsx** (AI Interface)
- **Purpose:** Chat with AI agents
- **Features:**
  - Multi-LLM support (OpenAI, Anthropic, Google, Groq, Ollama)
  - Investor persona selection (20+ agents)
  - Streaming responses
  - Context recording/playback
  - Markdown rendering with syntax highlighting
  - Tool call display
- **Integration:** Uses `llmApi.ts` service
- **Size:** ~42KB (1,000+ lines)

#### **NodeEditorTab.tsx** (Workflow Builder)
- **Purpose:** Visual workflow editor
- **Technology:** ReactFlow
- **Node Types:**
  - Data Source Node
  - Python Agent Node
  - MCP Tool Node
  - Technical Indicator Node
  - Agent Mediator Node
  - Results Display Node
  - Backtest Node
  - Optimization Node
- **Features:**
  - Drag-and-drop node creation
  - Connection validation
  - Workflow execution
  - Save/load workflows
  - Export workflow to JSON
- **Integration:**
  - `workflowService.ts`: Persistence
  - `workflowExecutor.ts`: Execution engine
  - `nodeExecutionManager.ts`: Node orchestration
- **Size:** ~60KB (1,500+ lines)

#### **AnalyticsTab.tsx** (Financial Analytics)
- **Purpose:** Run CFA-level analytics
- **Features:**
  - Portfolio optimization (Mean-Variance, Black-Litterman)
  - DCF valuation
  - Options pricing
  - Risk metrics (VaR, Sharpe, Max Drawdown)
  - Technical indicators
- **Backend:** Rust commands → Python scripts
- **Python Libraries:** pandas, numpy, scipy, yfinance, skfolio

#### **EquityResearchTab.tsx** (Stock Analysis)
- **Purpose:** Comprehensive stock research
- **Features:**
  - Financial statements
  - Analyst ratings
  - Price targets
  - Valuation multiples
  - DCF models
  - Chart integration
- **Data Sources:** Alpha Vantage, Polygon.io, Yahoo Finance
- **Size:** ~126KB (largest component)

#### **MaritimeTab.tsx** (3D Globe Tracking)
- **Purpose:** Track ships, aircraft, satellites
- **Features:**
  - 3D globe visualization
  - AIS data integration
  - Trade routes
  - Satellite orbital paths
- **Libraries:** Cesium or Three.js (implied)

#### **Data Sources Tab** (`data-sources/DataSourcesTab.tsx`)
- **Purpose:** Manage data connections
- **Supported Sources:** 100+ connectors
  - **Databases:** PostgreSQL, MySQL, MongoDB, Redis, Snowflake, BigQuery, etc.
  - **Market Data:** Kraken, Binance, Polygon, Alpha Vantage, etc.
  - **Economics:** DBnomics, World Bank, IMF, OECD, BIS, ECB
  - **Streaming:** Kafka, WebSocket, MQTT
  - **Cloud Storage:** S3, GCS, Azure Blob
  - **Custom:** API Mapper (GraphQL, REST)
- **Adapters:** Each source has a TypeScript adapter class in `data-sources/adapters/`
- **Configuration:** `dataSourceConfigs.ts` (400+ lines)

#### **MCP Tab** (`mcp/index.tsx`)
- **Purpose:** Model Context Protocol integration
- **Features:**
  - MCP server management
  - Server marketplace
  - Tool discovery
  - Log viewing
- **Supported Servers:** PostgreSQL, QuestDB, Kite (stock broker), custom servers
- **Use Case:** Extend LLM capabilities with external tools

### Common Components (`src/components/common/`)

| Component | Purpose |
|-----------|---------|
| `Header.tsx` | Auth screen header |
| `Footer.tsx` | Auth screen footer |
| `BackgroundPattern.tsx` | Animated background grid |
| `RecordedContextsManager.tsx` | Manage saved chat contexts |
| `RecordingControlPanel.tsx` | Context recording controls |
| `MarkdownRenderer.tsx` | Render markdown with syntax highlighting |
| `AutoUpdater.tsx` | App update checker |
| `ContextSelector.tsx` | Select context for chat |

### UI Primitives (`src/components/ui/`)
**40+ shadcn/ui components** (Radix UI + Tailwind):
- Layout: `card`, `separator`, `tabs`, `accordion`, `collapsible`
- Forms: `input`, `textarea`, `select`, `checkbox`, `switch`, `radio-group`, `slider`
- Overlays: `dialog`, `sheet`, `drawer`, `popover`, `tooltip`, `hover-card`, `context-menu`, `dropdown-menu`
- Feedback: `alert`, `alert-dialog`, `progress`, `skeleton`, `sonner` (toasts)
- Data Display: `table`, `avatar`, `badge`, `calendar`, `chart`
- Navigation: `menubar`, `breadcrumb`, `sidebar`
- Actions: `button`, `toggle`, `toggle-group`, `command` (command palette)

---

## Data Flow Patterns

### 1. **Authentication Flow**

```
User Input (LoginScreen)
    ↓
AuthApiService.login() [src/services/authApi.tsx]
    ↓
HTTP POST /api/v1/auth/login → Fincept Backend (zrok.io)
    ↓
Response: { api_key, username, email, credit_balance }
    ↓
AuthContext.setSession() [src/contexts/AuthContext.tsx]
    ↓
LocalStorage: Save session data
    ↓
App.tsx: Redirect to dashboard (or pricing if free plan)
```

### 2. **LLM Chat Flow**

```
User Types Message (ChatTab)
    ↓
llmApiService.chat() [src/services/llmApi.ts]
    ↓
Switch on config.provider:
  - OpenAI → POST https://api.openai.com/v1/chat/completions
  - Anthropic → POST https://api.anthropic.com/v1/messages
  - Google → POST https://generativelanguage.googleapis.com/v1beta/models/...
  - Groq → POST https://api.groq.com/openai/v1/chat/completions
  - Ollama → POST http://localhost:11434/api/chat
    ↓
Streaming Response (Server-Sent Events or streamed JSON)
    ↓
onStream callback → Update UI incrementally
    ↓
Store conversation in chat history (localStorage or SQLite)
```

### 3. **Workflow Execution Flow**

```
User Clicks "Run" (NodeEditorTab)
    ↓
workflowExecutor.executeWorkflow() [src/components/tabs/node-editor/WorkflowExecutor.ts]
    ↓
Topological Sort of Nodes (dependency order)
    ↓
For Each Node in Order:
  ↓
  nodeExecutionManager.executeNode() [src/services/nodeExecutionManager.ts]
    ↓
    Switch on Node Type:
      - DataSourceNode → Fetch data from configured source
      - PythonAgentNode → invoke('execute_python_agent') → Rust → Python
      - MCPToolNode → mcpToolService.executeTool() → MCP server
      - TechnicalIndicatorNode → Calculate indicator (SMA, RSI, etc.)
      - AgentMediatorNode → Coordinate multiple agents
    ↓
    Node Result (data object)
    ↓
  Store in execution context
    ↓
End Loop
    ↓
Display Results (ResultsDisplayNode or ResultsModal)
    ↓
Save Workflow to workflowService (localStorage)
```

### 4. **Data Source Connection Flow**

```
User Adds Data Source (DataSourcesTab)
    ↓
Select Source Type (e.g., PostgreSQL)
    ↓
Fill Configuration Form (host, port, database, username, password)
    ↓
Click "Test Connection"
    ↓
Adapter.testConnection() [e.g., PostgreSQLAdapter.ts]
    ↓
(For databases, uses Tauri SQL plugin or HTTP proxy)
(For APIs, uses Tauri HTTP plugin or axios)
    ↓
Success/Error feedback
    ↓
Click "Save"
    ↓
dataSourceRegistry.registerSource() [src/services/dataSourceRegistry.ts]
    ↓
Store in DataSourceContext
    ↓
Persist to localStorage or Tauri SQL
    ↓
Available for use in Node Editor, Analytics, etc.
```

### 5. **Tauri Command Flow** (Frontend → Rust → Python)

```
Frontend: invoke('execute_python_agent', { agent_name, params })
    ↓
Tauri IPC (JSON-RPC over WebSocket/HTTP)
    ↓
Rust Handler: commands/ai_agents.rs
    ↓
Rust: Load Python script from resources/scripts/agents/
    ↓
Rust: Execute Python via embedded interpreter
    ↓
Python: Run LangChain agent logic
    ↓
Python: Return result as JSON
    ↓
Rust: Parse and return to frontend
    ↓
Frontend: Receive result, update UI
```

### 6. **Market Data Real-Time Flow**

```
User Opens Markets Tab or Watchlist
    ↓
Establish WebSocket Connection (via useWebSocket hook)
    ↓
WebSocket → Kraken, Binance, or custom data feed
    ↓
Message Received (price update, order book, trades)
    ↓
Parse Message (JSON)
    ↓
Update Component State (useState or useReducer)
    ↓
Re-render Chart/Table
    ↓
(Optional) Store in SQLite for history
```

---

## State Management

### Global State (React Context)

#### **AuthContext** (`src/contexts/AuthContext.tsx`)
**Purpose:** Authentication & session management

**State:**
```typescript
interface AuthContextState {
  session: {
    authenticated: boolean;
    user_type: 'guest' | 'registered';
    api_key: string;
    user_info: {
      username: string;
      email: string;
      account_type: 'free' | 'pro' | 'enterprise';
      credit_balance: number;
      is_verified: boolean;
      mfa_enabled: boolean;
    };
    subscription: {
      has_subscription: boolean;
      plan: { name: string; plan_id: string; price: number };
      status: string;
      days_remaining: number;
    };
  } | null;
  isLoading: boolean;
}
```

**Methods:**
- `login(email, password)`: Authenticate user
- `register(email, username, password)`: Create account
- `logout()`: Clear session
- `guestLogin()`: Anonymous access
- `updateProfile()`: Modify user data

**Usage:** Consumed by all components needing auth status

#### **ThemeContext** (`src/contexts/ThemeContext.tsx`)
**Purpose:** Theme & appearance settings

**State:**
```typescript
interface ThemeContextState {
  theme: 'dark' | 'light';
  terminalTheme: {
    background: string;
    foreground: string;
    accent: string;
    // Terminal color palette
  };
}
```

**Methods:**
- `toggleTheme()`: Switch dark/light
- `setTerminalTheme(name)`: Apply preset theme

#### **DataSourceContext** (`src/contexts/DataSourceContext.tsx`)
**Purpose:** Data source registry

**State:**
```typescript
interface DataSourceContextState {
  sources: DataSource[]; // Configured data sources
  activeSource: string | null; // Currently selected source
}
```

**Methods:**
- `addSource(config)`: Register new source
- `removeSource(id)`: Delete source
- `testSource(id)`: Test connection
- `setActiveSource(id)`: Select for use

#### **NavigationContext** (`src/contexts/NavigationContext.tsx`)
**Purpose:** Screen navigation

**State:**
```typescript
interface NavigationContextState {
  currentScreen: Screen;
  activeTab: string;
}
```

**Methods:**
- `navigateToScreen(screen)`: Change screen
- `setActiveTab(tab)`: Switch tab in dashboard

#### **ProviderContext** (`src/contexts/ProviderContext.tsx`)
**Purpose:** Provider configurations (backtesting, trading)

**State:**
```typescript
interface ProviderContextState {
  backtestingProvider: string; // 'python' | 'rust' | etc.
  tradingProvider: string; // 'fyers' | 'paper' | etc.
}
```

### Local Component State

**Patterns Used:**
- `useState`: Simple state (forms, toggles, UI state)
- `useReducer`: Complex state (chat messages, workflow nodes)
- `useRef`: DOM references, mutable values (WebSocket clients)
- `useCallback`, `useMemo`: Performance optimization

### Persistence Layer

#### **LocalStorage**
- **Workflows:** `fincept_workflows` key
- **User Preferences:** Theme, terminal settings
- **Chat History:** Conversations (limited)
- **Data Source Configs:** Source credentials (encrypted?)

#### **Tauri SQLite** (`@tauri-apps/plugin-sql`)
- **Chat History:** Full conversation storage
- **Workflow Results:** Execution outputs
- **Market Data Cache:** Historical data
- **Analytics Results:** Computed metrics

**Example:**
```typescript
import Database from '@tauri-apps/plugin-sql';

const db = await Database.load('sqlite:fincept.db');
await db.execute('CREATE TABLE IF NOT EXISTS chats (...)');
await db.execute('INSERT INTO chats VALUES (?, ?)', [id, message]);
```

---

## Build System & Configuration

### Vite Configuration (`vite.config.ts`)

**Key Settings:**
- **Plugins:** `@vitejs/plugin-react`, `@tailwindcss/vite`
- **Dev Server:** Port 1420, strict port mode
- **Proxy:** `/api` → `https://finceptbackend.share.zrok.io` (backend API)
- **Alias:** `@/*` → `./src/*`
- **Code Splitting:**
  - `react-vendor`: React, React DOM, React Router
  - `chart-vendor`: Recharts, Lightweight Charts
  - `ui-vendor`: Radix UI components
  - `flow-vendor`: ReactFlow
  - `tauri-vendor`: Tauri plugins
- **Build Output:** `dist/` (frontendDist in Tauri)

### Tauri Configuration (`src-tauri/tauri.conf.json`)

**App Identity:**
- **Product Name:** `FinceptTerminal`
- **Identifier:** `com.fincept.terminal`
- **Version:** `3.0.11`

**Window Settings:**
- **Title:** `FinceptTerminal`
- **Size:** 1400x900 (min: 1024x768)
- **Resizable:** Yes
- **Center:** Yes

**Build Commands:**
- **Dev:** `npm run dev` (Vite dev server)
- **Build:** `npm run build` (TypeScript + Vite build)
- **Frontend Dist:** `../dist`

**Bundle:**
- **Targets:** `nsis` (Windows), `appimage` (Linux)
- **Icon:** `icons/icon.ico` (Windows), `icons/icon.icns` (macOS)
- **Resources:** `resources/scripts` (bundled Python scripts)
- **External Binary:** `binaries/python-interpreter` (Python 3.11+)
- **Auto-Updater:** Enabled, GitHub releases endpoint

**Plugins:**
- **updater:** Auto-update from GitHub releases
- **http:** Native HTTP requests (CORS bypass)
- **shell:** Execute system commands
- **sql:** SQLite database
- **dialog:** File/folder pickers
- **process:** Process management
- **opener:** Open URLs in browser

### TypeScript Configuration (`tsconfig.json`)

**Compiler Options:**
- **Target:** ES2020
- **Module:** ESNext
- **JSX:** react-jsx
- **Strict:** Yes (strict type checking)
- **Module Resolution:** bundler (Vite)
- **Path Aliases:** `@/*` → `./src/*`

**Exclusions:**
- `src/services/duckdbService.ts` (not in use)

### NPM Scripts (`package.json`)

| Script | Command | Purpose |
|--------|---------|---------|
| `dev` | `vite` | Start dev server (port 1420) |
| `build` | `tsc && vite build` | TypeScript check + production build |
| `preview` | `vite preview` | Preview production build |
| `tauri` | `tauri` | Tauri CLI commands |
| `tauri dev` | `tauri dev` | Run Tauri dev (Rust + Vite) |
| `tauri build` | `tauri build` | Build desktop app bundle |
| `bump-version` | `node scripts/bump-version.js` | Increment version |

---

## Key Architectural Insights

### Strengths
1. **Modular Design:** Clear separation of concerns (components, services, contexts)
2. **Extensible Data Sources:** 100+ adapters, easy to add more
3. **Flexible LLM Integration:** Supports multiple providers with unified interface
4. **Visual Workflow Builder:** ReactFlow-based, intuitive node editor
5. **Tauri Benefits:** Native performance, small bundle size, Rust security

### Areas of Complexity
1. **Large Monolith:** 30+ tabs in single app, ~150KB+ of component code
2. **Custom Routing:** No React Router, screen-based state switching
3. **Mixed State Management:** Contexts + localStorage + SQLite (no single source of truth)
4. **Python Coupling:** Heavy reliance on Python scripts for analytics
5. **API Proxy:** Backend via zrok tunnel (not production-ready for Bina)

### Adaptation Opportunities for Bina
1. **Remove Trading Features:** Clean up Fyers, paper trading, order management
2. **Simplify Auth Flow:** Remove pricing/payment screens
3. **Replace Data Sources:** Focus on Supabase, n8n, custom APIs
4. **Rebrand UI:** Change logo, colors, terminology (Terminal → Console)
5. **Task-Centric Dashboard:** Replace financial tabs with research orchestration
6. **Client Reporting:** Build out ReportBuilderTab for client deliverables

---

## Next Steps
- Review [INTEGRATION_POINTS.md](./INTEGRATION_POINTS.md) for backend integration strategy
- See [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md) for step-by-step modifications
- Follow [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) for phased implementation

---

**Document Version:** 1.0  
**Last Updated:** December 2024  
**Maintainer:** Bina Analytics Team
