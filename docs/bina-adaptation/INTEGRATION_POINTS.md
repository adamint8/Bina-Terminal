# Integration Points Map
# Fincept Terminal → Bina Analytics Control Console

**Purpose:** Identify and document all integration points that need modification for Bina Analytics  
**Version:** 1.0  
**Date:** December 2024

---

## Table of Contents
1. [Backend API Integration](#backend-api-integration)
2. [AI Agent Integration](#ai-agent-integration)
3. [Data Sources Integration](#data-sources-integration)
4. [Workflow/Orchestration Integration](#workfloworchestration-integration)
5. [Database/Storage Integration](#databasestorage-integration)
6. [Authentication & Authorization](#authentication--authorization)

---

## Backend API Integration

### Current State

#### **API Base URL**
- **Location:** `vite.config.ts` (line 36-47)
- **Current Value:** `https://finceptbackend.share.zrok.io`
- **Proxy Config:**
```typescript
proxy: {
  '/api': {
    target: 'https://finceptbackend.share.zrok.io',
    changeOrigin: true,
    rewrite: (path) => path.replace(/^\/api/, ''),
  }
}
```

#### **Authentication API Service**
- **File:** `src/services/authApi.tsx` (370 lines)
- **Current Endpoints:**
  - `POST /api/v1/auth/login` - User login
  - `POST /api/v1/auth/register` - User registration
  - `POST /api/v1/auth/logout` - User logout
  - `POST /api/v1/auth/forgot-password` - Password reset
  - `POST /api/v1/auth/guest-register` - Guest access
  - `GET /api/v1/auth/profile` - User profile
  - `PUT /api/v1/auth/profile` - Update profile
  - `POST /api/v1/auth/verify-mfa` - MFA verification

#### **HTTP Client**
- **Primary:** Tauri HTTP Plugin (`@tauri-apps/plugin-http`)
- **Fallback:** Axios 1.12.2
- **Usage Pattern:**
```typescript
import { fetch } from '@tauri-apps/plugin-http';

const response = await fetch(`${API_BASE_URL}/endpoint`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-API-Key': apiKey
  },
  body: JSON.stringify(data)
});
```

---

### Replacement Strategy for Supabase

#### **1. Replace Authentication Flow**

**Current Location:** `src/services/authApi.tsx`

**Supabase Replacement:**
```typescript
// Install Supabase client
npm install @supabase/supabase-js

// New file: src/services/supabaseAuth.ts
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);

export const supabaseAuth = {
  async login(email: string, password: string) {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password
    });
    if (error) throw error;
    return data;
  },

  async register(email: string, password: string) {
    const { data, error } = await supabase.auth.signUp({
      email,
      password
    });
    if (error) throw error;
    return data;
  },

  async logout() {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;
  },

  async getSession() {
    const { data } = await supabase.auth.getSession();
    return data.session;
  },

  onAuthStateChange(callback) {
    return supabase.auth.onAuthStateChange(callback);
  }
};
```

**Files to Modify:**
1. `src/contexts/AuthContext.tsx` - Replace `AuthApiService` with `supabaseAuth`
2. `src/components/auth/LoginScreen.tsx` - Update login handler
3. `src/components/auth/RegisterScreen.tsx` - Update registration handler
4. `src/App.tsx` - Update authentication check logic

**Environment Variables:**
```bash
# Add to .env file
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

#### **2. Replace Data Fetching**

**Current Pattern:** Custom REST API calls via `/api` proxy

**Supabase Replacement:**
```typescript
// New file: src/services/supabaseClient.ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);

// Example: Fetch research tasks
export async function fetchResearchTasks() {
  const { data, error } = await supabase
    .from('research_tasks')
    .select('*')
    .order('created_at', { ascending: false });
  
  if (error) throw error;
  return data;
}

// Example: Subscribe to real-time updates
export function subscribeToTasks(callback) {
  return supabase
    .channel('research_tasks')
    .on('postgres_changes', 
      { event: '*', schema: 'public', table: 'research_tasks' },
      callback
    )
    .subscribe();
}
```

**Files to Modify:**
1. Create new service files for each data domain:
   - `src/services/supabaseResearch.ts` - Research tasks
   - `src/services/supabaseIntel.ts` - Intel states
   - `src/services/supabaseReports.ts` - Client reports
   - `src/services/supabaseData.ts` - Market/news data

#### **3. Remove Payment Integration**

**Files to DELETE:**
- `src/services/paymentApi.tsx` (630 lines)
- `src/components/payment/` (entire directory)
  - `InAppPaymentWindow.tsx`
  - `PaymentOverlay.tsx`
  - `PaymentProcessingScreen.tsx`
  - `PaymentSuccessScreen.tsx`
- `src/components/auth/PricingScreen.tsx`

**Files to MODIFY:**
- `src/App.tsx` - Remove pricing screen logic (lines 31, 145-170, 299-314)
- `src/contexts/AuthContext.tsx` - Remove subscription/payment types (lines 62-100)

---

## AI Agent Integration

### Current State

#### **LLM API Service**
- **File:** `src/services/llmApi.ts` (1,200+ lines)
- **Supported Providers:**
  - OpenAI (GPT-3.5, GPT-4)
  - Anthropic (Claude)
  - Google (Gemini)
  - Groq (Llama, Mixtral)
  - Ollama (local models)

**Interface:**
```typescript
interface LLMConfig {
  provider: string;
  apiKey?: string;
  baseUrl?: string;
  model: string;
  temperature: number;
  maxTokens: number;
  systemPrompt: string;
}

class LLMApiService {
  async chat(
    messages: ChatMessage[],
    config: LLMConfig,
    onStream?: StreamCallback
  ): Promise<LLMResponse>;
}
```

#### **Python AI Agents**
- **Location:** `src-tauri/resources/scripts/agents/`
- **Execution:** Rust command `execute_python_agent`
- **File:** `src-tauri/src/commands/ai_agents.rs`
- **Framework:** LangChain

**Current Agent Types:**
- Investor personas (Buffett, Dalio, Graham, etc.)
- Hedge fund strategies (Bridgewater, Citadel, Renaissance)
- Generic research agents

**Invocation Pattern:**
```typescript
import { invoke } from '@tauri-apps/api/core';

const result = await invoke('execute_python_agent', {
  agent_name: 'warren_buffett',
  query: 'Analyze Tesla stock',
  context: { /* data */ }
});
```

---

### Replacement Strategy for Bina AI Agents

#### **1. Create Custom AI Agent Service**

**New File:** `src/services/binaAgentService.ts`

```typescript
import { supabase } from './supabaseClient';

interface AgentRequest {
  agentType: 'research' | 'intel' | 'report' | 'analysis';
  task: string;
  context: any;
  entityId?: string; // Portfolio, Company, Sector
  priority: 'low' | 'medium' | 'high';
}

interface AgentTask {
  id: string;
  status: 'pending' | 'running' | 'completed' | 'failed';
  result?: any;
  error?: string;
  created_at: string;
  completed_at?: string;
}

export class BinaAgentService {
  // Submit task to Bina backend
  async submitTask(request: AgentRequest): Promise<string> {
    const { data, error } = await supabase
      .from('ai_tasks')
      .insert({
        agent_type: request.agentType,
        task_description: request.task,
        context: request.context,
        entity_id: request.entityId,
        priority: request.priority,
        status: 'pending'
      })
      .select()
      .single();
    
    if (error) throw error;
    
    // Trigger n8n workflow via webhook
    await this.triggerN8NWorkflow(data.id, request);
    
    return data.id;
  }

  // Trigger n8n workflow
  async triggerN8NWorkflow(taskId: string, request: AgentRequest) {
    const response = await fetch(
      import.meta.env.VITE_N8N_WEBHOOK_URL,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          taskId,
          agentType: request.agentType,
          task: request.task,
          context: request.context
        })
      }
    );
    
    if (!response.ok) {
      throw new Error('Failed to trigger workflow');
    }
  }

  // Poll task status
  async getTaskStatus(taskId: string): Promise<AgentTask> {
    const { data, error } = await supabase
      .from('ai_tasks')
      .select('*')
      .eq('id', taskId)
      .single();
    
    if (error) throw error;
    return data;
  }

  // Subscribe to task updates (real-time)
  subscribeToTask(taskId: string, callback: (task: AgentTask) => void) {
    return supabase
      .channel(`task:${taskId}`)
      .on('postgres_changes', {
        event: 'UPDATE',
        schema: 'public',
        table: 'ai_tasks',
        filter: `id=eq.${taskId}`
      }, (payload) => {
        callback(payload.new as AgentTask);
      })
      .subscribe();
  }

  // List all tasks
  async listTasks(filters?: {
    status?: string;
    agentType?: string;
    entityId?: string;
  }): Promise<AgentTask[]> {
    let query = supabase.from('ai_tasks').select('*');
    
    if (filters?.status) query = query.eq('status', filters.status);
    if (filters?.agentType) query = query.eq('agent_type', filters.agentType);
    if (filters?.entityId) query = query.eq('entity_id', filters.entityId);
    
    const { data, error } = await query.order('created_at', { ascending: false });
    
    if (error) throw error;
    return data;
  }
}

export const binaAgentService = new BinaAgentService();
```

#### **2. Modify ChatTab for Bina Agents**

**File:** `src/components/tabs/ChatTab.tsx` (lines 1-1000+)

**Changes:**
1. Replace investor persona selection with Bina agent types
2. Update agent invocation to use `binaAgentService`
3. Add task status monitoring UI
4. Display running tasks in sidebar

**New Agent Types for Bina:**
```typescript
const BINA_AGENTS = [
  {
    id: 'market_analyst',
    name: 'Market Intelligence',
    description: 'Analyzes market trends and data',
    icon: '📊'
  },
  {
    id: 'news_researcher',
    name: 'News Research',
    description: 'Monitors and analyzes news sources',
    icon: '📰'
  },
  {
    id: 'sentiment_analyst',
    name: 'Sentiment Analysis',
    description: 'Analyzes market sentiment',
    icon: '💭'
  },
  {
    id: 'report_writer',
    name: 'Report Generator',
    description: 'Creates client-facing reports',
    icon: '📄'
  },
  {
    id: 'intel_updater',
    name: 'Intel State Manager',
    description: 'Maintains entity intel states',
    icon: '🧠'
  }
];
```

#### **3. Remove Unused Agent Code**

**Files to MODIFY/DELETE:**
- `src-tauri/resources/scripts/agents/` - Keep structure, replace with Bina agents
- `src-tauri/src/commands/ai_agents.rs` - Simplify or remove if not using Python
- `src/services/agentLLMService.ts` - Adapt for Bina agents

---

## Data Sources Integration

### Current State

#### **Data Source Architecture**
- **Registry:** `src/services/dataSourceRegistry.ts`
- **Context:** `src/contexts/DataSourceContext.tsx`
- **UI:** `src/components/tabs/data-sources/DataSourcesTab.tsx`
- **Adapters:** `src/components/tabs/data-sources/adapters/` (100+ files)

**Adapter Pattern:**
```typescript
// Base adapter interface
export abstract class BaseAdapter {
  abstract testConnection(config: any): Promise<boolean>;
  abstract fetchData(config: any, query: any): Promise<any>;
  abstract getSchema(config: any): Promise<any>;
}

// Example: PostgreSQL adapter
export class PostgreSQLAdapter extends BaseAdapter {
  async testConnection(config) {
    // Test database connection
  }
  
  async fetchData(config, query) {
    // Execute SQL query
  }
}
```

**Configuration Storage:**
```typescript
// src/components/tabs/data-sources/dataSourceConfigs.ts
export const DATA_SOURCE_CONFIGS: DataSourceConfig[] = [
  {
    id: 'postgresql',
    name: 'PostgreSQL',
    type: 'postgresql',
    category: 'database',
    fields: [
      { name: 'host', type: 'text', required: true },
      { name: 'port', type: 'number', defaultValue: 5432 },
      // ...
    ]
  },
  // 100+ more configs
];
```

---

### Replacement Strategy for Bina Data Sources

#### **1. Prioritize Supabase as Primary Data Source**

**New File:** `src/services/dataSources/supabaseDataSource.ts`

```typescript
import { supabase } from '../supabaseClient';

export class SupabaseDataSource {
  async fetchMarketData(filters?: any) {
    const { data, error } = await supabase
      .from('market_data')
      .select('*')
      .order('timestamp', { ascending: false })
      .limit(1000);
    
    if (error) throw error;
    return data;
  }

  async fetchNewsData(filters?: any) {
    const { data, error } = await supabase
      .from('news_articles')
      .select('*')
      .order('published_at', { ascending: false })
      .limit(100);
    
    if (error) throw error;
    return data;
  }

  async fetchIntelState(entityId: string) {
    const { data, error } = await supabase
      .from('intel_states')
      .select('*')
      .eq('entity_id', entityId)
      .single();
    
    if (error) throw error;
    return data;
  }

  // Real-time subscriptions
  subscribeToMarketData(callback: (data: any) => void) {
    return supabase
      .channel('market_data')
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'market_data'
      }, (payload) => {
        callback(payload.new);
      })
      .subscribe();
  }
}

export const supabaseDataSource = new SupabaseDataSource();
```

#### **2. Keep Essential External Data Sources**

**Recommended to Keep:**
- **DBnomics** - Macro/economic data
- **Alpha Vantage** - Market data
- **Polygon.io** - Financial data
- **Custom API Mapper** - For proprietary APIs

**Recommended to Remove:**
- Trading-specific sources (Kraken, Binance, Fyers)
- Redundant database adapters (keep PostgreSQL for Supabase)
- Cloud storage adapters (unless needed)

**Files to DELETE:**
```
src/components/tabs/data-sources/adapters/
  ├── BinanceAdapter.ts (DELETE - crypto trading)
  ├── KrakenAdapter.ts (DELETE - crypto trading)
  ├── FyersAdapter.ts (DELETE - stock trading)
  ├── CoinbaseAdapter.ts (DELETE)
  ├── CoinGeckoAdapter.ts (DELETE)
  ├── [Keep: AlphaVantageAdapter.ts]
  ├── [Keep: PolygonAdapter.ts]
  ├── [Keep: PostgreSQLAdapter.ts]
  └── ...
```

#### **3. Create Bina-Specific Data Adapters**

**New File:** `src/services/dataSources/n8nDataSource.ts`

```typescript
// Fetch data from n8n workflows
export class N8NDataSource {
  private baseUrl = import.meta.env.VITE_N8N_API_URL;
  private apiKey = import.meta.env.VITE_N8N_API_KEY;

  async triggerWorkflow(workflowId: string, data: any) {
    const response = await fetch(`${this.baseUrl}/webhook/${workflowId}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-N8N-API-KEY': this.apiKey
      },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`n8n workflow failed: ${response.statusText}`);
    }
    
    return response.json();
  }

  async getWorkflowStatus(executionId: string) {
    const response = await fetch(`${this.baseUrl}/executions/${executionId}`, {
      headers: {
        'X-N8N-API-KEY': this.apiKey
      }
    });
    
    return response.json();
  }

  async listWorkflows() {
    const response = await fetch(`${this.baseUrl}/workflows`, {
      headers: {
        'X-N8N-API-KEY': this.apiKey
      }
    });
    
    return response.json();
  }
}

export const n8nDataSource = new N8NDataSource();
```

---

## Workflow/Orchestration Integration

### Current State

#### **Node Editor Architecture**
- **Main Tab:** `src/components/tabs/NodeEditorTab.tsx`
- **Node Types:** `src/components/tabs/node-editor/`
  - `DataSourceNode.tsx` - Fetch from data sources
  - `PythonAgentNode.tsx` - Execute Python agent
  - `MCPToolNode.tsx` - Use MCP tools
  - `TechnicalIndicatorNode.tsx` - Calculate indicators
  - `AgentMediatorNode.tsx` - Coordinate agents
  - `ResultsDisplayNode.tsx` - Show results
  - `BacktestNode.tsx` - Backtesting
  - `OptimizationNode.tsx` - Portfolio optimization

**Workflow Service:**
- **File:** `src/services/workflowService.ts` (300 lines)
- **Storage:** LocalStorage (`fincept_workflows` key)
- **Execution:** `src/components/tabs/node-editor/WorkflowExecutor.ts`
- **Orchestration:** `src/services/nodeExecutionManager.ts`

**ReactFlow Integration:**
```typescript
import ReactFlow, { Node, Edge } from 'reactflow';

const nodeTypes: NodeTypes = {
  'data-source': DataSourceNode,
  'python-agent': PythonAgentNode,
  'mcp-tool': MCPToolNode,
  // ...
};

<ReactFlow
  nodes={nodes}
  edges={edges}
  nodeTypes={nodeTypes}
  onConnect={onConnect}
/>
```

---

### Integration Strategy with n8n

#### **1. Hybrid Approach: Keep Node Editor + Add n8n Integration**

**Rationale:**
- Node Editor is useful for visual workflow creation in UI
- n8n handles production execution, scheduling, and monitoring
- Best of both worlds: UI/UX + Production robustness

**Architecture:**
```
Bina Control Console (Node Editor)
    ↓
  Design Workflow Visually
    ↓
  Export to n8n Format
    ↓
  POST /api/n8n/workflows (create in n8n)
    ↓
  n8n executes workflow
    ↓
  Real-time status via Supabase
    ↓
  Results displayed in Console
```

#### **2. Add n8n Integration Service**

**New File:** `src/services/n8nIntegration.ts`

```typescript
import { n8nDataSource } from './dataSources/n8nDataSource';
import { Node, Edge } from 'reactflow';

interface N8NWorkflowNode {
  name: string;
  type: string;
  parameters: any;
  position: [number, number];
}

interface N8NWorkflow {
  name: string;
  nodes: N8NWorkflowNode[];
  connections: any;
  settings: any;
}

export class N8NIntegrationService {
  // Convert ReactFlow workflow to n8n format
  convertToN8NFormat(
    nodes: Node[],
    edges: Edge[],
    workflowName: string
  ): N8NWorkflow {
    const n8nNodes: N8NWorkflowNode[] = nodes.map(node => ({
      name: node.data.label,
      type: this.mapNodeType(node.type),
      parameters: this.mapNodeParameters(node.data),
      position: [node.position.x, node.position.y]
    }));

    const connections = this.buildConnections(edges);

    return {
      name: workflowName,
      nodes: n8nNodes,
      connections,
      settings: {
        executionOrder: 'v1'
      }
    };
  }

  // Map Fincept node types to n8n node types
  private mapNodeType(nodeType: string): string {
    const typeMap = {
      'data-source': 'n8n-nodes-base.postgres', // or HTTP Request
      'python-agent': 'n8n-nodes-base.code', // or Function
      'mcp-tool': 'n8n-nodes-base.httpRequest',
      // Add more mappings
    };
    return typeMap[nodeType] || 'n8n-nodes-base.noOp';
  }

  private mapNodeParameters(data: any): any {
    // Convert node configuration to n8n parameters
    return {
      // ... parameter mapping logic
    };
  }

  private buildConnections(edges: Edge[]): any {
    // Build n8n connection structure
    const connections: any = {};
    
    edges.forEach(edge => {
      if (!connections[edge.source]) {
        connections[edge.source] = { main: [[]] };
      }
      connections[edge.source].main[0].push({
        node: edge.target,
        type: 'main',
        index: 0
      });
    });
    
    return connections;
  }

  // Create workflow in n8n
  async createWorkflow(workflow: N8NWorkflow): Promise<string> {
    const result = await n8nDataSource.triggerWorkflow(
      'create-workflow',
      workflow
    );
    return result.workflowId;
  }

  // Execute workflow in n8n
  async executeWorkflow(workflowId: string, data?: any): Promise<string> {
    const result = await n8nDataSource.triggerWorkflow(workflowId, data);
    return result.executionId;
  }

  // Monitor workflow execution
  async monitorExecution(executionId: string): Promise<any> {
    return n8nDataSource.getWorkflowStatus(executionId);
  }
}

export const n8nIntegration = new N8NIntegrationService();
```

#### **3. Add n8n Task Templates**

**New File:** `src/services/binaTaskTemplates.ts`

```typescript
// Pre-defined task templates that map to n8n workflows

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
  required: boolean;
  defaultValue?: any;
  options?: string[];
}

export const BINA_TASK_TEMPLATES: TaskTemplate[] = [
  {
    id: 'daily_market_intel',
    name: 'Daily Market Intelligence',
    description: 'Collect and analyze daily market data across all sources',
    n8nWorkflowId: 'wf_market_intel_001',
    category: 'intel',
    parameters: [
      {
        name: 'markets',
        type: 'select',
        required: true,
        options: ['US', 'EU', 'APAC', 'Global']
      },
      {
        name: 'depth',
        type: 'select',
        required: true,
        options: ['shallow', 'standard', 'deep']
      }
    ]
  },
  {
    id: 'company_research',
    name: 'Company Deep Dive Research',
    description: 'Comprehensive research report on a specific company',
    n8nWorkflowId: 'wf_company_research_001',
    category: 'research',
    parameters: [
      {
        name: 'company_id',
        type: 'entity',
        required: true
      },
      {
        name: 'focus_areas',
        type: 'select',
        required: false,
        options: ['financials', 'competitive', 'risks', 'growth', 'all']
      }
    ]
  },
  {
    id: 'sector_sentiment',
    name: 'Sector Sentiment Analysis',
    description: 'Analyze sentiment across news and social media for a sector',
    n8nWorkflowId: 'wf_sector_sentiment_001',
    category: 'intel',
    parameters: [
      {
        name: 'sector',
        type: 'entity',
        required: true
      },
      {
        name: 'timeframe',
        type: 'select',
        required: true,
        options: ['24h', '7d', '30d']
      }
    ]
  },
  {
    id: 'client_report',
    name: 'Client Investment Report',
    description: 'Generate client-facing investment advisory report',
    n8nWorkflowId: 'wf_client_report_001',
    category: 'report',
    parameters: [
      {
        name: 'portfolio_id',
        type: 'entity',
        required: true
      },
      {
        name: 'report_type',
        type: 'select',
        required: true,
        options: ['weekly', 'monthly', 'quarterly', 'custom']
      }
    ]
  }
];

export function getTemplateById(id: string): TaskTemplate | undefined {
  return BINA_TASK_TEMPLATES.find(t => t.id === id);
}

export function getTemplatesByCategory(category: string): TaskTemplate[] {
  return BINA_TASK_TEMPLATES.filter(t => t.category === category);
}
```

#### **4. Modify Node Editor to Support Export to n8n**

**File:** `src/components/tabs/NodeEditorTab.tsx`

**Add New Buttons:**
```typescript
// Add to toolbar
<button onClick={handleExportToN8N}>
  Export to n8n
</button>
<button onClick={handleDeployToN8N}>
  Deploy & Execute in n8n
</button>
```

**Add Export Handler:**
```typescript
const handleExportToN8N = async () => {
  try {
    const n8nWorkflow = n8nIntegration.convertToN8NFormat(
      nodes,
      edges,
      workflowName
    );
    
    // Download as JSON for manual import
    const blob = new Blob([JSON.stringify(n8nWorkflow, null, 2)], {
      type: 'application/json'
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${workflowName}_n8n.json`;
    a.click();
    
    toast.success('Workflow exported for n8n');
  } catch (error) {
    toast.error('Failed to export workflow');
  }
};

const handleDeployToN8N = async () => {
  try {
    const n8nWorkflow = n8nIntegration.convertToN8NFormat(
      nodes,
      edges,
      workflowName
    );
    
    const workflowId = await n8nIntegration.createWorkflow(n8nWorkflow);
    
    toast.success(`Workflow deployed to n8n: ${workflowId}`);
    
    // Optionally execute immediately
    const executionId = await n8nIntegration.executeWorkflow(workflowId);
    
    // Monitor execution
    monitorN8NExecution(executionId);
  } catch (error) {
    toast.error('Failed to deploy workflow');
  }
};
```

---

## Database/Storage Integration

### Current State

#### **Storage Layers**

1. **LocalStorage**
   - Workflows: `fincept_workflows`
   - User preferences
   - Cached data

2. **Tauri SQLite** (`@tauri-apps/plugin-sql`)
   - Chat history
   - Workflow results
   - Market data cache

**Usage Example:**
```typescript
import Database from '@tauri-apps/plugin-sql';

const db = await Database.load('sqlite:fincept.db');
await db.execute(`
  CREATE TABLE IF NOT EXISTS chats (
    id INTEGER PRIMARY KEY,
    message TEXT,
    timestamp INTEGER
  )
`);
```

3. **Backend API** (via zrok)
   - User profiles
   - Subscription data
   - Authentication

---

### Migration Path to Supabase

#### **1. Replace All Backend Queries with Supabase**

**Tables to Create in Supabase:**

```sql
-- Users (handled by Supabase Auth)

-- Intel States
CREATE TABLE intel_states (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  entity_id VARCHAR NOT NULL,
  entity_type VARCHAR NOT NULL, -- 'portfolio', 'company', 'sector'
  state JSONB NOT NULL, -- Current intel state
  last_updated TIMESTAMP DEFAULT NOW(),
  metadata JSONB
);

-- Research Tasks
CREATE TABLE research_tasks (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  agent_type VARCHAR NOT NULL,
  task_description TEXT NOT NULL,
  context JSONB,
  entity_id VARCHAR,
  priority VARCHAR DEFAULT 'medium',
  status VARCHAR DEFAULT 'pending', -- 'pending', 'running', 'completed', 'failed'
  result JSONB,
  error TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  created_by UUID REFERENCES auth.users(id)
);

-- Market Data
CREATE TABLE market_data (
  id BIGSERIAL PRIMARY KEY,
  symbol VARCHAR NOT NULL,
  timestamp TIMESTAMP NOT NULL,
  price NUMERIC,
  volume BIGINT,
  source VARCHAR,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- News Articles
CREATE TABLE news_articles (
  id BIGSERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT,
  source VARCHAR,
  url VARCHAR,
  published_at TIMESTAMP,
  sentiment NUMERIC, -- -1 to 1
  entities JSONB, -- Extracted entities
  created_at TIMESTAMP DEFAULT NOW()
);

-- Client Reports
CREATE TABLE client_reports (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  client_id UUID NOT NULL,
  report_type VARCHAR NOT NULL,
  portfolio_id VARCHAR,
  content JSONB,
  generated_at TIMESTAMP DEFAULT NOW(),
  delivered_at TIMESTAMP,
  status VARCHAR DEFAULT 'draft'
);

-- Workflows (for Node Editor persistence)
CREATE TABLE workflows (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR NOT NULL,
  description TEXT,
  nodes JSONB NOT NULL,
  edges JSONB NOT NULL,
  status VARCHAR DEFAULT 'draft',
  n8n_workflow_id VARCHAR, -- If deployed to n8n
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES auth.users(id)
);

-- Workflow Executions
CREATE TABLE workflow_executions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  workflow_id UUID REFERENCES workflows(id),
  n8n_execution_id VARCHAR,
  status VARCHAR DEFAULT 'running',
  started_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  result JSONB,
  error TEXT
);
```

#### **2. Create Supabase Service Layer**

**New File:** `src/services/supabase/index.ts`

```typescript
export * from './supabaseClient';
export * from './supabaseAuth';
export * from './supabaseResearch';
export * from './supabaseIntel';
export * from './supabaseReports';
export * from './supabaseWorkflows';
export * from './supabaseMarketData';
```

**Example Service:** `src/services/supabase/supabaseWorkflows.ts`

```typescript
import { supabase } from './supabaseClient';
import { Node, Edge } from 'reactflow';

export interface Workflow {
  id: string;
  name: string;
  description?: string;
  nodes: Node[];
  edges: Edge[];
  status: string;
  n8n_workflow_id?: string;
  created_at: string;
  updated_at: string;
}

export async function saveWorkflow(
  name: string,
  description: string,
  nodes: Node[],
  edges: Edge[]
): Promise<string> {
  const { data, error } = await supabase
    .from('workflows')
    .insert({
      name,
      description,
      nodes,
      edges,
      status: 'draft'
    })
    .select()
    .single();
  
  if (error) throw error;
  return data.id;
}

export async function loadWorkflow(id: string): Promise<Workflow> {
  const { data, error } = await supabase
    .from('workflows')
    .select('*')
    .eq('id', id)
    .single();
  
  if (error) throw error;
  return data;
}

export async function listWorkflows(): Promise<Workflow[]> {
  const { data, error } = await supabase
    .from('workflows')
    .select('*')
    .order('updated_at', { ascending: false });
  
  if (error) throw error;
  return data;
}

export async function deleteWorkflow(id: string): Promise<void> {
  const { error } = await supabase
    .from('workflows')
    .delete()
    .eq('id', id);
  
  if (error) throw error;
}
```

#### **3. Remove LocalStorage for Critical Data**

**Keep LocalStorage For:**
- UI preferences (theme, terminal colors)
- Temporary draft states

**Move to Supabase:**
- Workflows
- Execution history
- User-generated content

**Migration Code:**
```typescript
// Migrate existing workflows from localStorage to Supabase
async function migrateWorkflows() {
  const localWorkflows = localStorage.getItem('fincept_workflows');
  if (!localWorkflows) return;
  
  const workflows = JSON.parse(localWorkflows);
  
  for (const wf of workflows) {
    await saveWorkflow(wf.name, wf.description, wf.nodes, wf.edges);
  }
  
  // Optionally clear localStorage after migration
  localStorage.removeItem('fincept_workflows');
}
```

---

## Authentication & Authorization

### Current State

**Auth Flow:**
1. User logs in via `LoginScreen.tsx`
2. `AuthApiService.login()` calls backend
3. Backend returns API key
4. API key stored in `AuthContext` and localStorage
5. API key sent with all requests via `X-API-Key` header

**Session Management:**
- Stored in React Context (`AuthContext`)
- Persisted to localStorage
- No JWT/refresh tokens

---

### Supabase Auth Migration

**Replace with Supabase Auth:**
1. User logs in via Supabase
2. Supabase returns JWT access token
3. Token automatically handled by Supabase client
4. Token refresh handled automatically

**Benefits:**
- Built-in session management
- Automatic token refresh
- Row-level security (RLS) in database
- Magic links, OAuth, MFA support

**Files to Modify:**
1. `src/contexts/AuthContext.tsx` - Replace with Supabase auth
2. `src/components/auth/LoginScreen.tsx` - Use `supabase.auth.signInWithPassword()`
3. `src/components/auth/RegisterScreen.tsx` - Use `supabase.auth.signUp()`
4. All API services - Remove manual API key headers

---

## Summary of Integration Work

### High Priority
1. ✅ Replace backend API with Supabase (auth, data fetching)
2. ✅ Integrate n8n for workflow orchestration
3. ✅ Create Bina-specific AI agent service
4. ✅ Migrate storage from localStorage to Supabase

### Medium Priority
1. ✅ Simplify data source adapters (remove trading sources)
2. ✅ Add task template system
3. ✅ Build real-time monitoring UI

### Low Priority
1. ✅ Remove payment flow
2. ✅ Clean up unused services

---

**Next Steps:**
- See [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md) for UI/branding changes
- Follow [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) for phased implementation

---

**Document Version:** 1.0  
**Last Updated:** December 2024
