# Customization Guide
# Fincept Terminal → Bina Analytics Control Console

**Purpose:** Step-by-step instructions for customizing UI, branding, layout, and features  
**Version:** 1.0  
**Date:** December 2024

---

## Table of Contents
1. [Branding & Theming](#branding--theming)
2. [Layout Modifications](#layout-modifications)
3. [Component Replacement](#component-replacement)
4. [Feature Removal](#feature-removal)
5. [Adding New Features](#adding-new-features)

---

## Branding & Theming

### Change App Name and Title

#### **1. Update Tauri Configuration**

**File:** `src-tauri/tauri.conf.json`

**Changes:**
```json
{
  "productName": "BinaAnalytics",  // Line 3: Change from "FinceptTerminal"
  "identifier": "com.bina.analytics",  // Line 5: Change from "com.fincept.terminal"
  "app": {
    "windows": [
      {
        "title": "Bina Analytics Control Console",  // Line 16: Change window title
        // ... rest remains same
      }
    ]
  },
  "bundle": {
    "publisher": "Bina Analytics",  // Line 42: Change publisher
    "copyright": "Copyright © 2024 Bina Analytics. All rights reserved.",  // Line 43
    "category": "Productivity",  // Line 44: Change from "Finance"
    "shortDescription": "Business intelligence control console for continuous research",  // Line 45
    "longDescription": "Bina Analytics Control Console manages AI research tasks, intel states, and client deliverables for investment advisory."  // Line 46
  }
}
```

#### **2. Update Package.json**

**File:** `package.json`

```json
{
  "name": "bina-analytics-desktop",  // Line 2: Change from "fincept-terminal-desktop"
  "version": "1.0.0",  // Line 4: Reset version
  // ... rest of dependencies remain same
}
```

#### **3. Update HTML Title**

**File:** `index.html`

```html
<head>
  <title>Bina Analytics Control Console</title>
  <!-- Change from "Fincept Terminal" -->
</head>
```

#### **4. Update Version Constant**

**File:** `src/constants/version.ts`

```typescript
export const APP_VERSION = '1.0.0';  // Reset from '3.0.11'
export const APP_NAME = 'Bina Analytics Control Console';  // Add new constant
export const APP_SHORT_NAME = 'Bina Analytics';  // Add new constant
```

#### **5. Update Window Title References**

**Files to modify:**
- `src/components/dashboard/DashboardScreen.tsx` (Line ~200)
- `src/App.tsx` (Loading screen text, Line 208)

**Before:**
```typescript
<p className="text-zinc-400 text-sm">Initializing Fincept Terminal...</p>
```

**After:**
```typescript
<p className="text-zinc-400 text-sm">Initializing Bina Analytics...</p>
```

---

### Change Logo and Icons

#### **1. Replace App Icons**

**Location:** `src-tauri/icons/`

**Files to replace:**
- `icon.ico` (Windows icon)
- `icon.icns` (macOS icon)
- `32x32.png`, `64x64.png`, `128x128.png`, `128x128@2x.png` (Various sizes)

**Tool Recommendations:**
- Use [Tauri Icon Generator](https://github.com/tauri-apps/tauri-icon)
- Or manually create icons:
  ```bash
  # Install tauri-cli icon generator
  cargo install tauri-cli
  
  # Generate icons from a single 1024x1024 PNG
  cd src-tauri
  cargo tauri icon path/to/bina-logo.png
  ```

#### **2. Replace Logo in UI**

**Option A: Use Text Logo**

**File:** `src/components/common/Header.tsx`

**Find the logo section** (around line 30-50):
```tsx
// Current (likely has Fincept logo or text)
<div className="flex items-center gap-2">
  <span className="text-xl font-bold">Fincept Terminal</span>
</div>
```

**Replace with:**
```tsx
<div className="flex items-center gap-2">
  <span className="text-xl font-bold bg-gradient-to-r from-blue-400 to-blue-600 bg-clip-text text-transparent">
    Bina Analytics
  </span>
  <span className="text-xs text-zinc-500">Control Console</span>
</div>
```

**Option B: Use Image Logo**

1. Add logo file to `public/` directory:
   - `public/bina-logo.svg` or `public/bina-logo.png`

2. Update Header component:
```tsx
<div className="flex items-center gap-2">
  <img src="/bina-logo.svg" alt="Bina Analytics" className="h-8 w-8" />
  <span className="text-xl font-bold">Bina Analytics</span>
</div>
```

#### **3. Update Favicon**

**File:** `public/favicon.ico`

Replace with Bina logo favicon (16x16, 32x32, 48x48 sizes)

**File:** `index.html`

```html
<head>
  <link rel="icon" type="image/png" href="/bina-favicon.png" />
</head>
```

---

### Modify Color Scheme

#### **1. Update Tailwind Theme Colors**

**File:** `tailwind.config.js` (may need to create if using inline config)

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        // Bina Analytics Brand Colors
        'bina': {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',  // Primary accent
          500: '#3b82f6',
          600: '#2563eb',  // Primary brand
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        },
        // Keep existing terminal colors or customize
        'terminal': {
          background: '#0a0a0a',
          foreground: '#e0e0e0',
          accent: '#60a5fa',  // Match bina-400
        }
      }
    }
  }
}
```

#### **2. Update Theme Context**

**File:** `src/contexts/ThemeContext.tsx`

**Find the theme definitions** (around line 20-60):

```typescript
const BINA_THEME = {
  background: '#0a0a0a',
  foreground: '#e0e0e0',
  accent: '#60a5fa',  // Bina blue
  success: '#10b981',
  warning: '#f59e0b',
  error: '#ef4444',
  muted: '#6b7280',
  border: '#1f2937',
};

export const ThemeContext = createContext({
  theme: 'dark',
  terminalTheme: BINA_THEME,
  // ...
});
```

#### **3. Update CSS Variables**

**File:** `src/App.css`

**Find or add CSS custom properties:**

```css
:root {
  --color-bina-primary: #3b82f6;
  --color-bina-accent: #60a5fa;
  --color-background: #0a0a0a;
  --color-foreground: #e0e0e0;
  --color-border: #1f2937;
  --color-muted: #6b7280;
}

/* Update any hard-coded Fincept brand colors */
.accent-color {
  color: var(--color-bina-accent);
}

.bg-primary {
  background-color: var(--color-bina-primary);
}

.border-accent {
  border-color: var(--color-bina-accent);
}
```

#### **4. Update Component Accent Colors**

**Search and replace across components:**

**Find files with hard-coded Fincept brand colors:**
```bash
cd src
grep -r "#[Cc]06524" .  # Fincept orange
grep -r "#[Ff][Ff][Cc]131" .  # Fincept yellow
```

**Replace with Bina colors:**
- Fincept Orange `#C06524` → Bina Blue `#3b82f6`
- Fincept Yellow `#FFC131` → Bina Light Blue `#60a5fa`

**Files likely to contain brand colors:**
- `src/components/auth/*.tsx` (auth screens)
- `src/components/common/Header.tsx`
- `src/components/common/Footer.tsx`
- `src/components/dashboard/DashboardScreen.tsx`

---

### Update Metadata and Credits

#### **1. Update Footer**

**File:** `src/components/common/Footer.tsx`

**Find copyright section:**

```tsx
// Before
<p className="text-xs text-zinc-500">
  © 2024-2025 Fincept Corporation. All rights reserved.
</p>

// After
<p className="text-xs text-zinc-500">
  © 2024 Bina Analytics. All rights reserved.
</p>
```

**Update links:**
```tsx
// Remove or update social/external links
<div className="flex gap-4">
  <a href="https://bina-analytics.com" target="_blank">Website</a>
  <a href="mailto:support@bina-analytics.com">Support</a>
  {/* Remove Twitter, GitHub, Discord links if not applicable */}
</div>
```

#### **2. Update README**

**File:** `README.md`

Replace entire content with Bina-specific README (see [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) Phase 1)

---

## Layout Modifications

### Add/Remove Sidebar Items

#### **1. Locate Tab Configuration**

**File:** `src/components/dashboard/DashboardScreen.tsx`

**Find the Tabs component** (around line 200-400):

```tsx
<Tabs value={activeTab} onValueChange={setActiveTab}>
  <TabsList className="...">
    <TabsTrigger value="dashboard">Dashboard</TabsTrigger>
    <TabsTrigger value="markets">Markets</TabsTrigger>
    <TabsTrigger value="news">News</TabsTrigger>
    {/* ... 30+ tabs */}
  </TabsList>

  <TabsContent value="dashboard">
    <DashboardTab />
  </TabsContent>
  {/* ... */}
</Tabs>
```

#### **2. Remove Trading-Related Tabs**

**Tabs to DELETE:**
- `trading` - Remove `<TabsTrigger value="trading">Trading</TabsTrigger>`
- `fyers` - Remove Fyers integration tab
- `kraken` - Remove Kraken crypto tab
- `portfolio` - Remove (or repurpose for client portfolios)
- `backtesting` - Remove (unless relevant to Bina)

**Example:**
```tsx
// BEFORE: 30+ tabs including trading
<TabsTrigger value="trading">Trading</TabsTrigger>
<TabsTrigger value="portfolio">Portfolio</TabsTrigger>
<TabsTrigger value="backtesting">Backtesting</TabsTrigger>

// AFTER: Remove these lines entirely
// (Trading features not needed for Bina)
```

#### **3. Add Bina-Specific Tabs**

**New tabs to ADD:**

```tsx
{/* Research & Intelligence */}
<TabsTrigger value="research-tasks">
  <Rocket className="w-4 h-4 mr-2" />
  Research Tasks
</TabsTrigger>

<TabsTrigger value="intel-state">
  <Brain className="w-4 h-4 mr-2" />
  Intel State
</TabsTrigger>

{/* Client Deliverables */}
<TabsTrigger value="reports">
  <FileText className="w-4 h-4 mr-2" />
  Client Reports
</TabsTrigger>

{/* Operations */}
<TabsTrigger value="task-monitor">
  <Activity className="w-4 h-4 mr-2" />
  Task Monitor
</TabsTrigger>

<TabsTrigger value="data-pipelines">
  <Database className="w-4 h-4 mr-2" />
  Data Pipelines
</TabsTrigger>
```

**Import icons:**
```typescript
import { Rocket, Brain, FileText, Activity, Database } from 'lucide-react';
```

#### **4. Organize Tabs into Categories**

**Create tabbed sections** using separators or grouping:

```tsx
<TabsList className="...">
  {/* Core */}
  <div className="flex gap-2 px-2 border-r border-zinc-700">
    <TabsTrigger value="dashboard">Dashboard</TabsTrigger>
    <TabsTrigger value="research-tasks">Research Tasks</TabsTrigger>
  </div>

  {/* Intelligence */}
  <div className="flex gap-2 px-2 border-r border-zinc-700">
    <TabsTrigger value="intel-state">Intel State</TabsTrigger>
    <TabsTrigger value="news">News</TabsTrigger>
    <TabsTrigger value="markets">Markets</TabsTrigger>
  </div>

  {/* Operations */}
  <div className="flex gap-2 px-2 border-r border-zinc-700">
    <TabsTrigger value="task-monitor">Task Monitor</TabsTrigger>
    <TabsTrigger value="node-editor">Workflows</TabsTrigger>
    <TabsTrigger value="data-pipelines">Data Pipelines</TabsTrigger>
  </div>

  {/* Reports & AI */}
  <div className="flex gap-2 px-2">
    <TabsTrigger value="reports">Client Reports</TabsTrigger>
    <TabsTrigger value="chat">AI Assistant</TabsTrigger>
  </div>
</TabsList>
```

---

### Create New Pages/Routes

#### **Example: Create "Research Tasks" Tab**

**1. Create Component File**

**New File:** `src/components/tabs/ResearchTasksTab.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { binaAgentService, AgentTask } from '@/services/binaAgentService';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Rocket, Play, Pause, CheckCircle, XCircle, Clock } from 'lucide-react';

const ResearchTasksTab: React.FC = () => {
  const [tasks, setTasks] = useState<AgentTask[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadTasks();
    
    // Subscribe to real-time updates
    const subscription = binaAgentService.subscribeToTasks((newTask) => {
      setTasks(prev => {
        const index = prev.findIndex(t => t.id === newTask.id);
        if (index >= 0) {
          const updated = [...prev];
          updated[index] = newTask;
          return updated;
        }
        return [newTask, ...prev];
      });
    });

    return () => subscription.unsubscribe();
  }, []);

  const loadTasks = async () => {
    try {
      const data = await binaAgentService.listTasks();
      setTasks(data);
    } catch (error) {
      console.error('Failed to load tasks:', error);
    } finally {
      setLoading(false);
    }
  };

  const getStatusIcon = (status: string) => {
    switch (status) {
      case 'completed': return <CheckCircle className="w-4 h-4 text-green-500" />;
      case 'failed': return <XCircle className="w-4 h-4 text-red-500" />;
      case 'running': return <Play className="w-4 h-4 text-blue-500 animate-pulse" />;
      default: return <Clock className="w-4 h-4 text-yellow-500" />;
    }
  };

  const getStatusColor = (status: string) => {
    switch (status) {
      case 'completed': return 'bg-green-500/10 text-green-500';
      case 'failed': return 'bg-red-500/10 text-red-500';
      case 'running': return 'bg-blue-500/10 text-blue-500';
      default: return 'bg-yellow-500/10 text-yellow-500';
    }
  };

  if (loading) {
    return (
      <div className="flex items-center justify-center h-full">
        <div className="animate-spin rounded-full h-8 w-8 border-2 border-bina-400 border-t-transparent" />
      </div>
    );
  }

  return (
    <div className="p-6 space-y-6">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-3">
          <Rocket className="w-6 h-6 text-bina-400" />
          <div>
            <h1 className="text-2xl font-bold text-white">Research Tasks</h1>
            <p className="text-sm text-zinc-400">Monitor and manage AI research tasks</p>
          </div>
        </div>
        <Button onClick={() => {/* Open create task modal */}}>
          <Rocket className="w-4 h-4 mr-2" />
          New Task
        </Button>
      </div>

      {/* Stats */}
      <div className="grid grid-cols-4 gap-4">
        {['pending', 'running', 'completed', 'failed'].map(status => {
          const count = tasks.filter(t => t.status === status).length;
          return (
            <Card key={status} className="p-4 bg-zinc-900 border-zinc-800">
              <div className="flex items-center justify-between">
                <div>
                  <p className="text-sm text-zinc-400 capitalize">{status}</p>
                  <p className="text-2xl font-bold text-white">{count}</p>
                </div>
                {getStatusIcon(status)}
              </div>
            </Card>
          );
        })}
      </div>

      {/* Task List */}
      <div className="space-y-3">
        {tasks.map(task => (
          <Card key={task.id} className="p-4 bg-zinc-900 border-zinc-800 hover:border-bina-400 transition-colors">
            <div className="flex items-start justify-between">
              <div className="flex-1">
                <div className="flex items-center gap-2 mb-2">
                  {getStatusIcon(task.status)}
                  <Badge className={getStatusColor(task.status)}>
                    {task.status}
                  </Badge>
                  <Badge variant="outline">{task.agentType}</Badge>
                </div>
                <h3 className="text-lg font-semibold text-white mb-1">
                  {task.task_description}
                </h3>
                <p className="text-sm text-zinc-400">
                  Created: {new Date(task.created_at).toLocaleString()}
                </p>
              </div>
              <Button variant="ghost" size="sm">
                View Details
              </Button>
            </div>
          </Card>
        ))}
      </div>
    </div>
  );
};

export default ResearchTasksTab;
```

**2. Import in DashboardScreen**

**File:** `src/components/dashboard/DashboardScreen.tsx`

**Add import:**
```typescript
import ResearchTasksTab from '@/components/tabs/ResearchTasksTab';
```

**Add tab content:**
```tsx
<TabsContent value="research-tasks">
  <ResearchTasksTab />
</TabsContent>
```

---

### Modify Dashboard Layout

#### **1. Customize Dashboard Widgets**

**File:** `src/components/tabs/DashboardTab.tsx`

**Current structure** uses `react-grid-layout` for draggable widgets.

**To modify:**

**Remove financial widgets:**
```typescript
// Remove these widgets:
// - ForexWidget
// - CryptoWidget  
// - CommoditiesWidget
// - WatchlistWidget (unless repurposed)
```

**Add Bina widgets:**

**New File:** `src/components/tabs/dashboard/widgets/ActiveTasksWidget.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { BaseWidget } from './BaseWidget';
import { binaAgentService } from '@/services/binaAgentService';
import { Activity } from 'lucide-react';

export const ActiveTasksWidget: React.FC<{ onRemove: () => void }> = ({ onRemove }) => {
  const [tasks, setTasks] = useState([]);

  useEffect(() => {
    loadActiveTasks();
  }, []);

  const loadActiveTasks = async () => {
    const data = await binaAgentService.listTasks({ status: 'running' });
    setTasks(data);
  };

  return (
    <BaseWidget
      title="Active Research Tasks"
      icon={<Activity className="w-4 h-4" />}
      onRemove={onRemove}
    >
      <div className="space-y-2">
        {tasks.length === 0 ? (
          <p className="text-sm text-zinc-500">No active tasks</p>
        ) : (
          tasks.map(task => (
            <div key={task.id} className="flex items-center justify-between p-2 bg-zinc-800 rounded">
              <span className="text-sm text-white">{task.task_description}</span>
              <div className="w-2 h-2 bg-blue-500 rounded-full animate-pulse" />
            </div>
          ))
        )}
      </div>
    </BaseWidget>
  );
};
```

**Update DashboardTab to include new widget:**

```typescript
import { ActiveTasksWidget } from './dashboard/widgets/ActiveTasksWidget';
import { IntelStateWidget } from './dashboard/widgets/IntelStateWidget';
import { RecentReportsWidget } from './dashboard/widgets/RecentReportsWidget';

// In the widget grid configuration:
const defaultLayout = [
  { i: 'active-tasks', x: 0, y: 0, w: 6, h: 4 },
  { i: 'intel-state', x: 6, y: 0, w: 6, h: 4 },
  { i: 'recent-reports', x: 0, y: 4, w: 12, h: 4 },
  // ... more widgets
];
```

---

## Component Replacement

### Replace Chart Components

#### **Current:** TradingView-style candlestick charts for trading

#### **Replace with:** Business intelligence dashboards

**New File:** `src/components/charts/TaskStatusChart.tsx`

```typescript
import React from 'react';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

interface TaskStatusChartProps {
  data: {
    date: string;
    completed: number;
    failed: number;
    pending: number;
  }[];
}

export const TaskStatusChart: React.FC<TaskStatusChartProps> = ({ data }) => {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <BarChart data={data}>
        <CartesianGrid strokeDasharray="3 3" stroke="#333" />
        <XAxis dataKey="date" stroke="#999" />
        <YAxis stroke="#999" />
        <Tooltip
          contentStyle={{ backgroundColor: '#1a1a1a', border: '1px solid #333' }}
          labelStyle={{ color: '#fff' }}
        />
        <Bar dataKey="completed" fill="#10b981" name="Completed" />
        <Bar dataKey="failed" fill="#ef4444" name="Failed" />
        <Bar dataKey="pending" fill="#f59e0b" name="Pending" />
      </BarChart>
    </ResponsiveContainer>
  );
};
```

**Usage:**
```typescript
import { TaskStatusChart } from '@/components/charts/TaskStatusChart';

<TaskStatusChart data={taskStats} />
```

---

### Swap Out AI Chat Component

#### **Current:** ChatTab with investor personas

#### **Adapt for:** Bina research assistant

**File:** `src/components/tabs/ChatTab.tsx`

**Key changes:**

1. **Replace agent selection** (lines ~100-200):

```typescript
// BEFORE: Investor personas
const AGENTS = [
  { id: 'warren_buffett', name: 'Warren Buffett', ... },
  { id: 'ray_dalio', name: 'Ray Dalio', ... },
  // ...
];

// AFTER: Bina agents
const BINA_AGENTS = [
  { 
    id: 'research_assistant', 
    name: 'Research Assistant',
    description: 'General research and analysis',
    systemPrompt: 'You are a research analyst for Bina Analytics...'
  },
  { 
    id: 'intel_analyst', 
    name: 'Intel Analyst',
    description: 'Updates intel states with new data',
    systemPrompt: 'You maintain intel states for entities...'
  },
  {
    id: 'report_writer',
    name: 'Report Writer',
    description: 'Generates client-facing reports',
    systemPrompt: 'You write professional investment reports...'
  }
];
```

2. **Add context from intel states:**

```typescript
const enhanceMessageWithContext = async (message: string) => {
  // Fetch relevant intel state
  const entities = extractEntities(message); // Extract company/sector mentions
  
  const intelData = await Promise.all(
    entities.map(e => supabaseDataSource.fetchIntelState(e))
  );
  
  return {
    message,
    context: intelData
  };
};
```

3. **Add task creation button:**

```tsx
<Button onClick={handleCreateTaskFromChat}>
  <Rocket className="w-4 h-4 mr-2" />
  Create Research Task
</Button>
```

---

### Modify Node Editor

#### **Keep** ReactFlow-based workflow editor (it's excellent!)

#### **Customize** node types for Bina:

**1. Remove trading nodes:**
- `BacktestNode.tsx` - DELETE
- `OptimizationNode.tsx` - DELETE (or repurpose)

**2. Add Bina nodes:**

**New File:** `src/components/tabs/node-editor/IntelStateNode.tsx`

```typescript
import React from 'react';
import { Handle, Position } from 'reactflow';
import { Brain } from 'lucide-react';

export const IntelStateNode = ({ data, id }: any) => {
  return (
    <div className="bg-zinc-900 border-2 border-blue-500 rounded-lg p-4 min-w-[200px]">
      <Handle type="target" position={Position.Left} className="w-3 h-3 bg-blue-500" />
      
      <div className="flex items-center gap-2 mb-3">
        <Brain className="w-4 h-4 text-blue-400" />
        <span className="font-semibold text-white">Intel State</span>
      </div>
      
      <div className="space-y-2">
        <div>
          <label className="text-xs text-zinc-400">Entity ID</label>
          <input
            type="text"
            value={data.entityId || ''}
            onChange={(e) => data.onChange(id, 'entityId', e.target.value)}
            className="w-full px-2 py-1 bg-zinc-800 text-white text-sm rounded"
            placeholder="AAPL"
          />
        </div>
        
        <div>
          <label className="text-xs text-zinc-400">Action</label>
          <select
            value={data.action || 'read'}
            onChange={(e) => data.onChange(id, 'action', e.target.value)}
            className="w-full px-2 py-1 bg-zinc-800 text-white text-sm rounded"
          >
            <option value="read">Read State</option>
            <option value="update">Update State</option>
            <option value="merge">Merge Data</option>
          </select>
        </div>
      </div>
      
      <Handle type="source" position={Position.Right} className="w-3 h-3 bg-blue-500" />
    </div>
  );
};
```

**3. Register new node types:**

**File:** `src/components/tabs/NodeEditorTab.tsx`

```typescript
import { IntelStateNode } from './node-editor/IntelStateNode';
import { ReportGeneratorNode } from './node-editor/ReportGeneratorNode';
import { N8NTriggerNode } from './node-editor/N8NTriggerNode';

const nodeTypes: NodeTypes = {
  'data-source': DataSourceNode,
  'python-agent': PythonAgentNode,
  'mcp-tool': MCPToolNode,
  'intel-state': IntelStateNode,  // NEW
  'report-generator': ReportGeneratorNode,  // NEW
  'n8n-trigger': N8NTriggerNode,  // NEW
  'results-display': ResultsDisplayNode,
};
```

---

## Feature Removal

### Safe Removal Process

#### **Step 1: Identify Dependencies**

**Use search to find all references:**

```bash
cd src
grep -r "TradingTab" .  # Find all imports/references
grep -r "FyersTab" .
grep -r "BacktestingTab" .
```

#### **Step 2: Remove Import Statements**

**File:** `src/components/dashboard/DashboardScreen.tsx`

```typescript
// DELETE these imports:
import { TradingTab } from '@/components/tabs/TradingTab';
import FyersTab from '@/components/tabs/fyers';
import KrakenTab from '@/components/tabs/KrakenTab';
import BacktestingTab from '@/components/tabs/BacktestingTab';
import PortfolioTab from '@/components/tabs/PortfolioTab';
```

#### **Step 3: Remove Tab Triggers and Content**

```tsx
{/* DELETE these TabsTrigger elements */}
<TabsTrigger value="trading">Trading</TabsTrigger>
<TabsTrigger value="fyers">Fyers</TabsTrigger>
<TabsTrigger value="kraken">Kraken</TabsTrigger>
<TabsTrigger value="backtesting">Backtesting</TabsTrigger>
<TabsTrigger value="portfolio">Portfolio</TabsTrigger>

{/* DELETE these TabsContent elements */}
<TabsContent value="trading"><TradingTab /></TabsContent>
<TabsContent value="fyers"><FyersTab /></TabsContent>
<TabsContent value="kraken"><KrakenTab /></TabsContent>
<TabsContent value="backtesting"><BacktestingTab /></TabsContent>
<TabsContent value="portfolio"><PortfolioTab /></TabsContent>
```

#### **Step 4: Remove Service Files**

**DELETE these files/directories:**

```
src/components/tabs/TradingTab.tsx
src/components/tabs/fyers/ (entire directory)
src/components/tabs/KrakenTab.tsx
src/components/tabs/BacktestingTab.tsx
src/components/tabs/PortfolioTab.tsx
src/components/tabs/trading/ (entire directory)
src/components/tabs/portfolio/ (entire directory)
src/services/fyersService.ts
src/services/fyersAuth.ts
src/services/fyersWebSocket.ts
src/services/hyperliquid/ (entire directory)
src/services/backtesting/ (entire directory, unless reusable)
src/stockBrokers/ (entire directory)
src/hooks/usePaperTrading.ts
```

#### **Step 5: Remove Dependencies**

**File:** `package.json`

**Dependencies to REMOVE:**
```json
{
  "dependencies": {
    "fyers-web-sdk-v3": "^1.3.1",  // DELETE - Fyers trading
    // Keep all others (React, Tauri, UI libraries, etc.)
  }
}
```

**Run cleanup:**
```bash
npm install  # Updates package-lock.json
```

#### **Step 6: Remove Rust Commands (if applicable)**

**File:** `src-tauri/src/lib.rs`

**Remove trading-related command registrations** (if any):

```rust
// DELETE any trading-specific commands
// Keep: ai_agents, analytics, market_data, etc.
```

#### **Step 7: Test Compilation**

```bash
npm run build
```

**Fix any TypeScript errors** that arise from removed dependencies.

---

### Clean Up Unused Dependencies

**Run dependency analyzer:**

```bash
npx depcheck
```

**Review and remove unused packages** (beyond trading):

**Likely candidates:**
- `react-confetti` (used in payment success screen - remove if deleting payment flow)
- `otpauth`, `otplib` (MFA - keep if using Supabase MFA, remove if not)

---

### Remove Unused Routes/Screens

**Files to DELETE:**

```
src/components/auth/PricingScreen.tsx
src/components/payment/ (entire directory)
src/components/info/ContactUsScreen.tsx (if not needed)
src/components/info/TermsOfServiceScreen.tsx (replace with Bina ToS)
src/components/info/PrivacyPolicyScreen.tsx (replace with Bina privacy policy)
```

**File:** `src/App.tsx`

**Remove pricing screen logic:**

```typescript
// DELETE these lines:
export type Screen =
  | 'login'
  | 'register'
  | 'forgotPassword'
  | 'help'
  | 'contactUs'
  | 'termsOfService'
  | 'trademarks'
  | 'privacyPolicy'
  | 'pricing'  // DELETE
  | 'paymentProcessing'  // DELETE
  | 'paymentSuccess'  // DELETE
  | 'dashboard';
```

**Remove pricing navigation logic** (lines 145-170, 299-314 in current App.tsx)

---

## Adding New Features

### Example: Add Task Monitor Tab

**1. Create Component**

**New File:** `src/components/tabs/TaskMonitorTab.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { Activity, Play, Pause, Square, RefreshCw } from 'lucide-react';
import { binaAgentService } from '@/services/binaAgentService';
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Progress } from '@/components/ui/progress';

const TaskMonitorTab: React.FC = () => {
  const [tasks, setTasks] = useState([]);
  const [autoRefresh, setAutoRefresh] = useState(true);

  useEffect(() => {
    loadTasks();
    
    if (autoRefresh) {
      const interval = setInterval(loadTasks, 5000);
      return () => clearInterval(interval);
    }
  }, [autoRefresh]);

  const loadTasks = async () => {
    try {
      const runningTasks = await binaAgentService.listTasks({ status: 'running' });
      setTasks(runningTasks);
    } catch (error) {
      console.error('Failed to load tasks:', error);
    }
  };

  return (
    <div className="p-6 space-y-6">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-3">
          <Activity className="w-6 h-6 text-bina-400" />
          <div>
            <h1 className="text-2xl font-bold text-white">Task Monitor</h1>
            <p className="text-sm text-zinc-400">Real-time task execution monitoring</p>
          </div>
        </div>
        <div className="flex items-center gap-2">
          <Button
            variant={autoRefresh ? 'default' : 'outline'}
            size="sm"
            onClick={() => setAutoRefresh(!autoRefresh)}
          >
            {autoRefresh ? <Pause className="w-4 h-4 mr-2" /> : <Play className="w-4 h-4 mr-2" />}
            {autoRefresh ? 'Pause' : 'Resume'}
          </Button>
          <Button variant="outline" size="sm" onClick={loadTasks}>
            <RefreshCw className="w-4 h-4" />
          </Button>
        </div>
      </div>

      {/* Task List */}
      <div className="space-y-4">
        {tasks.length === 0 ? (
          <Card className="p-12 bg-zinc-900 border-zinc-800 text-center">
            <Activity className="w-12 h-12 text-zinc-600 mx-auto mb-4" />
            <p className="text-zinc-400">No tasks currently running</p>
          </Card>
        ) : (
          tasks.map(task => (
            <Card key={task.id} className="p-6 bg-zinc-900 border-zinc-800">
              <div className="flex items-start justify-between mb-4">
                <div className="flex-1">
                  <div className="flex items-center gap-2 mb-2">
                    <div className="w-3 h-3 bg-blue-500 rounded-full animate-pulse" />
                    <Badge variant="outline">{task.agentType}</Badge>
                    <span className="text-xs text-zinc-500">Started {new Date(task.created_at).toLocaleTimeString()}</span>
                  </div>
                  <h3 className="text-lg font-semibold text-white mb-1">
                    {task.task_description}
                  </h3>
                </div>
                <Button variant="ghost" size="sm">
                  <Square className="w-4 h-4 mr-2" />
                  Stop
                </Button>
              </div>
              
              {/* Progress (if available) */}
              {task.progress !== undefined && (
                <div className="space-y-1">
                  <Progress value={task.progress} className="h-2" />
                  <p className="text-xs text-zinc-500">{task.progress}% complete</p>
                </div>
              )}
              
              {/* Logs preview */}
              {task.logs && (
                <div className="mt-4 p-3 bg-black rounded font-mono text-xs text-zinc-400 max-h-32 overflow-y-auto">
                  {task.logs.slice(-5).map((log, i) => (
                    <div key={i}>{log}</div>
                  ))}
                </div>
              )}
            </Card>
          ))
        )}
      </div>
    </div>
  );
};

export default TaskMonitorTab;
```

**2. Register in DashboardScreen**

```typescript
import TaskMonitorTab from '@/components/tabs/TaskMonitorTab';

// Add tab trigger
<TabsTrigger value="task-monitor">
  <Activity className="w-4 h-4 mr-2" />
  Task Monitor
</TabsTrigger>

// Add tab content
<TabsContent value="task-monitor">
  <TaskMonitorTab />
</TabsContent>
```

---

## Summary Checklist

### Branding
- [ ] Update app name in `tauri.conf.json`
- [ ] Replace icons in `src-tauri/icons/`
- [ ] Update logo in Header component
- [ ] Change color scheme in Tailwind config
- [ ] Update footer copyright

### Layout
- [ ] Remove trading tabs
- [ ] Add Bina-specific tabs
- [ ] Organize tabs into logical groups
- [ ] Customize dashboard widgets

### Components
- [ ] Adapt ChatTab for Bina agents
- [ ] Add Bina node types to Node Editor
- [ ] Create task monitoring UI
- [ ] Build client report components

### Features
- [ ] Remove payment flow
- [ ] Remove trading features
- [ ] Remove unused dependencies
- [ ] Clean up unused routes

---

**Next Steps:**
- Review [INTEGRATION_POINTS.md](./INTEGRATION_POINTS.md) for backend integration
- Follow [BINA_ADAPTATION_ROADMAP.md](./BINA_ADAPTATION_ROADMAP.md) for phased approach

---

**Document Version:** 1.0  
**Last Updated:** December 2024
