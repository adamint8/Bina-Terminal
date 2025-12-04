# Bina Adaptation Roadmap
# Fincept Terminal → Bina Analytics Control Console

**Purpose:** Phased implementation plan for transforming Fincept Terminal into Bina Analytics  
**Version:** 1.0  
**Date:** December 2024  
**Timeline:** 8 weeks (estimated)

---

## Overview

This roadmap breaks down the adaptation into 4 phases over 8 weeks:
- **Phase 1:** Foundation (Weeks 1-2) - Branding, cleanup, basic setup
- **Phase 2:** Core Integration (Weeks 3-4) - Supabase, n8n, data sources
- **Phase 3:** Feature Adaptation (Weeks 5-6) - Research tasks, intel states, reports
- **Phase 4:** Polish & Testing (Weeks 7-8) - Refinement, testing, deployment

**Each task includes:**
- ✅ Description
- 📁 Files to modify
- 🔧 Approach
- ⚠️ Potential challenges

---

## Phase 1: Foundation (Weeks 1-2)

**Goal:** Rebrand, clean up unused features, establish development environment

### Week 1: Rebranding & Cleanup

#### Task 1.1: Update App Branding
**Estimated Time:** 4 hours

**Description:** Change all instances of "Fincept Terminal" to "Bina Analytics Control Console"

**Files to Modify:**
- `src-tauri/tauri.conf.json` (lines 3, 5, 16, 42-46)
- `package.json` (line 2)
- `index.html` (title tag)
- `src/constants/version.ts` (add APP_NAME constant)
- `src/App.tsx` (line 208 - loading message)
- `src/components/common/Header.tsx` (logo/title)
- `src/components/common/Footer.tsx` (copyright)
- `README.md` (entire file - rewrite for Bina)

**Approach:**
1. Use find-and-replace for "Fincept" → "Bina"
2. Use find-and-replace for "Terminal" → "Control Console" (context-aware)
3. Update product identifier: `com.fincept.terminal` → `com.bina.analytics`
4. Reset version to `1.0.0`

**Potential Challenges:**
- Some "Terminal" references may be intentional (e.g., terminal UI theme)
- Be careful not to replace in code comments explaining original Fincept architecture

**Verification:**
```bash
grep -r "Fincept" src/  # Should find minimal results
grep -r "FinceptTerminal" .  # Check for missed references
```

---

#### Task 1.2: Replace Icons and Logo
**Estimated Time:** 2 hours

**Description:** Replace app icons with Bina branding

**Files to Modify:**
- `src-tauri/icons/` (all PNG, ICO, ICNS files)
- `public/favicon.ico`
- `public/` (add `bina-logo.svg` or `bina-logo.png`)

**Approach:**
1. Design Bina logo (or obtain from brand team)
2. Create 1024x1024 PNG source image
3. Generate icons using Tauri CLI:
   ```bash
   cd src-tauri
   cargo tauri icon path/to/bina-logo.png
   ```
4. Replace favicon in `public/`
5. Update Header component to use new logo

**Potential Challenges:**
- Icon generation requires exact dimensions
- May need manual tweaking for best results

**Verification:**
- Build app and check taskbar/dock icon
- Open app and verify Header logo

---

#### Task 1.3: Remove Payment & Subscription Flow
**Estimated Time:** 3 hours

**Description:** Delete entire payment and pricing functionality

**Files to DELETE:**
- `src/services/paymentApi.tsx` (630 lines)
- `src/components/payment/` (entire directory)
  - `InAppPaymentWindow.tsx`
  - `PaymentOverlay.tsx`
  - `PaymentProcessingScreen.tsx`
  - `PaymentSuccessScreen.tsx`
- `src/components/auth/PricingScreen.tsx`

**Files to MODIFY:**
- `src/App.tsx`:
  - Remove `'pricing' | 'paymentProcessing' | 'paymentSuccess'` from Screen type (line 31-43)
  - Remove pricing screen logic (lines 145-170, 299-314)
  - Remove payment window state (lines 46-66, 69-109)
  - Remove PaymentOverlay component (lines 227, 241, 346)
- `src/contexts/AuthContext.tsx`:
  - Remove subscription-related types (lines 62-100)
  - Remove subscription state from session
  - Simplify to basic auth only

