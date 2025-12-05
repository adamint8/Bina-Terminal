# Fincept Terminal - Comprehensive Codebase Overview

**Version:** 3.0.11  
**Last Updated:** 2025-01-05  
**Purpose:** Main gateway document for understanding this codebase before making significant changes

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Directory Structure](#directory-structure)
3. [Core Concepts & Data Flow](#core-concepts--data-flow)
4. [Auth & Session Model](#auth--session-model)
5. [External Integrations](#external-integrations)
6. [Important Files to Know](#important-files-to-know)
7. [Risks, Tech Debt & Recommendations](#risks-tech-debt--recommendations)

---

## Architecture Overview

### What is Fincept Terminal?

Fincept Terminal is a **desktop financial analysis platform** built with:
- **Frontend**: React 19 + TypeScript + TailwindCSS v4
- **Desktop Framework**: Tauri 2.0 (Rust-based)
- **Backend Processing**: Python 3.11+ (AI agents, analytics, CFA-level calculations)
- **Database**: SQLite (local storage via Tauri plugin)
- **External APIs**: 100+ data adapters, WebSocket streaming, REST APIs

Think of it as a **Bloomberg Terminal alternative** with AI-powered automation, unlimited data connectivity, and open-source flexibility.

---

### High-Level Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  React 19 Components + TypeScript + TailwindCSS             │
│  • 40+ Tabs (Dashboard, Markets, Chat, Analytics, etc.)     │
│  • Real-time charts (Recharts, Lightweight Charts)          │
│  • Visual workflow editor (ReactFlow)                       │
└─────────────────────────────────────────────────────────────┘
                            ↕ (Tauri IPC)
┌─────────────────────────────────────────────────────────────┐
│                   APPLICATION LAYER                          │
│  Tauri 2.0 (Rust) - Desktop App Shell                       │
│  • Tauri Commands (~63 .rs files)                           │
│  • Window management, file system, security                 │
│  • Process spawning for Python/MCP servers                  │
│  • SQLite database access                                   │
└─────────────────────────────────────────────────────────────┘
                            ↕ (subprocess calls)
┌─────────────────────────────────────────────────────────────┐
│                 BUSINESS LOGIC LAYER                         │
│  Python Scripts (Analytics, AI Agents)                      │
│  • CFA-level portfolio analytics                            │
│  • 20+ AI investor personas (Buffett, Dalio, etc.)          │
│  • Hedge fund strategy simulations                          │
│  • ML/quant models (scikit-learn, pandas, numpy)            │
└─────────────────────────────────────────────────────────────┘
                            ↕ (HTTP/WebSocket)
┌─────────────────────────────────────────────────────────────┐
│                    INTEGRATION LAYER                         │
│  • 93+ Data Source Adapters (REST APIs, databases, cloud)   │
│  • WebSocket Manager (real-time market data)                │
│  • MCP (Model Context Protocol) servers                     │
│  • External APIs: Polygon, Kraken, DBnomics, etc.           │
└─────────────────────────────────────────────────────────────┘
```

---

### What Runs Where?

| Component | Language | Runtime | Purpose |
|-----------|----------|---------|---------|
| **UI Components** | TypeScript/React | Browser (via Tauri WebView) | Rendering, user interaction |
| **State Management** | TypeScript | Browser | React Context API, no Redux |
| **Desktop Shell** | Rust | Native (via Tauri) | Window, filesystem, security, SQLite |
| **Data Adapters** | TypeScript | Browser | API clients, connection management |
| **Analytics Engine** | Python | Subprocess | CFA calculations, ML models |
| **AI Agents** | Python | Subprocess | LLM orchestration, investment personas |
| **MCP Servers** | Node.js/Python | Subprocess | Tool execution, external integrations |

**Key Insight**: This is a **hybrid architecture**. Heavy computation (Python) runs in subprocesses spawned by Rust, while UI logic stays in React. Tauri bridges them via IPC commands.

---

## Directory Structure

### Root Level
```
fincept-terminal-desktop/
├── src/                    # Frontend React application (~397 TS/TSX files)
├── src-tauri/              # Rust backend (~63 .rs files)
├── public/                 # Static assets (images, icons)
├── docs/                   # Documentation
├── scripts/                # Build/deployment scripts
├── package.json            # Node dependencies
├── tsconfig.json           # TypeScript config
├── vite.config.ts          # Vite bundler config
└── components.json         # shadcn/ui component config
```

---

### Frontend Structure (`src/`)

```
src/
├── main.tsx                       # Entry point - renders <App /> inside <AuthProvider>
├── App.tsx                        # Root component - routing, screen management
├── App.css                        # Global styles
│
├── components/                    # All UI components (~300+ files)
│   ├── auth/                     # Login, Register, ForgotPassword screens
│   ├── dashboard/                # Main dashboard container
│   │   └── DashboardScreen.tsx   # Tab system, menu bar, auto-updater
│   ├── tabs/                     # 40+ feature tabs (Dashboard, Markets, Chat, etc.)
│   │   ├── DashboardTab.tsx      # Customizable widget dashboard
│   │   ├── MarketsTab.tsx        # Market data grid
│   │   ├── ChatTab.tsx           # AI chat interface (Ollama/OpenAI)
│   │   ├── PortfolioTab.tsx      # Portfolio management
│   │   ├── AnalyticsTab.tsx      # Python analytics executor
│   │   ├── NodeEditorTab.tsx     # Visual workflow builder (ReactFlow)
│   │   ├── data-sources/         # Data source connection UI + 93 adapters
│   │   ├── data-mapping/         # API-to-data mapper (JSONPath, JMESPath)
│   │   ├── mcp/                  # MCP server management
│   │   ├── equity-research/      # Stock research & DCF models
│   │   ├── fyers/                # Fyers broker integration
│   │   └── ...                   # 30+ more tabs
│   ├── common/                   # Shared components (Header, Footer, BackgroundPattern)
│   ├── payment/                  # Payment screens (Stripe integration)
│   ├── info/                     # Static pages (Contact Us, Terms, Privacy)
│   ├── ui/                       # shadcn/ui components (Button, Dialog, etc.)
│   └── settings/                 # Settings screens
│
├── contexts/                      # React Context providers (global state)
│   ├── AuthContext.tsx           # Session management, login/logout, subscription
│   ├── NavigationContext.tsx     # Screen navigation helper
│   ├── ThemeContext.tsx          # Light/dark theme
│   ├── DataSourceContext.tsx     # Data source connections state
│   └── ProviderContext.tsx       # WebSocket provider configs
│
├── services/                      # Business logic & API clients (~40 files)
│   ├── authApi.tsx               # Backend auth API client
│   ├── paymentApi.tsx            # Stripe payment API
│   ├── sqliteService.ts          # SQLite database wrapper
│   ├── workflowService.ts        # Workflow execution engine
│   ├── mcpManager.ts             # MCP server lifecycle management
│   ├── mcpClient.ts              # MCP protocol implementation
│   ├── dataSourceRegistry.ts    # Unified data source registry
│   ├── websocket/                # WebSocket manager + adapters
│   ├── backtesting/              # Backtesting engine (QuantConnect Lean)
│   ├── trading/                  # Trading API integrations
│   ├── pythonAgentService.ts     # Python agent executor
│   ├── ollamaService.ts          # Local LLM (Ollama) integration
│   ├── llmApi.ts                 # Cloud LLM API (OpenAI, Anthropic)
│   └── ...                       # 30+ more services
│
├── hooks/                         # Custom React hooks
│   ├── useAutoUpdater.ts         # Auto-update checker (Tauri updater)
│   ├── useDataSource.ts          # Data source hook (WebSocket/REST)
│   ├── useWebSocket.ts           # WebSocket connection hook
│   └── usePaperTrading.ts        # Paper trading simulator hook
│
├── lib/                           # Utility functions
│   └── utils.ts                  # Helpers (cn, formatters, etc.)
│
├── types/                         # TypeScript type definitions
│   ├── fyers-web-sdk-v3.d.ts
│   ├── hyperliquid.d.ts
│   └── trading.d.ts
│
└── constants/
    └── version.ts                # App version constant
```

**Key Insight**: Components are organized by **feature** (tabs/) rather than type. Each tab is self-contained with its own logic.

---

### Backend Structure (`src-tauri/`)

```
src-tauri/
├── src/
│   ├── main.rs                   # Tauri app entry point
│   ├── lib.rs                    # Main library - registers all commands
│   │
│   ├── commands/                 # Tauri commands (callable from frontend)
│   │   ├── ai_agents.rs          # Python AI agent executor
│   │   ├── analytics.rs          # Python analytics executor
│   │   ├── jupyter.rs            # Jupyter notebook integration
│   │   ├── market_data.rs        # Market data APIs (yfinance, etc.)
│   │   ├── alphavantage.rs       # Alpha Vantage API
│   │   ├── polygon.rs            # Polygon.io API
│   │   ├── bis.rs                # Bank for International Settlements
│   │   ├── ecb.rs                # European Central Bank
│   │   ├── imf.rs                # International Monetary Fund
│   │   ├── fiscaldata.rs         # US Treasury Fiscal Data
│   │   ├── congress_gov.rs       # US Congress API
│   │   └── ...                   # 20+ more API commands
│   │
│   ├── data_sources/             # Data source utilities
│   │   └── mod.rs
│   │
│   └── utils/                    # Backend utilities
│       ├── mod.rs
│       ├── python.rs             # Python interpreter management
│       └── ...
│
├── resources/                    # Bundled resources
│   ├── python/                   # Bundled Python 3.11 runtime
│   └── scripts/                  # Python scripts
│       ├── Analytics/            # CFA-level analytics (DCF, VaR, Sharpe, etc.)
│       │   ├── portfolioManagement/
│       │   ├── equityInvestment/
│       │   ├── derivatives/
│       │   ├── technical_analysis/
│       │   ├── quant/
│       │   ├── backtesting/
│       │   ├── ml4Trading/       # Machine learning models
│       │   └── ...
│       └── agents/               # AI agent personas
│           ├── TraderInvestorsAgent/  # Buffett, Dalio, Graham, Lynch, etc.
│           ├── hedgeFundAgents/       # Bridgewater, Citadel, Renaissance
│           ├── GeopoliticsAgents/     # Grand Chessboard, Prisoners of Geography
│           └── EconomicAgents/        # Central bank agents
│
├── Cargo.toml                    # Rust dependencies
├── tauri.conf.json               # Tauri app configuration
└── build.rs                      # Build script
```

**Key Insight**: Rust commands are thin wrappers that spawn Python processes. Heavy lifting happens in Python scripts.

---

## Core Concepts & Data Flow

### Domain Models

The application revolves around these core entities:

1. **User/Session**
   - Authenticated users (registered accounts)
   - Guest users (device-based temporary access)
   - Subscription status (free/premium plans via Stripe)

2. **Data Sources**
   - 93+ adapters (PostgreSQL, MongoDB, S3, Kafka, APIs, etc.)
   - Unified connection registry (stored in SQLite)
   - Two types: WebSocket (real-time) and REST API (on-demand)

3. **Workflows**
   - Visual node-based automation (ReactFlow editor)
   - Node types: DataSource, PythonAgent, MCPTool, Backtest, TechnicalIndicator, etc.
   - Stored as JSON graphs in localStorage (nodes + edges)

4. **MCP Servers**
   - Model Context Protocol servers (external tools)
   - Lifecycle managed by MCPManager
   - Each server provides "tools" callable from workflows/chat

5. **Chat Sessions**
   - AI chat conversations (stored in SQLite)
   - Supports multiple LLM providers (Ollama, OpenAI, Anthropic)
   - Context recording for debugging

6. **Portfolios**
   - Multi-asset portfolios (stocks, crypto, derivatives)
   - Portfolio optimization (max Sharpe, min variance)
   - Performance analytics (VaR, drawdown, alpha, beta)

7. **Watchlists**
   - Custom ticker lists
   - Real-time price updates (WebSocket)

8. **Ticker Storage**
   - Persistent ticker selections across tabs

---

### Data Flow Patterns

#### Pattern 1: Simple UI → REST API → Display
```
User Action (e.g., search ticker)
    ↓
React Component (e.g., MarketsTab)
    ↓
Service Call (e.g., yfinanceService.searchTickers())
    ↓
Tauri Command via IPC (invoke('search_ticker'))
    ↓
Rust Command Handler (commands/yfinance.rs)
    ↓
Python Subprocess (yfinance library)
    ↓
External API (Yahoo Finance)
    ↓
Response flows back up the stack
    ↓
React state update → UI re-render
```

#### Pattern 2: Real-time WebSocket Data
```
WebSocket Subscription (e.g., Kraken BTC/USD)
    ↓
WebSocketManager.subscribe('kraken.trades.BTC/USD')
    ↓
Opens WebSocket connection (managed in service)
    ↓
Incoming messages → callbacks fire
    ↓
React state updates via useState/useEffect
    ↓
Live chart updates (Lightweight Charts)
```

#### Pattern 3: Visual Workflow Execution
```
User creates workflow in NodeEditorTab
    ↓
Saves workflow to localStorage (workflowService)
    ↓
Executes workflow
    ↓
WorkflowExecutor processes nodes in topological order
    ↓
For each node:
  - DataSourceNode → fetches data via adapter
  - PythonAgentNode → spawns Python agent subprocess
  - MCPToolNode → calls MCP server tool
  - BacktestNode → runs backtest strategy
    ↓
Results displayed in ResultsModal
```

#### Pattern 4: Python Analytics Flow
```
User clicks "Run Analytics" (e.g., DCF valuation)
    ↓
AnalyticsTab.tsx
    ↓
invoke('execute_python_script', { script: 'dcf_model.py', params: {...} })
    ↓
Rust command (commands/analytics.rs)
    ↓
Spawns Python process with bundled interpreter
    ↓
Python script executes (uses pandas, numpy, scipy)
    ↓
Outputs JSON result to stdout
    ↓
Rust captures output, returns to frontend
    ↓
React displays result in chart/table
```

---

### State Management

**No Redux/Zustand** - Uses React's built-in Context API exclusively.

#### Contexts and Their Responsibilities

| Context | Purpose | State Stored |
|---------|---------|--------------|
| **AuthContext** | User authentication & subscription | session, user_info, subscription, api_key |
| **ThemeContext** | Dark/light mode | theme ('dark' \| 'light') |
| **NavigationContext** | Screen routing | Helper functions for navigation |
| **DataSourceContext** | Data source connections | connections[] array |
| **ProviderContext** | WebSocket provider configs | providerConfigs map |

**State Flow**:
- Global state → Context API
- Component-local state → useState
- Async state → useEffect + useState
- Derived state → useMemo
- Persistent state → SQLite or localStorage

**SQLite Tables** (key ones):
- `credentials` - API keys, secrets
- `settings` - App settings
- `llm_configs` - LLM provider configs
- `chat_sessions` - Chat history
- `data_sources` - Unified data source registry
- `ws_provider_configs` - WebSocket provider credentials
- `mcp_servers` - MCP server installations
- `mcp_tools` - Available MCP tools
- `workflows` - (localStorage, not SQLite)

---

## Auth & Session Model

### How Authentication Works Today

The auth system supports **two modes**:

1. **Guest Mode** (Device Registration)
   - User clicks "Continue as Guest"
   - Frontend generates unique `device_id` (UUID)
   - Calls backend API: `POST /auth/device/register`
   - Backend returns `temp_api_key` (valid 7 days)
   - Daily request limit: typically 100 requests
   - No email verification needed

2. **Registered User Mode**
   - User creates account: `POST /auth/register`
   - Receives OTP via email
   - Verifies OTP: `POST /auth/verify-otp`
   - Logs in: `POST /auth/login`
   - Backend returns `api_key` (permanent, no expiration)
   - Can upgrade to paid subscription (Stripe integration)

---

### Session Storage

**Where Sessions Live**:
- **Memory**: AuthContext state (React Context)
- **SQLite**: `credentials` table stores `api_key`
- **localStorage**: Used for `device_id` generation

**Session Structure** (AuthContext):
```typescript
interface SessionData {
  authenticated: boolean;           // true if logged in
  user_type: 'guest' | 'registered' | null;
  api_key: string | null;           // Auth token for backend
  device_id: string;                // Unique device identifier
  user_info?: {
    username?: string;
    email?: string;
    account_type?: string;          // 'free', 'premium', etc.
    credit_balance?: number;
    is_verified?: boolean;
    mfa_enabled?: boolean;
  };
  expires_at?: string;              // For guest sessions only
  daily_limit?: number;             // For guest sessions only
  requests_today?: number;          // For guest sessions only
  subscription?: UserSubscription;  // Stripe subscription details
}
```

---

### Protected Routes

**Frontend Route Protection**:
- `App.tsx` checks `session.authenticated`
- If `false` → shows LoginScreen/RegisterScreen
- If `true` → shows DashboardScreen

**No JWT validation** - Backend API expects `api_key` in headers:
```typescript
headers: {
  'Content-Type': 'application/json',
  'x-api-key': session.api_key
}
```

**Backend API** (external service):
- Base URL (prod): `https://finceptbackend.share.zrok.io`
- Base URL (dev): `/api` (proxied via Vite)
- Validates `x-api-key` header on every request
- Returns 401 if invalid/expired

---

### Session Validation Flow

1. **On App Startup**:
   ```
   main.tsx renders <AuthProvider>
     ↓
   AuthContext useEffect runs
     ↓
   loadStoredSession() from SQLite
     ↓
   If api_key found → calls GET /auth/status
     ↓
   Backend validates api_key
     ↓
   If valid → sets session.authenticated = true
   If invalid → shows login screen
   ```

2. **On API Call**:
   ```
   Service function (e.g., authApi.login())
     ↓
   Includes x-api-key header
     ↓
   Backend checks api_key in database
     ↓
   Returns 401 if invalid
     ↓
   Frontend catches error → logout() → redirect to login
   ```

---

### Logout Flow

```typescript
// AuthContext.logout()
1. Clear session state (set to null)
2. Delete api_key from SQLite credentials table
3. Clear localStorage device_id (optional)
4. Navigate to login screen
```

---

### Limitations & Tech Debt

1. **No Token Refresh** - api_key never expires (security risk)
2. **No Session Timeout** - User stays logged in forever unless manual logout
3. **API Key in Headers** - Not using standard Bearer tokens
4. **Device ID Storage** - localStorage is web-only, doesn't persist across OS reinstalls
5. **Guest Session Expiry** - Not enforced on frontend (backend returns 401 after 7 days)
6. **MFA Not Implemented** - `mfa_enabled` field exists but no 2FA flow
7. **Password Reset** - OTP-based but no rate limiting on frontend
8. **Concurrent Sessions** - No multi-device session management
9. **API Key Rotation** - No mechanism to rotate keys
10. **Session Hijacking Risk** - API key stored in plaintext in SQLite

**Recommendation**: Implement JWT with refresh tokens, enforce session expiry, add token rotation.

---

## External Integrations

### Backend API (Critical Dependency)

**Service**: Custom Python FastAPI backend  
**URLs**:
- Production: `https://finceptbackend.share.zrok.io`
- Development: Proxied via Vite to `/api`

**Endpoints Used**:
| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/register` | POST | Create user account |
| `/auth/verify-otp` | POST | Verify email OTP |
| `/auth/login` | POST | Login user |
| `/auth/status` | GET | Check session validity |
| `/auth/device/register` | POST | Guest registration |
| `/auth/forgot-password` | POST | Request password reset OTP |
| `/auth/reset-password` | POST | Reset password with OTP |
| `/payment/plans` | GET | List subscription plans |
| `/payment/create-session` | POST | Create Stripe checkout |
| `/payment/subscription` | GET | Get user subscription |
| `/forum/*` | Various | Forum/community features |
| `/marketplace/*` | Various | Marketplace features |
| `/support/*` | Various | Support tickets |

**Hard-coded in**:
- `src/services/authApi.tsx` (line 8)
- `vite.config.ts` (line 37 proxy config)

**Failure Mode**: If backend is down, auth fails → app unusable. No offline mode.

---

### Stripe Payment Integration

**Purpose**: Subscription payment processing  
**Implementation**: `src/services/paymentApi.tsx`

**Flow**:
1. User clicks plan in PricingScreen
2. Frontend calls `createPaymentSession(planId)`
3. Backend returns Stripe `checkout_url`
4. Opens PaymentOverlay with embedded Stripe iframe
5. User completes payment on Stripe
6. Stripe webhook notifies backend
7. Frontend polls `getUserSubscription()` to confirm
8. Navigates to PaymentSuccessScreen

**Limitations**:
- No webhook listener in desktop app (relies on polling)
- Payment success detection is delayed (1-2 seconds)
- No payment failure handling (user stuck on payment screen)

---

### Data Source Adapters (93 Total)

**Location**: `src/components/tabs/data-sources/adapters/`

**Categories**:
- **Databases** (17): PostgreSQL, MySQL, MongoDB, Redis, Cassandra, CockroachDB, etc.
- **Cloud Storage** (8): S3, GCS, Azure Blob, Cloudflare R2, Backblaze B2, etc.
- **Streaming** (5): Kafka, RabbitMQ, MQTT, WebSocket, Pulsar
- **Market Data** (12): Polygon, Alpha Vantage, Yahoo Finance, Kraken, Coinbase, etc.
- **Analytics** (6): Google Analytics, Mixpanel, Amplitude, Segment, Plausible, etc.
- **Search** (5): Elasticsearch, Algolia, Meilisearch, Typesense, OpenSearch
- **Graph** (3): Neo4j, ArangoDB, OrientDB
- **Time-Series** (4): InfluxDB, Prometheus, TimescaleDB, QuestDB
- **Misc APIs** (33): Bloomberg, Refinitiv, IEX Cloud, Quandl, FRED, World Bank, etc.

**Base Class**: `BaseAdapter` (adapters/BaseAdapter.ts)

**Interface**:
```typescript
abstract class BaseAdapter {
  abstract testConnection(): Promise<TestConnectionResult>;
  async connect(): Promise<void>;
  async disconnect(): Promise<void>;
  async query(query: string, request?: any): Promise<any>;
  async getMetadata(): Promise<Record<string, any>>;
}
```

**Example Adapter**: `PostgreSQLAdapter.ts`
```typescript
testConnection() {
  // Uses node-postgres library
  // Returns { success, message }
}
query(sql) {
  // Executes SQL query
  // Returns rows
}
```

**Registry**: `src/services/dataSourceRegistry.ts`
- Creates WebSocket or REST API data sources
- Stores in SQLite `data_sources` table
- Auto-initializes connections

---

### WebSocket Providers

**Location**: `src/services/websocket/`

**Supported Providers**:
1. **Kraken** - Crypto market data (adapters/kraken.ts)
2. **Polygon.io** - US stocks & forex (adapters/polygon.ts)
3. **Coinbase** - Crypto market data (adapters/coinbase.ts)
4. **Binance** - Crypto market data (adapters/binance.ts)
5. **Generic WebSocket** - Custom URLs (adapters/generic.ts)

**Manager**: `WebSocketManager` (services/websocket/index.ts)
- Singleton pattern
- Manages multiple connections
- Topic-based subscriptions: `provider.channel.symbol`
- Auto-reconnect on disconnect
- Message queueing

**Usage Example**:
```typescript
const wsManager = getWebSocketManager();
wsManager.subscribe('kraken.trades.BTC/USD', (data) => {
  console.log('New trade:', data);
});
```

---

### MCP (Model Context Protocol) Integration

**Purpose**: Execute external tools via MCP servers (similar to LangChain tools)

**Location**: `src/services/mcpManager.ts`, `src/services/mcpClient.ts`

**How It Works**:
1. MCP servers are Node.js/Python processes
2. MCPManager spawns servers via Tauri command (`spawn_mcp_server`)
3. Communication via stdio (JSON-RPC 2.0)
4. Each server exposes "tools" (functions callable from frontend)
5. Tools used in ChatTab (AI agents) and NodeEditorTab (workflows)

**Example MCP Servers**:
- `@modelcontextprotocol/server-filesystem` - File operations
- `@modelcontextprotocol/server-git` - Git operations
- `@modelcontextprotocol/server-postgres` - PostgreSQL queries
- `@modelcontextprotocol/server-slack` - Slack integration
- Custom servers (can install any MCP-compatible server)

**Lifecycle**:
- Install: `mcpManager.installServer(config)`
- Start: `mcpManager.startServer(serverId)`
- Call Tool: `mcpManager.callTool(serverId, toolName, args)`
- Stop: `mcpManager.stopServer(serverId)`

**Storage**: SQLite tables `mcp_servers`, `mcp_tools`

---

### AI/LLM Providers

**Supported Providers**:
1. **Ollama** (Local) - `src/services/ollamaService.ts`
   - Runs locally (default: localhost:11434)
   - Models: llama3, mistral, phi3, etc.
   - No API key needed

2. **OpenAI** - `src/services/llmApi.ts`
   - Models: gpt-4, gpt-3.5-turbo
   - Requires API key

3. **Anthropic** - `src/services/llmApi.ts`
   - Models: claude-3-opus, claude-3-sonnet
   - Requires API key

4. **Together.ai** - `src/services/llmApi.ts`
   - Various open-source models
   - Requires API key

**Storage**: SQLite table `llm_configs`

**Global Settings**: Temperature, max_tokens, system_prompt (stored in `llm_global_settings`)

---

### Python Analytics & Agents

**Python Scripts Location**: `src-tauri/resources/scripts/`

**Categories**:

1. **Analytics** (CFA-Level Calculations)
   - `portfolioManagement/` - Sharpe ratio, VaR, max drawdown, optimization
   - `equityInvestment/` - DCF (FCFF, FCFE), dividend discount, multiples
   - `derivatives/` - Black-Scholes, Greeks, binomial tree
   - `technical_analysis/` - RSI, MACD, Bollinger Bands, candlestick patterns
   - `quant/` - Factor models, backtesting, pairs trading
   - `backtesting/` - Vectorized backtesting engine
   - `ml4Trading/` - ML models (Random Forest, XGBoost for price prediction)

2. **AI Agents** (Investment Personas)
   - `TraderInvestorsAgent/` - Warren Buffett, Benjamin Graham, Peter Lynch, etc.
   - `hedgeFundAgents/` - Bridgewater (Ray Dalio), Citadel, Renaissance Technologies
   - `GeopoliticsAgents/` - Grand Chessboard, Prisoners of Geography frameworks
   - `EconomicAgents/` - Central bank policy agents

**Execution**:
- Frontend calls Tauri command: `invoke('execute_python_script')`
- Rust spawns Python subprocess with bundled interpreter
- Passes parameters as JSON to stdin
- Captures stdout/stderr
- Returns JSON result to frontend

**Example** (DCF Valuation):
```python
# scripts/Analytics/equityInvestment/dcf_fcff.py
import sys
import json
import numpy as np

def calculate_dcf(params):
    fcf = params['free_cash_flow']
    growth_rate = params['growth_rate']
    discount_rate = params['wacc']
    terminal_growth = params['terminal_growth_rate']
    
    # Project cash flows
    projected_fcf = [fcf * (1 + growth_rate) ** i for i in range(1, 6)]
    
    # Terminal value
    terminal_value = projected_fcf[-1] * (1 + terminal_growth) / (discount_rate - terminal_growth)
    
    # Present values
    pv_fcf = [cf / (1 + discount_rate) ** i for i, cf in enumerate(projected_fcf, 1)]
    pv_terminal = terminal_value / (1 + discount_rate) ** 5
    
    enterprise_value = sum(pv_fcf) + pv_terminal
    
    return {
        'enterprise_value': enterprise_value,
        'projected_fcf': projected_fcf,
        'terminal_value': terminal_value
    }

if __name__ == '__main__':
    params = json.loads(sys.stdin.read())
    result = calculate_dcf(params)
    print(json.dumps(result))
```

---

### Third-Party APIs

**Direct API Integrations** (non-adapter):

| API | Purpose | Authentication | Location |
|-----|---------|----------------|----------|
| **Polygon.io** | US stock/forex data | API key | `src/services/polygonService.ts` |
| **Alpha Vantage** | Stock/forex/crypto data | API key | `src/services/alphaVantageService.ts` |
| **Yahoo Finance** | Stock data (via yfinance) | None | `src/services/yfinanceService.ts` → Rust → Python |
| **Kraken** | Crypto market data | API key/secret | `src/services/websocket/adapters/kraken.ts` |
| **DBnomics** | Economic indicators | None | `src-tauri/src/commands/dbnomics.rs` |
| **Fyers** | Indian stock broker | OAuth 2.0 | `src/services/fyersService.ts` |

**Hard-Coded API URLs**:
- Polygon: `https://api.polygon.io`
- Alpha Vantage: `https://www.alphavantage.co/query`
- DBnomics: `https://api.db.nomics.world`
- Fyers: `https://api-t1.fyers.in`

---

## Important Files to Know

### Frontend Entry Points

1. **`src/main.tsx`** (14 lines)
   - Renders root React component
   - Wraps in `<AuthProvider>`
   - Imports global CSS
   - Initializes Tauri CORS plugin

2. **`src/App.tsx`** (~500 lines)
   - Root component
   - Manages screen routing (login, register, dashboard, etc.)
   - Handles payment overlay
   - Sets up all context providers:
     - NavigationProvider
     - ThemeProvider
     - DataSourceProvider
     - ProviderProvider

3. **`src/components/dashboard/DashboardScreen.tsx`** (~800 lines)
   - Main dashboard container
   - Tab system (40+ tabs)
   - Menu bar (File, Edit, View, etc.)
   - Auto-updater integration
   - Keyboard shortcuts

---

### Key Context Files

4. **`src/contexts/AuthContext.tsx`** (~450 lines)
   - **Most critical context**
   - Manages user session, login, logout
   - Subscription management
   - API connectivity checks
   - Guest vs registered user logic

5. **`src/contexts/DataSourceContext.tsx`** (~150 lines)
   - Manages data source connections
   - CRUD operations for connections
   - Test connection logic
   - SQLite persistence

---

### Core Services

6. **`src/services/authApi.tsx`** (~400 lines)
   - Backend API client for auth
   - All auth-related API calls
   - **Base URL defined here** (line 8)

7. **`src/services/sqliteService.ts`** (~1200 lines)
   - **Critical**: SQLite database wrapper
   - All database operations
   - Schema initialization
   - Type-safe queries
   - Tables: credentials, settings, llm_configs, chat_sessions, data_sources, mcp_servers, etc.

8. **`src/services/workflowService.ts`** (~200 lines)
   - Workflow management
   - Save/load workflows
   - Execution tracking
   - localStorage persistence

9. **`src/services/mcpManager.ts`** (~600 lines)
   - MCP server lifecycle management
   - Install, start, stop servers
   - Tool aggregation
   - Health monitoring

10. **`src/services/websocket/index.ts`** (~400 lines)
    - WebSocket connection manager
    - Topic-based subscriptions
    - Auto-reconnect
    - Multiple provider support

---

### Node Editor (Workflow System)

11. **`src/components/tabs/node-editor/WorkflowExecutor.ts`** (~500 lines)
    - Executes visual workflows
    - Topological sort for node order
    - Error handling & retry logic
    - Result caching

12. **`src/components/tabs/node-editor/DataSourceNode.tsx`**
    - Node for fetching data
    - Uses data source adapters

13. **`src/components/tabs/node-editor/PythonAgentNode.tsx`**
    - Node for running Python agents
    - Uses pythonAgentService

14. **`src/components/tabs/node-editor/MCPToolNode.tsx`**
    - Node for calling MCP tools
    - Uses mcpManager

---

### Backend Entry Points

15. **`src-tauri/src/main.rs`** (~20 lines)
    - Tauri app entry point
    - Calls `lib.rs` to run app

16. **`src-tauri/src/lib.rs`** (~1000 lines)
    - **Most critical Rust file**
    - Registers all Tauri commands
    - MCP server spawning logic
    - Python subprocess management
    - Imports all command modules

17. **`src-tauri/src/utils/python.rs`** (~300 lines)
    - Python interpreter detection
    - Bundled Python path resolution
    - Python script execution helper

---

### Configuration Files

18. **`src-tauri/tauri.conf.json`** (~80 lines)
    - Tauri app configuration
    - Window settings, icons, bundle config
    - Auto-updater endpoint
    - Plugin configs

19. **`vite.config.ts`** (~50 lines)
    - Vite bundler config
    - Proxy config for backend API (**important**)
    - Build optimization (chunk splitting)
    - TypeScript path aliases

20. **`package.json`**
    - Node.js dependencies
    - Scripts (dev, build, tauri)
    - Key deps: React 19, Tauri 2.0, TailwindCSS 4

21. **`src-tauri/Cargo.toml`**
    - Rust dependencies
    - Tauri plugins: sql, http, shell, dialog, updater, etc.

---

### Data Mapping System

22. **`src/components/tabs/data-mapping/engine/MappingEngine.ts`** (~800 lines)
    - Universal API-to-data mapper
    - Supports JSONPath, JMESPath, regex, scripts
    - Schema validation
    - Caching

---

### Python Analytics Examples

23. **`src-tauri/resources/scripts/Analytics/portfolioManagement/sharpe_ratio.py`**
    - Calculates portfolio Sharpe ratio
    - Example of Python analytics script structure

24. **`src-tauri/resources/scripts/agents/TraderInvestorsAgent/warren_buffett.py`**
    - Warren Buffett investment persona
    - Example of AI agent structure

---

## Risks, Tech Debt & Recommendations

### 🔴 Critical Risks

#### 1. **Backend Dependency (Single Point of Failure)**
- **Risk**: Backend at `finceptbackend.share.zrok.io` is external, not part of this repo
- **Impact**: If backend is down, entire app is unusable (no auth, no subscription checks)
- **Evidence**: Hard-coded URL in `authApi.tsx` line 8
- **Recommendation**: 
  - Add offline mode (local SQLite-only auth)
  - Bundle a lightweight local backend (FastAPI) with the app
  - Add health check + graceful degradation

#### 2. **No Token Expiry or Refresh**
- **Risk**: `api_key` stored in SQLite never expires
- **Impact**: Stolen api_key = permanent access to user account
- **Evidence**: `AuthContext.tsx` - no expiry check, no token refresh
- **Recommendation**: 
  - Implement JWT with 1-hour access tokens + refresh tokens
  - Add session timeout (e.g., 24 hours inactivity)
  - Add token rotation on sensitive actions

#### 3. **API Key Stored in Plaintext**
- **Risk**: SQLite database stores `api_key` unencrypted
- **Impact**: Anyone with file system access can steal api_key
- **Evidence**: `sqliteService.ts` - no encryption layer
- **Recommendation**: 
  - Encrypt SQLite database (use SQLCipher)
  - Use OS keychain/credential manager (macOS Keychain, Windows Credential Manager)
  - Implement app-level encryption for sensitive fields

#### 4. **Python Subprocess Security**
- **Risk**: Python scripts run with full file system access
- **Impact**: Malicious script could delete files, exfiltrate data
- **Evidence**: `commands/analytics.rs` - no sandboxing
- **Recommendation**: 
  - Sandbox Python execution (use containers or restricted user)
  - Validate script paths (prevent arbitrary script execution)
  - Implement script signing/verification

#### 5. **WebSocket Connection Credential Exposure**
- **Risk**: WebSocket API keys stored in React state (memory)
- **Impact**: XSS attack could steal API keys
- **Evidence**: `ProviderContext.tsx` stores credentials in context
- **Recommendation**: 
  - Move credential storage to secure backend/Rust layer
  - Use short-lived session tokens for WebSocket connections
  - Implement CSP (Content Security Policy) headers

---

### ⚠️ Major Tech Debt

#### 6. **No Automated Tests**
- **Issue**: 397 TS/TSX files, 63 RS files - **zero test coverage**
- **Impact**: Regressions go undetected, refactoring is risky
- **Evidence**: No `*.test.ts`, `*.spec.ts`, or `tests/` directory
- **Recommendation**: 
  - Add Vitest for frontend unit tests
  - Add Rust tests (`#[cfg(test)]`)
  - Start with critical paths: auth, data sources, workflow execution
  - Target 60% coverage for core services

#### 7. **localStorage for Critical State**
- **Issue**: Workflows stored in localStorage (not SQLite)
- **Impact**: Data loss if browser clears cache, no backup
- **Evidence**: `workflowService.ts` line 22-32
- **Recommendation**: 
  - Migrate workflows to SQLite
  - Add export/import functionality
  - Implement cloud sync (optional premium feature)

#### 8. **Massive Component Files**
- **Issue**: Many components > 800 lines (e.g., DashboardScreen.tsx, NodeEditorTab.tsx)
- **Impact**: Hard to maintain, high cognitive load
- **Evidence**: File sizes from `find src -name "*.tsx" -exec wc -l {} \;`
- **Recommendation**: 
  - Split into smaller components (< 300 lines)
  - Extract hooks (e.g., `useDashboard`, `useWorkflow`)
  - Use component composition patterns

#### 9. **Duplicated API Logic**
- **Issue**: Many services have copy-pasted fetch/error handling
- **Impact**: Inconsistent error handling, harder to add retry logic
- **Evidence**: Compare `authApi.tsx`, `paymentApi.tsx`, `userApi.tsx`
- **Recommendation**: 
  - Create unified API client (use Axios or custom fetch wrapper)
  - Centralize error handling (interceptors)
  - Add request/response logging

#### 10. **TypeScript `any` Overuse**
- **Issue**: Many functions use `any` for parameters/return types
- **Impact**: Loss of type safety, runtime errors
- **Evidence**: `sqliteService.ts` line 54: `data?: any`
- **Recommendation**: 
  - Define proper TypeScript interfaces for all data structures
  - Enable strict mode in tsconfig.json
  - Use `unknown` instead of `any` for gradual typing

#### 11. **No Error Boundaries**
- **Issue**: React components don't have error boundaries
- **Impact**: One component crash takes down entire app
- **Evidence**: No `<ErrorBoundary>` in App.tsx or DashboardScreen.tsx
- **Recommendation**: 
  - Add top-level ErrorBoundary in App.tsx
  - Add per-tab ErrorBoundaries in DashboardScreen.tsx
  - Implement error reporting (Sentry, Rollbar)

#### 12. **Unhandled Promise Rejections**
- **Issue**: Many async functions don't handle errors
- **Impact**: Silent failures, no user feedback
- **Evidence**: `workflowService.ts` - missing try/catch in many places
- **Recommendation**: 
  - Add try/catch to all async functions
  - Use toast notifications for user-facing errors
  - Log errors to SQLite for debugging

---

### 🟡 Medium Priority Issues

#### 13. **No Logging System**
- **Issue**: `console.log` everywhere, no structured logging
- **Impact**: Hard to debug production issues
- **Recommendation**: 
  - Add structured logging library (winston, pino)
  - Log to SQLite table + file
  - Add log level filtering (debug, info, warn, error)

#### 14. **Hard-Coded URLs**
- **Issue**: Many API URLs hard-coded in source files
- **Impact**: Hard to switch environments (dev/staging/prod)
- **Evidence**: `authApi.tsx` line 8, `vite.config.ts` line 37
- **Recommendation**: 
  - Move URLs to environment variables
  - Create config file (src/config.ts)
  - Use `import.meta.env.VITE_*` pattern

#### 15. **No Rate Limiting**
- **Issue**: API calls not rate-limited on frontend
- **Impact**: Could hit API rate limits, waste credits
- **Recommendation**: 
  - Add client-side rate limiting (use p-queue)
  - Debounce/throttle frequent calls (useDebounce hook)
  - Cache API responses (React Query or custom cache)

#### 16. **Memory Leaks Potential**
- **Issue**: WebSocket connections, Python subprocesses not always cleaned up
- **Impact**: Memory usage grows over time
- **Evidence**: `websocket/index.ts` - no cleanup on unmount
- **Recommendation**: 
  - Use `useEffect` cleanup functions
  - Track all spawned processes, kill on app exit
  - Add memory monitoring (heap snapshots)

#### 17. **No Accessibility (a11y)**
- **Issue**: Many components missing ARIA labels, keyboard navigation
- **Impact**: Unusable for screen reader users
- **Recommendation**: 
  - Add ARIA labels to interactive elements
  - Ensure keyboard navigation works (Tab, Enter, Escape)
  - Test with screen reader (NVDA, JAWS)

---

### ✅ Safe to Extend (Low-Risk Areas)

These areas have good abstraction and are safe to modify:

1. **Data Source Adapters** (`src/components/tabs/data-sources/adapters/`)
   - Well-abstracted with BaseAdapter
   - Add new adapter by extending BaseAdapter
   - Register in `adapters/index.ts`

2. **UI Components** (`src/components/ui/`)
   - shadcn/ui components are isolated
   - Safe to customize styles

3. **Python Analytics Scripts** (`src-tauri/resources/scripts/Analytics/`)
   - Self-contained Python scripts
   - Add new scripts without breaking existing ones
   - Follow existing JSON input/output pattern

4. **AI Agent Personas** (`src-tauri/resources/scripts/agents/`)
   - Self-contained agent scripts
   - Add new agents by copying existing structure

5. **Workflow Nodes** (`src/components/tabs/node-editor/`)
   - Modular node system
   - Add new node type by creating new component
   - Register in WorkflowExecutor.ts

---

### 🚧 High-Risk Refactoring Areas (Handle with Care)

These areas are fragile and tightly coupled:

1. **AuthContext** (`src/contexts/AuthContext.tsx`)
   - Used by 50+ components
   - Changing interface breaks many components
   - **Recommendation**: Add deprecation warnings, migrate gradually

2. **SQLite Schema** (`src/services/sqliteService.ts`)
   - No migration system
   - Schema changes require manual updates
   - **Recommendation**: Add migration framework (like Knex.js or custom)

3. **Workflow Execution** (`src/components/tabs/node-editor/WorkflowExecutor.ts`)
   - Complex state machine
   - Many edge cases
   - **Recommendation**: Add unit tests before refactoring

4. **WebSocket Manager** (`src/services/websocket/index.ts`)
   - Singleton with global state
   - Tricky reconnection logic
   - **Recommendation**: Add integration tests, refactor to React Context

5. **MCP Server Spawning** (`src-tauri/src/lib.rs`)
   - Complex process management
   - Platform-specific code (Windows vs Unix)
   - **Recommendation**: Add error handling, test on all platforms

---

### 📊 Performance Bottlenecks

1. **Initial Bundle Size**: ~15 MB (React, charts, UI components)
   - **Recommendation**: Lazy load tabs (`React.lazy`)

2. **Python Subprocess Startup**: ~500ms per script
   - **Recommendation**: Keep Python process alive (daemon mode)

3. **SQLite Queries**: No indexes on large tables
   - **Recommendation**: Add indexes on `data_sources.provider`, `chat_messages.session_uuid`

4. **Real-time Chart Rendering**: Can lag with 1000+ data points
   - **Recommendation**: Use data downsampling (e.g., LTTB algorithm)

---

### 🔒 Security Recommendations

1. **Implement Content Security Policy (CSP)**
2. **Add input validation on all user inputs** (prevent XSS, SQL injection)
3. **Sanitize file paths** before passing to Rust commands (prevent directory traversal)
4. **Implement rate limiting** on expensive operations (Python execution, MCP calls)
5. **Add audit logging** for sensitive actions (login, payment, data export)
6. **Encrypt WebSocket connections** (WSS instead of WS)
7. **Validate Python script signatures** before execution
8. **Implement CORS properly** (not just `csp: null`)

---

### 🎯 Immediate Action Items (Priority Order)

1. **Add Authentication Token Expiry** (AuthContext.tsx)
   - Implement JWT with refresh tokens
   - Add 24-hour session timeout
   - Est. effort: 2-3 days

2. **Encrypt SQLite Database** (sqliteService.ts)
   - Migrate to SQLCipher or add app-level encryption
   - Encrypt api_key field at minimum
   - Est. effort: 1-2 days

3. **Add Error Boundaries** (App.tsx, DashboardScreen.tsx)
   - Catch React errors
   - Display user-friendly error screens
   - Est. effort: 1 day

4. **Implement Offline Mode** (AuthContext.tsx, authApi.tsx)
   - Allow app usage without backend
   - Local-only auth with reduced features
   - Est. effort: 3-4 days

5. **Add Unit Tests** (Start with authApi.ts, sqliteService.ts)
   - Set up Vitest
   - Write tests for critical services
   - Est. effort: 1 week

6. **Migrate Workflows to SQLite** (workflowService.ts)
   - Create `workflows` table
   - Add migration from localStorage
   - Est. effort: 2 days

7. **Add Logging System** (Create logger.ts)
   - Structured logging with levels
   - Log to SQLite + file
   - Est. effort: 2 days

8. **Reduce Bundle Size** (Lazy load tabs)
   - Convert tabs to React.lazy
   - Reduce initial load time
   - Est. effort: 2-3 days

---

## Conclusion

**Fincept Terminal is a powerful, ambitious platform** with:

✅ **Strengths**:
- Rich feature set (40+ tabs, 93+ data adapters, AI agents)
- Modern tech stack (React 19, Tauri 2.0, Python 3.11)
- Extensible architecture (adapters, nodes, MCP)
- Active development (v3.0.11)

⚠️ **Weaknesses**:
- Critical security issues (plaintext keys, no expiry, no encryption)
- Heavy tech debt (no tests, large files, localStorage overuse)
- Backend dependency (single point of failure)
- Performance bottlenecks (bundle size, subprocess startup)

**Before Making Big Changes**:
1. Read this document thoroughly
2. Set up local dev environment (`npm run tauri dev`)
3. Understand AuthContext and SQLiteService (most critical)
4. Add tests for the area you're changing
5. Test on all platforms (Windows, macOS, Linux)

**Safe Entry Points for New Contributors**:
- Add new data source adapter (extend BaseAdapter)
- Add new Python analytics script (copy existing structure)
- Add new AI agent persona (copy existing agent)
- Add new workflow node (extend node types)
- Improve UI/UX in individual tabs (mostly isolated)

**High-Risk Changes (Require Careful Planning)**:
- Modify AuthContext or session management
- Change SQLite schema or sqliteService
- Refactor WebSocket manager
- Change workflow execution logic
- Modify Tauri command structure

---

**Questions?** Open an issue on GitHub: https://github.com/Fincept-Corporation/FinceptTerminal/issues

**Want to Contribute?** See [CONTRIBUTING.md](CONTRIBUTING.md) and [DEVELOPER_WALKTHROUGH.md](DEVELOPER_WALKTHROUGH.md)

**Need Help?** Join Discord: https://discord.gg/ae87a8ygbN

---

**Document Version:** 1.0  
**Author:** Senior Full-Stack Engineer (Codebase Analysis)  
**Last Reviewed:** 2025-01-05