**Approach:**
1. Delete payment-related files
2. Remove imports in App.tsx
3. Simplify authentication flow to skip pricing
4. Update AuthContext to remove subscription logic
5. Clean up unused dependencies:
   ```bash
   npm uninstall react-confetti  # Used in payment success screen
   ```

**Potential Challenges:**
- Many references to subscription in conditional logic
- May break authentication flow if not careful

**Verification:**
```bash
npm run build  # Should compile without errors
npm run dev    # Test login flow
```

---

#### Task 1.4: Remove Trading Features
**Estimated Time:** 4 hours

**Description:** Delete all trading-specific tabs and services

**Files to DELETE:**
- `src/components/tabs/TradingTab.tsx`
- `src/components/tabs/fyers/` (entire directory)
- `src/components/tabs/KrakenTab.tsx`
- `src/components/tabs/BacktestingTab.tsx` (optional - might repurpose)
- `src/components/tabs/PortfolioTab.tsx` (or repurpose for client portfolios)
- `src/components/tabs/trading/` (entire directory)
- `src/components/tabs/portfolio/` (entire directory)
- `src/services/fyersService.ts`
- `src/services/fyersAuth.ts`
- `src/services/fyersWebSocket.ts`
- `src/services/hyperliquid/` (entire directory)
- `src/services/backtesting/` (optional)
- `src/stockBrokers/` (entire directory)
- `src/hooks/usePaperTrading.ts`
- `src/components/settings/BacktestingProvidersPanel.tsx`

**Files to MODIFY:**
- `src/components/dashboard/DashboardScreen.tsx`:
  - Remove trading tab imports (lines 39, 35, 38, 41)
  - Remove TabsTrigger for trading tabs
  - Remove TabsContent for trading tabs
  - Remove trading-related menu items

**Approach:**
1. Delete all files listed above
2. Remove imports in DashboardScreen
3. Remove tab triggers and content
4. Clean up dependencies:
   ```bash
   npm uninstall fyers-web-sdk-v3
   ```
5. Test compilation

**Potential Challenges:**
- Portfolio tab might have reusable components for client portfolio management
- Backtesting might be useful for research validation

**Verification:**
```bash
npm run build
npm run dev  # Verify app still loads without trading tabs
```

---

### Week 2: Environment Setup & Supabase Foundation

#### Task 1.5: Set Up Supabase Project
**Estimated Time:** 3 hours

**Description:** Create Supabase project and initial database schema

**No code files modified yet - infrastructure setup**

**Approach:**
1. Go to [supabase.com](https://supabase.com)
2. Create new project: "bina-analytics"
3. Note project URL and anon key
4. Create database tables (SQL in Supabase SQL Editor):

```sql
-- Users table (handled by Supabase Auth)

-- Intel States
CREATE TABLE intel_states (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  entity_id VARCHAR NOT NULL,
  entity_type VARCHAR NOT NULL CHECK (entity_type IN ('portfolio', 'company', 'sector', 'market')),
  state JSONB NOT NULL DEFAULT '{}',
  last_updated TIMESTAMP DEFAULT NOW(),
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_intel_states_entity ON intel_states(entity_id, entity_type);

-- Research Tasks
CREATE TABLE research_tasks (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  agent_type VARCHAR NOT NULL,
  task_description TEXT NOT NULL,
  context JSONB DEFAULT '{}',
  entity_id VARCHAR,
  priority VARCHAR DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high')),
  status VARCHAR DEFAULT 'pending' CHECK (status IN ('pending', 'running', 'completed', 'failed')),
  result JSONB,
  error TEXT,
  progress INTEGER DEFAULT 0,
  logs TEXT[],
  created_at TIMESTAMP DEFAULT NOW(),
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  created_by UUID REFERENCES auth.users(id)
);

CREATE INDEX idx_research_tasks_status ON research_tasks(status);
CREATE INDEX idx_research_tasks_created_by ON research_tasks(created_by);

-- Market Data
CREATE TABLE market_data (
  id BIGSERIAL PRIMARY KEY,
  symbol VARCHAR NOT NULL,
  timestamp TIMESTAMP NOT NULL,
  price NUMERIC,
  volume BIGINT,
  source VARCHAR,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_market_data_symbol ON market_data(symbol, timestamp DESC);

-- News Articles
CREATE TABLE news_articles (
  id BIGSERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT,
  source VARCHAR,
  url VARCHAR UNIQUE,
  published_at TIMESTAMP,
  sentiment NUMERIC,
  entities JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_news_articles_published ON news_articles(published_at DESC);

-- Client Reports
CREATE TABLE client_reports (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  client_id UUID NOT NULL,
  report_type VARCHAR NOT NULL,
  portfolio_id VARCHAR,
  content JSONB NOT NULL,
  generated_at TIMESTAMP DEFAULT NOW(),
  delivered_at TIMESTAMP,
  status VARCHAR DEFAULT 'draft' CHECK (status IN ('draft', 'generated', 'delivered')),
  metadata JSONB DEFAULT '{}'
);

-- Workflows
CREATE TABLE workflows (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR NOT NULL,
  description TEXT,
  nodes JSONB NOT NULL,
  edges JSONB NOT NULL,
  status VARCHAR DEFAULT 'draft' CHECK (status IN ('draft', 'active', 'archived')),
  n8n_workflow_id VARCHAR,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES auth.users(id)
);

-- Workflow Executions
CREATE TABLE workflow_executions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  workflow_id UUID REFERENCES workflows(id),
  n8n_execution_id VARCHAR,
  status VARCHAR DEFAULT 'running' CHECK (status IN ('running', 'completed', 'failed')),
  started_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  result JSONB,
  error TEXT
);

-- Enable Row Level Security (RLS)
ALTER TABLE intel_states ENABLE ROW LEVEL SECURITY;
ALTER TABLE research_tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE client_reports ENABLE ROW LEVEL SECURITY;
ALTER TABLE workflows ENABLE ROW LEVEL SECURITY;

-- Create policies (example for research_tasks)
CREATE POLICY "Users can view their own tasks"
  ON research_tasks FOR SELECT
  USING (auth.uid() = created_by);

CREATE POLICY "Users can create tasks"
  ON research_tasks FOR INSERT
  WITH CHECK (auth.uid() = created_by);

-- Add more policies as needed
```

5. Enable Realtime for `research_tasks` table
6. Save credentials securely

**Potential Challenges:**
- Database schema may need adjustments based on Bina requirements
- RLS policies need careful configuration

---

#### Task 1.6: Install Supabase Client
**Estimated Time:** 2 hours

**Description:** Add Supabase SDK and create base service layer

**Files to CREATE:**
- `src/services/supabase/supabaseClient.ts`
- `src/services/supabase/supabaseAuth.ts`
- `.env` (if doesn't exist)

**Files to MODIFY:**
- `package.json` (add dependency)

**Approach:**

1. **Install Supabase:**
```bash
npm install @supabase/supabase-js
```

2. **Create environment file (`.env`):**
```bash
# Supabase
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# n8n (placeholder for Phase 2)
VITE_N8N_WEBHOOK_URL=
VITE_N8N_API_URL=
VITE_N8N_API_KEY=

# LLM providers (existing)
VITE_OPENAI_API_KEY=
VITE_ANTHROPIC_API_KEY=
```

3. **Create Supabase client:**

**File:** `src/services/supabase/supabaseClient.ts`
```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

4. **Create auth service:**

**File:** `src/services/supabase/supabaseAuth.ts`
```typescript
import { supabase } from './supabaseClient';

export const supabaseAuth = {
  async signIn(email: string, password: string) {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });
    if (error) throw error;
    return data;
  },

  async signUp(email: string, password: string) {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
    });
    if (error) throw error;
    return data;
  },

  async signOut() {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;
  },

  async getSession() {
    const { data, error } = await supabase.auth.getSession();
    if (error) throw error;
    return data.session;
  },

  onAuthStateChange(callback: (session: any) => void) {
    return supabase.auth.onAuthStateChange((event, session) => {
      callback(session);
    });
  },
};
```

5. **Create index file:**

**File:** `src/services/supabase/index.ts`
```typescript
export * from './supabaseClient';
export * from './supabaseAuth';
```

**Potential Challenges:**
- Environment variables must be prefixed with `VITE_`
- Need to restart dev server after adding env vars

**Verification:**
```bash
npm run build  # Should compile successfully
```

---

## Phase 2: Core Integration (Weeks 3-4)

**Goal:** Replace backend API with Supabase, integrate n8n, refactor data sources

### Week 3: Supabase Integration

#### Task 2.1: Replace AuthContext with Supabase Auth
**Estimated Time:** 6 hours

**Description:** Migrate authentication from custom backend to Supabase

**Files to MODIFY:**
- `src/contexts/AuthContext.tsx` (major refactor)
- `src/components/auth/LoginScreen.tsx`
- `src/components/auth/RegisterScreen.tsx`
- `src/components/auth/ForgotPasswordScreen.tsx`
- `src/App.tsx` (authentication flow logic)

**Approach:**

**File:** `src/contexts/AuthContext.tsx`

**Replace entire authentication logic:**

```typescript
import React, { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { supabaseAuth } from '@/services/supabase';
import { Session, User } from '@supabase/supabase-js';

interface AuthContextState {
  session: Session | null;
  user: User | null;
  isLoading: boolean;
}

interface AuthContextValue extends AuthContextState {
  signIn: (email: string, password: string) => Promise<void>;
  signUp: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
  resetPassword: (email: string) => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [session, setSession] = useState<Session | null>(null);
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Get initial session
    supabaseAuth.getSession().then(initialSession => {
      setSession(initialSession);
      setUser(initialSession?.user ?? null);
      setIsLoading(false);
    });

    // Listen for auth changes
    const { data: authListener } = supabaseAuth.onAuthStateChange(newSession => {
      setSession(newSession);
      setUser(newSession?.user ?? null);
    });

    return () => {
      authListener.subscription.unsubscribe();
    };
  }, []);

  const signIn = async (email: string, password: string) => {
    const { session: newSession } = await supabaseAuth.signIn(email, password);
    setSession(newSession);
    setUser(newSession?.user ?? null);
  };

  const signUp = async (email: string, password: string) => {
    const { session: newSession } = await supabaseAuth.signUp(email, password);
    // Supabase may require email verification
    setSession(newSession);
    setUser(newSession?.user ?? null);
  };

  const signOut = async () => {
    await supabaseAuth.signOut();
    setSession(null);
    setUser(null);
  };

  const resetPassword = async (email: string) => {
    await supabaseAuth.resetPassword(email);
  };

  return (
    <AuthContext.Provider
      value={{ session, user, isLoading, signIn, signUp, signOut, resetPassword }}
    >
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

**Update auth screens to use new context:**

**File:** `src/components/auth/LoginScreen.tsx`

**Replace login handler:**
```typescript
const handleLogin = async (e: React.FormEvent) => {
  e.preventDefault();
  setError('');
  setLoading(true);

  try {
    await signIn(email, password);  // Uses new AuthContext
    onNavigate('dashboard');
  } catch (error: any) {
    setError(error.message || 'Login failed');
  } finally {
    setLoading(false);
  }
};
```

**Potential Challenges:**
- Supabase auth flow different from custom backend
- Email verification may be required
- Guest mode needs alternative implementation

**Verification:**
```bash
npm run dev
# Test: Register new user, login, logout
```

---

#### Task 2.2: Create Research Task Service
**Estimated Time:** 4 hours

**Description:** Build service layer for managing research tasks

**Files to CREATE:**
- `src/services/supabase/supabaseResearch.ts`

**Approach:**

```typescript
import { supabase } from './supabaseClient';

export interface ResearchTask {
  id: string;
  agent_type: string;
  task_description: string;
  context: any;
  entity_id?: string;
  priority: 'low' | 'medium' | 'high';
  status: 'pending' | 'running' | 'completed' | 'failed';
  result?: any;
  error?: string;
  progress?: number;
  logs?: string[];
  created_at: string;
  started_at?: string;
  completed_at?: string;
}

export async function createResearchTask(
  agentType: string,
  description: string,
  context: any = {},
  entityId?: string,
  priority: 'low' | 'medium' | 'high' = 'medium'
): Promise<string> {
  const { data, error } = await supabase
    .from('research_tasks')
    .insert({
      agent_type: agentType,
      task_description: description,
      context,
      entity_id: entityId,
      priority,
      status: 'pending',
    })
    .select()
    .single();

  if (error) throw error;
  return data.id;
}

export async function listResearchTasks(filters?: {
  status?: string;
  agentType?: string;
  entityId?: string;
}): Promise<ResearchTask[]> {
  let query = supabase
    .from('research_tasks')
    .select('*')
    .order('created_at', { ascending: false });

  if (filters?.status) query = query.eq('status', filters.status);
  if (filters?.agentType) query = query.eq('agent_type', filters.agentType);
  if (filters?.entityId) query = query.eq('entity_id', filters.entityId);

  const { data, error } = await query;
  if (error) throw error;
  return data;
}

export async function getResearchTask(id: string): Promise<ResearchTask> {
  const { data, error } = await supabase
    .from('research_tasks')
    .select('*')
    .eq('id', id)
    .single();

  if (error) throw error;
  return data;
}

export async function updateResearchTask(
  id: string,
  updates: Partial<ResearchTask>
): Promise<void> {
  const { error } = await supabase
    .from('research_tasks')
    .update(updates)
    .eq('id', id);

  if (error) throw error;
}

export function subscribeToResearchTasks(
  callback: (task: ResearchTask) => void,
  filters?: { taskId?: string }
) {
  let channel = supabase.channel('research_tasks_channel');

  if (filters?.taskId) {
    channel = channel.on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'research_tasks',
        filter: `id=eq.${filters.taskId}`,
      },
      (payload) => {
        callback(payload.new as ResearchTask);
      }
    );
  } else {
    channel = channel.on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'research_tasks',
      },
      (payload) => {
        callback(payload.new as ResearchTask);
      }
    );
  }

  const subscription = channel.subscribe();

  return {
    unsubscribe: () => subscription.unsubscribe(),
  };
}

// Export all functions
export const supabaseResearch = {
  createTask: createResearchTask,
  listTasks: listResearchTasks,
  getTask: getResearchTask,
  updateTask: updateResearchTask,
  subscribeToTasks: subscribeToResearchTasks,
};
```

**Update index:**

**File:** `src/services/supabase/index.ts`
```typescript
export * from './supabaseClient';
export * from './supabaseAuth';
export * from './supabaseResearch';  // ADD
```

**Verification:**
```typescript
// Test in browser console or temporary component
import { supabaseResearch } from '@/services/supabase';

const taskId = await supabaseResearch.createTask(
  'market_analyst',
  'Analyze AAPL stock',
  { symbol: 'AAPL' }
);
console.log('Created task:', taskId);

const tasks = await supabaseResearch.listTasks({ status: 'pending' });
console.log('Pending tasks:', tasks);
```

---

### Week 4: n8n Integration & Data Sources

#### Task 2.3: Set Up n8n Integration Service
**Estimated Time:** 5 hours

**Description:** Create service to interact with n8n workflows

**Files to CREATE:**
- `src/services/n8n/n8nClient.ts`
- `src/services/n8n/taskTemplates.ts`

**Approach:**

**File:** `src/services/n8n/n8nClient.ts`

```typescript
const N8N_API_URL = import.meta.env.VITE_N8N_API_URL;
const N8N_API_KEY = import.meta.env.VITE_N8N_API_KEY;

export interface N8NWorkflowExecution {
  id: string;
  workflowId: string;
  status: 'running' | 'success' | 'error';
  data: any;
  error?: string;
}

export const n8nClient = {
  async triggerWorkflow(workflowId: string, data: any): Promise<string> {
    const response = await fetch(`${N8N_API_URL}/webhook/${workflowId}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-N8N-API-KEY': N8N_API_KEY,
      },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      throw new Error(`Failed to trigger workflow: ${response.statusText}`);
    }

    const result = await response.json();
    return result.executionId;
  },

  async getExecutionStatus(executionId: string): Promise<N8NWorkflowExecution> {
    const response = await fetch(`${N8N_API_URL}/executions/${executionId}`, {
      headers: {
        'X-N8N-API-KEY': N8N_API_KEY,
      },
    });

    if (!response.ok) {
      throw new Error(`Failed to get execution status: ${response.statusText}`);
    }

    return response.json();
  },

  async listWorkflows(): Promise<any[]> {
    const response = await fetch(`${N8N_API_URL}/workflows`, {
      headers: {
        'X-N8N-API-KEY': N8N_API_KEY,
      },
    });

    if (!response.ok) {
      throw new Error(`Failed to list workflows: ${response.statusText}`);
    }

    const result = await response.json();
    return result.data;
  },
};
```

**File:** `src/services/n8n/taskTemplates.ts`

```typescript
export interface TaskTemplate {
  id: string;
  name: string;
  description: string;
  n8nWorkflowId: string;
  category: 'research' | 'intel' | 'report' | 'monitoring';
  parameters: TaskParameter[];
}

export interface TaskParameter {
  name: string;
  type: 'text' | 'number' | 'date' | 'select' | 'entity';
  label: string;
  required: boolean;
  defaultValue?: any;
  options?: string[];
}

export const TASK_TEMPLATES: TaskTemplate[] = [
  {
    id: 'daily_market_intel',
    name: 'Daily Market Intelligence',
    description: 'Collect and analyze daily market data',
    n8nWorkflowId: 'wf_001',  // Replace with actual n8n workflow ID
    category: 'intel',
    parameters: [
      {
        name: 'markets',
        type: 'select',
        label: 'Markets',
        required: true,
        options: ['US', 'EU', 'APAC', 'Global'],
        defaultValue: 'US',
      },
    ],
  },
  {
    id: 'company_research',
    name: 'Company Research Report',
    description: 'Generate comprehensive company research',
    n8nWorkflowId: 'wf_002',
    category: 'research',
    parameters: [
      {
        name: 'ticker',
        type: 'text',
        label: 'Stock Ticker',
        required: true,
      },
      {
        name: 'depth',
        type: 'select',
        label: 'Analysis Depth',
        required: true,
        options: ['shallow', 'standard', 'deep'],
        defaultValue: 'standard',
      },
    ],
  },
  // Add more templates as needed
];

export function getTemplateById(id: string): TaskTemplate | undefined {
  return TASK_TEMPLATES.find(t => t.id === id);
}
```

**Potential Challenges:**
- n8n API authentication varies by deployment
- Webhook URLs vs. API endpoints
- May need to handle CORS

---

#### Task 2.4: Simplify Data Sources
**Estimated Time:** 4 hours

**Description:** Remove trading-specific data source adapters

**Files to DELETE:**
```
src/components/tabs/data-sources/adapters/BinanceAdapter.ts
src/components/tabs/data-sources/adapters/KrakenAdapter.ts
src/components/tabs/data-sources/adapters/CoinbaseAdapter.ts
src/components/tabs/data-sources/adapters/CoinGeckoAdapter.ts
src/components/tabs/data-sources/adapters/CoinMarketCapAdapter.ts
src/components/tabs/data-sources/adapters/FyersAdapter.ts
src/components/tabs/data-sources/adapters/TradingViewAdapter.ts (if exists)
```

**Files to MODIFY:**
- `src/components/tabs/data-sources/dataSourceConfigs.ts`
  - Remove trading source configurations

**Approach:**
1. Delete adapter files
2. Remove from `dataSourceConfigs.ts`:
```typescript
// REMOVE entries for: binance, kraken, coinbase, coingecko, coinmarketcap, fyers
export const DATA_SOURCE_CONFIGS = DATA_SOURCE_CONFIGS.filter(
  config => !['binance', 'kraken', 'coinbase', 'coingecko', 'coinmarketcap', 'fyers'].includes(config.id)
);
```
3. Keep essential sources:
   - PostgreSQL (for Supabase)
   - Alpha Vantage
   - Polygon.io
   - DBnomics
   - REST API adapter (generic)
   - GraphQL adapter

**Verification:**
```bash
npm run build
npm run dev
# Navigate to Data Sources tab, verify trading sources removed
```

---

## Phase 3: Feature Adaptation (Weeks 5-6)

**Goal:** Build Bina-specific features (research tasks, intel states, reports)

### Week 5: Research Tasks & Intel States

#### Task 3.1: Create Research Tasks Tab
**Estimated Time:** 8 hours

**Description:** Build UI for managing research tasks

**Files to CREATE:**
- `src/components/tabs/ResearchTasksTab.tsx`
- `src/components/tabs/research-tasks/TaskCard.tsx`
- `src/components/tabs/research-tasks/CreateTaskModal.tsx`
- `src/components/tabs/research-tasks/TaskDetailModal.tsx`

**See CUSTOMIZATION_GUIDE.md Task Monitor example for reference**

**Key Features:**
- List all tasks with status
- Real-time updates via Supabase subscriptions
- Create new task from template
- View task details
- Stop running tasks
- Filter by status, agent type

**Verification:**
- Create test task in Supabase
- Verify real-time updates work
- Test filtering and sorting

---

#### Task 3.2: Create Intel State Management UI
**Estimated Time:** 8 hours

**Description:** Build interface for viewing/editing intel states

**Files to CREATE:**
- `src/components/tabs/IntelStateTab.tsx`
- `src/components/tabs/intel-state/EntitySelector.tsx`
- `src/components/tabs/intel-state/StateViewer.tsx`
- `src/components/tabs/intel-state/StateEditor.tsx`
- `src/services/supabase/supabaseIntel.ts`

**Key Features:**
- Select entity (company, sector, portfolio)
- View current intel state (JSON display)
- Edit state manually
- View update history
- Trigger AI update

**Approach:**
Similar to Research Tasks Tab, fetch from `intel_states` table

---

### Week 6: Client Reports & Task Monitor

#### Task 3.3: Build Client Reports Tab
**Estimated Time:** 8 hours

**Description:** Interface for generating and managing client reports

**Files to CREATE:**
- `src/components/tabs/ClientReportsTab.tsx`
- `src/components/tabs/client-reports/ReportBuilder.tsx`
- `src/components/tabs/client-reports/ReportPreview.tsx`
- `src/services/supabase/supabaseReports.ts`

**Key Features:**
- List generated reports
- Create new report from template
- Preview report content
- Deliver report to client (mark as delivered)
- Download as PDF (use library like `jsPDF`)

---

#### Task 3.4: Build Task Monitor Tab
**Estimated Time:** 6 hours

**Description:** Real-time monitoring of running tasks

**See CUSTOMIZATION_GUIDE.md for detailed example**

**Key Features:**
- Live task status
- Progress indicators
- Log streaming
- Stop/restart tasks

---

## Phase 4: Polish & Testing (Weeks 7-8)

**Goal:** Refinement, testing, documentation, deployment

### Week 7: UI Polish & Testing

#### Task 4.1: UI/UX Refinements
**Estimated Time:** 8 hours

**Focus Areas:**
- Consistent color scheme
- Loading states
- Error handling
- Empty states
- Responsive design (if needed)

---

#### Task 4.2: End-to-End Testing
**Estimated Time:** 10 hours

**Test Scenarios:**
1. User registration/login
2. Create research task
3. Monitor task execution
4. View intel state
5. Generate client report
6. Node editor workflow
7. Data source connection
8. Real-time updates

---

### Week 8: Documentation & Deployment

#### Task 4.3: Update Documentation
**Estimated Time:** 6 hours

**Files to UPDATE/CREATE:**
- `README.md` - Bina-specific readme
- `docs/USER_GUIDE.md` - End-user documentation
- `docs/ADMIN_GUIDE.md` - Admin/deployment guide
- `docs/API_REFERENCE.md` - Internal API reference

---

#### Task 4.4: Build & Deploy
**Estimated Time:** 8 hours

**Steps:**
1. Final testing on all platforms
2. Build production bundles:
   ```bash
   npm run tauri build
   ```
3. Create GitHub release
4. Upload installers
5. Configure auto-updater endpoint
6. Deploy backend (Supabase, n8n)
7. Test auto-update mechanism

---

## Risk Mitigation

### High-Risk Items
- **Supabase migration:** Test thoroughly, have rollback plan
- **n8n integration:** May need custom connector development
- **Real-time subscriptions:** Test under load

### Recommended Approach
- **Incremental commits:** Commit after each task
- **Feature flags:** Use environment variables to toggle new features
- **Parallel development:** Work on UI while backend is being set up

---

## Success Metrics

### Phase 1 Complete:
- [ ] App rebranded to Bina Analytics
- [ ] Payment/trading features removed
- [ ] Compiles without errors
- [ ] Basic Supabase connection works

### Phase 2 Complete:
- [ ] Authentication via Supabase
- [ ] Research tasks can be created
- [ ] n8n workflows can be triggered
- [ ] Data sources simplified

### Phase 3 Complete:
- [ ] Research Tasks tab functional
- [ ] Intel State tab functional
- [ ] Client Reports tab functional
- [ ] Real-time updates working

### Phase 4 Complete:
- [ ] All features tested end-to-end
- [ ] Documentation complete
- [ ] Production build created
- [ ] Deployed and accessible

---

## Next Steps After Phase 4

### Future Enhancements (Beyond 8 weeks)
- Advanced analytics dashboards
- Custom report templates
- Multi-user collaboration
- Mobile app companion
- Advanced AI agent orchestration
- Integration with more data sources

---

**Document Version:** 1.0  
**Last Updated:** December 2024  
**Estimated Total Effort:** 120-160 hours (3-4 months part-time, or 2 months full-time)
