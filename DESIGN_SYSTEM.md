# Fincept Terminal - Design System Documentation

## Overview
This document provides a comprehensive analysis of all design-related elements in the Fincept Terminal codebase, including UI frameworks, styling approaches, design tokens, component structures, and customization points.

---

## 1. UI Framework & Libraries

### Core Framework
- **React 19.1.0** - Primary UI framework
- **TypeScript 5.8.3** - Type-safe development
- **Vite 7.0.4** - Build tool and dev server

### UI Component Library
The project uses **shadcn/ui** (New York style variant), a collection of re-usable components built on top of Radix UI primitives:

#### Radix UI Primitives (v1.x - v2.x)
Complete set of headless, accessible UI components:
- `@radix-ui/react-accordion` (1.2.12)
- `@radix-ui/react-alert-dialog` (1.1.15)
- `@radix-ui/react-avatar` (1.1.10)
- `@radix-ui/react-checkbox` (1.3.3)
- `@radix-ui/react-collapsible` (1.1.12)
- `@radix-ui/react-context-menu` (2.2.16)
- `@radix-ui/react-dialog` (1.1.15)
- `@radix-ui/react-dropdown-menu` (2.1.16)
- `@radix-ui/react-hover-card` (1.1.15)
- `@radix-ui/react-label` (2.1.7)
- `@radix-ui/react-menubar` (1.1.16)
- `@radix-ui/react-popover` (1.1.15)
- `@radix-ui/react-progress` (1.1.7)
- `@radix-ui/react-radio-group` (1.3.8)
- `@radix-ui/react-scroll-area` (1.2.10)
- `@radix-ui/react-select` (2.2.6)
- `@radix-ui/react-separator` (1.1.7)
- `@radix-ui/react-slider` (1.3.6)
- `@radix-ui/react-slot` (1.2.3)
- `@radix-ui/react-switch` (1.2.6)
- `@radix-ui/react-tabs` (1.1.13)
- `@radix-ui/react-toggle` (1.1.10)
- `@radix-ui/react-toggle-group` (1.1.11)
- `@radix-ui/react-tooltip` (1.2.8)

#### Additional UI Libraries
- **lucide-react** (0.544.0) - Icon library
- **cmdk** (1.1.1) - Command menu component
- **sonner** (2.0.7) - Toast notifications
- **vaul** (1.1.2) - Drawer component
- **embla-carousel-react** (8.6.0) - Carousel component
- **react-day-picker** (9.11.0) - Date picker component

### Visualization & Charts
- **recharts** (2.15.4) - React charting library
- **lightweight-charts** (5.0.9) - TradingView lightweight charts
- **reactflow** (11.11.4) - Node-based workflow editor

### Layout & Interaction
- **react-grid-layout** (1.5.2) - Draggable and resizable grid layouts
- **@dnd-kit/core** (6.3.1) - Drag and drop toolkit
- **@dnd-kit/sortable** (10.0.0) - Sortable drag and drop

---

## 2. Styling Approach

### Tailwind CSS v4 (CSS-First Configuration)
The project uses **Tailwind CSS 4.1.13** with the new CSS-first configuration approach:

**File:** `fincept-terminal-desktop/src/App.css`

#### CSS Architecture
```css
@import "tailwindcss";
@import "tw-animate-css";
```

### Key Styling Features
1. **CSS Variables** - All design tokens defined as CSS custom properties
2. **Tailwind Utilities** - Utility-first CSS framework
3. **Class Composition** - Using `clsx` and `tailwind-merge` for dynamic classes
4. **Custom Scrollbars** - Transparent scrollbar styling for modern aesthetics
5. **Dark Mode Support** - Full dark mode theming via `.dark` class

### Utility Libraries
- **tailwind-merge** (3.3.1) - Merges Tailwind classes intelligently
- **clsx** (2.1.1) - Conditional className utility
- **class-variance-authority** (0.7.1) - Component variant management
- **tw-animate-css** (1.4.0) - Additional animation utilities

### Styling Utilities
**File:** `fincept-terminal-desktop/src/lib/utils.ts`

```typescript
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

This `cn()` utility is used throughout the codebase for combining class names with proper precedence.

---

## 3. Design System & Theme Configuration

### Design Tokens (CSS Variables)

#### Core Configuration
**File:** `fincept-terminal-desktop/components.json`

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/App.css",
    "baseColor": "stone",
    "cssVariables": true,
    "prefix": ""
  },
  "iconLibrary": "lucide"
}
```

### Color System

#### Light Mode Colors
**File:** `fincept-terminal-desktop/src/App.css` (lines 44-77)

```css
:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.147 0.004 49.25);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.147 0.004 49.25);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.147 0.004 49.25);
  --primary: oklch(0.216 0.006 56.043);
  --primary-foreground: oklch(0.985 0.001 106.423);
  --secondary: oklch(0.97 0.001 106.424);
  --secondary-foreground: oklch(0.216 0.006 56.043);
  --muted: oklch(0.97 0.001 106.424);
  --muted-foreground: oklch(0.553 0.013 58.071);
  --accent: oklch(0.97 0.001 106.424);
  --accent-foreground: oklch(0.216 0.006 56.043);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.923 0.003 48.717);
  --input: oklch(0.923 0.003 48.717);
  --ring: oklch(0.709 0.01 56.259);
  --chart-1: oklch(0.646 0.222 41.116);
  --chart-2: oklch(0.6 0.118 184.704);
  --chart-3: oklch(0.398 0.07 227.392);
  --chart-4: oklch(0.828 0.189 84.429);
  --chart-5: oklch(0.769 0.188 70.08);
  --sidebar: oklch(0.985 0.001 106.423);
  --sidebar-foreground: oklch(0.147 0.004 49.25);
  --sidebar-primary: oklch(0.216 0.006 56.043);
  --sidebar-primary-foreground: oklch(0.985 0.001 106.423);
  --sidebar-accent: oklch(0.97 0.001 106.424);
  --sidebar-accent-foreground: oklch(0.216 0.006 56.043);
  --sidebar-border: oklch(0.923 0.003 48.717);
  --sidebar-ring: oklch(0.709 0.01 56.259);
}
```

#### Dark Mode Colors
**File:** `fincept-terminal-desktop/src/App.css` (lines 79-111)

```css
.dark {
  --background: oklch(0.147 0.004 49.25);
  --foreground: oklch(0.985 0.001 106.423);
  --card: oklch(0.216 0.006 56.043);
  --card-foreground: oklch(0.985 0.001 106.423);
  --popover: oklch(0.216 0.006 56.043);
  --popover-foreground: oklch(0.985 0.001 106.423);
  --primary: oklch(0.923 0.003 48.717);
  --primary-foreground: oklch(0.216 0.006 56.043);
  --secondary: oklch(0.268 0.007 34.298);
  --secondary-foreground: oklch(0.985 0.001 106.423);
  --muted: oklch(0.268 0.007 34.298);
  --muted-foreground: oklch(0.709 0.01 56.259);
  --accent: oklch(0.268 0.007 34.298);
  --accent-foreground: oklch(0.985 0.001 106.423);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.553 0.013 58.071);
  --chart-1: oklch(0.488 0.243 264.376);
  --chart-2: oklch(0.696 0.17 162.48);
  --chart-3: oklch(0.769 0.188 70.08);
  --chart-4: oklch(0.627 0.265 303.9);
  --chart-5: oklch(0.645 0.246 16.439);
  --sidebar: oklch(0.216 0.006 56.043);
  --sidebar-foreground: oklch(0.985 0.001 106.423);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0.001 106.423);
  --sidebar-accent: oklch(0.268 0.007 34.298);
  --sidebar-accent-foreground: oklch(0.985 0.001 106.423);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.553 0.013 58.071);
}
```

### Terminal-Specific Theme System

**File:** `fincept-terminal-desktop/src/services/terminalThemeService.ts`

This service provides Bloomberg Terminal-inspired theming with:

#### 5 Predefined Color Themes

1. **Bloomberg Classic** (Default)
   - Primary: Orange (#FFA500)
   - Secondary: Green (#00C800)
   - Background: Black (#000000)
   - Text: White (#FFFFFF)

2. **Matrix Green**
   - Primary: Bright Green (#00FF41)
   - Background: Black (#000000)
   - Text: Matrix Green (#00FF41)

3. **Blue Terminal**
   - Primary: Cyan Blue (#00D4FF)
   - Background: Deep Blue (#000814)
   - Panel: Dark Blue (#001D3D)

4. **Amber Retro**
   - Primary: Amber (#FFAA00)
   - Background: Black (#000000)
   - Text: Amber (#FFBB33)

5. **Purple Neon**
   - Primary: Purple (#C864FF)
   - Background: Dark Purple (#0D0221)
   - Panel: Deep Purple (#1A0B3C)

#### Font Settings
```typescript
interface FontSettings {
  family: string;
  baseSize: number;
  weight: 'normal' | 'semibold' | 'bold';
  italic: boolean;
}
```

**Available Font Families:**
- Consolas (Default)
- Courier New
- Monaco
- Fira Code
- JetBrains Mono
- Source Code Pro

#### Typography Scale
Dynamic font sizes based on base size (default: 11px):
- **Heading (H1):** base × 1.8
- **Subheading (H2):** base × 1.4
- **Body:** base × 1
- **Small:** base × 0.85
- **Tiny:** base × 0.7

### Theme Context
**File:** `fincept-terminal-desktop/src/contexts/ThemeContext.tsx`

Provides application-wide theme management with:
- Theme state management
- Color customization
- Font customization
- Theme persistence (localStorage)

---

## 4. Component Structure

### UI Component Library (41 Components)
**Location:** `fincept-terminal-desktop/src/components/ui/`

All components follow shadcn/ui patterns with Radix UI primitives:

#### Core Components
1. **accordion.tsx** - Collapsible content sections
2. **alert.tsx** - Alert notifications
3. **alert-dialog.tsx** - Modal confirmation dialogs
4. **aspect-ratio.tsx** - Aspect ratio container
5. **avatar.tsx** - User avatars
6. **badge.tsx** - Labels and tags
7. **breadcrumb.tsx** - Navigation breadcrumbs
8. **button.tsx** - Primary action component
9. **calendar.tsx** - Date selection
10. **card.tsx** - Content container
11. **carousel.tsx** - Image/content carousel
12. **chart.tsx** - Chart container component
13. **checkbox.tsx** - Checkbox input
14. **collapsible.tsx** - Collapsible sections
15. **command.tsx** - Command palette
16. **context-menu.tsx** - Right-click menu
17. **dialog.tsx** - Modal dialogs
18. **drawer.tsx** - Slide-out panel
19. **dropdown-menu.tsx** - Dropdown menus
20. **hover-card.tsx** - Hover popover
21. **input.tsx** - Text input field
22. **label.tsx** - Form labels
23. **menubar.tsx** - Application menu bar
24. **popover.tsx** - Popup content
25. **progress.tsx** - Progress indicator
26. **radio-group.tsx** - Radio button group
27. **scroll-area.tsx** - Custom scrollable area
28. **select.tsx** - Select dropdown
29. **separator.tsx** - Visual divider
30. **sheet.tsx** - Side panel
31. **sidebar.tsx** - Navigation sidebar
32. **skeleton.tsx** - Loading placeholder
33. **slider.tsx** - Range slider
34. **sonner.tsx** - Toast notifications
35. **switch.tsx** - Toggle switch
36. **table.tsx** - Data table
37. **tabs.tsx** - Tab navigation
38. **textarea.tsx** - Multi-line text input
39. **toggle.tsx** - Toggle button
40. **toggle-group.tsx** - Toggle button group
41. **tooltip.tsx** - Hover tooltips

### Component Patterns

#### Example: Button Component with CVA
**File:** `fincept-terminal-desktop/src/components/ui/button.tsx`

```typescript
import { cva, type VariantProps } from "class-variance-authority"

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline: "border bg-background shadow-xs hover:bg-accent",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2 has-[>svg]:px-3",
        sm: "h-8 rounded-md gap-1.5 px-3",
        lg: "h-10 rounded-md px-6",
        icon: "size-9",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)
```

#### Card Component Structure
**File:** `fincept-terminal-desktop/src/components/ui/card.tsx`

```typescript
function Card({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="card"
      className={cn(
        "bg-card text-card-foreground flex flex-col gap-6 rounded-xl border py-6 shadow-sm",
        className
      )}
      {...props}
    />
  )
}
```

Includes sub-components:
- `CardHeader`
- `CardTitle`
- `CardDescription`
- `CardContent`
- `CardFooter`
- `CardAction`

### Application Components
**Location:** `fincept-terminal-desktop/src/components/`

#### Component Categories

1. **Auth Components** (`auth/`)
   - LoginScreen
   - RegisterScreen
   - ForgotPasswordScreen
   - HelpScreen

2. **Common Components** (`common/`)
   - BackgroundPattern
   - Header
   - Footer
   - MarkdownRenderer
   - RecordedContextsManager
   - RecordingControlPanel
   - AutoUpdater
   - ContextSelector

3. **Dashboard Components** (`dashboard/`)
   - DashboardScreen (main container)

4. **Chart Components** (`charts/`)
   - CandlestickChart
   - SimpleLineChart

5. **Tab Components** (`tabs/`)
   - 30+ functional tabs including:
     - DashboardTab, MarketsTab, ChatTab
     - EquityResearchTab, AnalyticsTab
     - GeopoliticsTab, MaritimeTab
     - NodeEditorTab (workflow editor)
     - CodeEditorTab, DocsTab
     - DataSourcesTab, DataMappingTab
     - And many more specialized tabs

6. **Payment Components** (`payment/`)
   - PaymentProcessingScreen
   - PaymentSuccessScreen
   - PaymentOverlay

7. **Info/Legal Components** (`info/`)
   - ContactUsScreen
   - TermsOfServiceScreen
   - PrivacyPolicyScreen
   - TrademarksScreen

8. **Settings Components** (`settings/`)
   - Settings configuration UI

---

## 5. Layout System

### Grid System
The application uses **react-grid-layout** for dashboard customization:

```json
{
  "react-grid-layout": "^1.5.2",
  "@types/react-grid-layout": "^1.3.5"
}
```

### Responsive Design

#### Tailwind CSS Breakpoints (Default)
- **sm:** 640px
- **md:** 768px
- **lg:** 1024px
- **xl:** 1280px
- **2xl:** 1536px

#### Usage Example
Found in DashboardScreen.tsx:
```tsx
<DialogContent className="sm:max-w-[500px]">
```

### Layout Patterns

1. **Tab-Based Navigation**
   - Primary layout uses Tabs component
   - Each tab contains specialized functionality

2. **Drag-and-Drop Grid**
   - Resizable panels using react-grid-layout
   - Sortable lists using @dnd-kit

3. **Sidebar Navigation**
   - Collapsible sidebar component
   - Context-aware navigation

4. **Modal/Dialog System**
   - Layered dialogs using Radix UI
   - Alert dialogs for confirmations

### Flex & Grid Utilities
Extensive use of Tailwind's flex and grid utilities:
- `flex`, `inline-flex`
- `flex-col`, `flex-row`
- `grid`, `grid-cols-*`
- `gap-*`, `space-*`

---

## 6. Visual Assets

### Icons

#### Lucide React (Primary Icon Library)
**Package:** `lucide-react` (0.544.0)

Usage throughout components:
```typescript
import { 
  Maximize, Minimize, Download, Settings, 
  RefreshCw, User, Database, Eye, HelpCircle, 
  LogOut, CheckCircle2, XCircle 
} from 'lucide-react';
```

**Benefits:**
- 1000+ open-source icons
- Tree-shakeable (only import what you use)
- Consistent design language
- Fully customizable via props

### Images

#### Application Screenshots
**Location:** `/images/`
- Chat.png
- Dashboard.png
- Economy.png
- Equity.png
- Forum.png
- Geopolitics.png
- GlobalTrade.png
- Markets.png
- TradeAnalysis.png

#### Application Icons
**Location:** `/fincept-terminal-desktop/src-tauri/icons/`

Platform-specific icons:
- **Windows:** icon.ico, various Square*.png
- **macOS:** icon.icns
- **Linux:** icon.png (various sizes)
- **Android:** android/ directory
- **iOS:** ios/ directory

Sizes available:
- 32x32, 64x64, 128x128
- Windows Store logos (various sizes)
- High-DPI variants (@2x)

### SVG Assets
**Location:** `/fincept-terminal-desktop/public/`
- tauri.svg
- vite.svg

**Location:** `/fincept-terminal-desktop/src/assets/`
- react.svg

### Background Patterns

**Component:** BackgroundPattern.tsx

Custom SVG geometric crosshatch pattern:
```typescript
<pattern
  id="geometric-grid"
  x="0" y="0" width="60" height="60"
  patternUnits="userSpaceOnUse"
>
  <line x1="0" y1="0" x2="60" y2="60" 
    stroke="#333333" strokeWidth="0.8" opacity="0.8" />
  <line x1="60" y1="0" x2="0" y2="60" 
    stroke="#333333" strokeWidth="0.8" opacity="0.8" />
</pattern>
```

### Font Loading
No custom web fonts are bundled. The system relies on:
- System fonts (Consolas, Courier New, Monaco)
- Google Fonts (can be added via link in index.html for Fira Code, JetBrains Mono, Source Code Pro)

---

## 7. Design Configuration Files

### Primary Configuration Files

1. **`components.json`** - shadcn/ui configuration
   ```json
   {
     "$schema": "https://ui.shadcn.com/schema.json",
     "style": "new-york",
     "rsc": false,
     "tsx": true,
     "tailwind": {
       "config": "",
       "css": "src/App.css",
       "baseColor": "stone",
       "cssVariables": true,
       "prefix": ""
     },
     "iconLibrary": "lucide"
   }
   ```

2. **`src/App.css`** - Tailwind CSS and theme variables
   - Main styling entry point
   - CSS variable definitions
   - Custom scrollbar styles
   - Dark mode variants

3. **`vite.config.ts`** - Build configuration
   ```typescript
   import tailwindcss from '@tailwindcss/vite'
   
   export default defineConfig({
     plugins: [react(), tailwindcss()],
     resolve: {
       alias: {
         "@": path.resolve(__dirname, "./src"),
       },
     },
   })
   ```

4. **`tsconfig.json`** - TypeScript paths
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

5. **`package.json`** - Dependency management
   - All UI library versions
   - Dev dependencies
   - Build scripts

### Service Configuration

**`src/services/terminalThemeService.ts`**
- Color theme definitions
- Font configurations
- Theme persistence logic
- Default theme values

---

## 8. Customization Points

### Theme Customization

#### 1. CSS Variables (Quick Changes)
**File:** `src/App.css`

Change colors by modifying OKLCH values:
```css
:root {
  --primary: oklch(0.216 0.006 56.043);  /* Change primary color */
  --radius: 0.625rem;                    /* Change border radius */
}
```

#### 2. Terminal Color Themes
**File:** `src/services/terminalThemeService.ts`

Add new color themes:
```typescript
export const COLOR_THEMES: Record<string, ColorTheme> = {
  'my-custom-theme': {
    name: 'My Custom Theme',
    primary: '#YOUR_COLOR',
    secondary: '#YOUR_COLOR',
    // ... other colors
  }
}
```

#### 3. Component Variants
**Example:** `src/components/ui/button.tsx`

Add new button variants:
```typescript
const buttonVariants = cva("base-classes", {
  variants: {
    variant: {
      default: "...",
      myVariant: "bg-custom text-custom hover:bg-custom/90",
    },
  }
})
```

### Typography Customization

#### 1. Font Family
**File:** `src/services/terminalThemeService.ts`

Add new fonts to the list:
```typescript
export const FONT_FAMILIES = [
  { value: 'Consolas', label: 'Consolas' },
  { value: 'Your Font', label: 'Your Font' },
  // Add more...
];
```

#### 2. Font Sizes
Modify base size or scale multipliers:
```typescript
getFontSizes(): FontSizes {
  const base = this.theme.font.baseSize;
  return {
    heading: `${base * 1.8}px`,    // Adjust multiplier
    subheading: `${base * 1.4}px`,
    body: `${base}px`,
    small: `${base * 0.85}px`,
    tiny: `${base * 0.7}px`
  };
}
```

### Layout Customization

#### 1. Border Radius
Global border radius via CSS variable:
```css
:root {
  --radius: 0.625rem;  /* Change this value */
}
```

Computed variants in theme:
```css
--radius-sm: calc(var(--radius) - 4px);
--radius-md: calc(var(--radius) - 2px);
--radius-lg: var(--radius);
--radius-xl: calc(var(--radius) + 4px);
```

#### 2. Spacing
Tailwind's default spacing scale (0-96) is available.
Customize in Tailwind v4 via CSS:
```css
@theme {
  --spacing-*: custom-value;
}
```

#### 3. Grid Layout
Customize dashboard grid via react-grid-layout props:
- Column count
- Row height
- Breakpoints
- Margins and padding

### Component Customization

#### 1. Using the `cn()` Utility
All components accept `className` prop:
```typescript
<Button className="my-custom-classes">Click</Button>
```

#### 2. Component Composition
Extend existing components:
```typescript
import { Button } from "@/components/ui/button"

export function MyButton(props) {
  return <Button variant="outline" {...props} />
}
```

#### 3. Shadcn Component Overrides
Directly modify files in `src/components/ui/` to change base component behavior.

### Icon Customization

#### 1. Using Lucide Icons
```typescript
import { YourIcon } from 'lucide-react'

<YourIcon size={24} color="orange" strokeWidth={2} />
```

#### 2. Custom SVG Icons
Add custom SVGs to assets and import:
```typescript
import CustomIcon from '@/assets/custom-icon.svg'
```

### Scrollbar Customization

**File:** `src/App.css`

Modify scrollbar appearance:
```css
::-webkit-scrollbar {
  width: 10px;  /* Change width */
}

::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.1);  /* Change color */
  border-radius: 5px;
}
```

### Dark Mode Customization

Toggle dark mode by adding/removing `.dark` class to root element:
```typescript
document.documentElement.classList.add('dark')
// or
document.documentElement.classList.remove('dark')
```

Managed via:
- **next-themes** (0.4.6) package
- `ThemeProvider` context

---

## 9. Path Aliases

Configured in both `tsconfig.json` and `vite.config.ts`:

```typescript
"@/*" → "./src/*"
```

**Usage:**
```typescript
import { Button } from "@/components/ui/button"
import { cn } from "@/lib/utils"
import { useAuth } from "@/contexts/AuthContext"
```

---

## 10. Build & Development

### Development Server
```bash
npm run dev
# Runs Vite dev server on port 1420
```

### Build
```bash
npm run build
# 1. TypeScript compilation (tsc)
# 2. Vite build with chunking strategy
```

### Preview
```bash
npm run preview
# Preview production build
```

### Tauri Desktop
```bash
npm run tauri dev     # Desktop development
npm run tauri build   # Desktop production build
```

---

## 11. Key Design Principles

### 1. Terminal-First Aesthetic
- Black backgrounds with high-contrast text
- Monospace font options
- Bloomberg Terminal inspiration
- Professional financial UI

### 2. Accessibility
- Radix UI primitives ensure ARIA compliance
- Keyboard navigation support
- Focus states and ring indicators
- Screen reader friendly

### 3. Performance
- Code splitting via Vite
- Tree-shaking for icons and components
- Lazy loading for tabs
- Optimized chunk strategy

### 4. Consistency
- Unified color system via CSS variables
- Component variant patterns with CVA
- Consistent spacing and sizing
- Predictable component APIs

### 5. Customization
- CSS variable overrides
- Component variant extensions
- Theme switching
- User preferences persistence

---

## 12. Summary

### Design Stack Overview
```
┌─────────────────────────────────────────┐
│         React 19 + TypeScript           │
├─────────────────────────────────────────┤
│    shadcn/ui (New York) + Radix UI      │
├─────────────────────────────────────────┤
│         Tailwind CSS v4 (CSS-first)     │
├─────────────────────────────────────────┤
│    CSS Variables (OKLCH color space)   │
├─────────────────────────────────────────┤
│         Lucide Icons + Custom SVG       │
├─────────────────────────────────────────┤
│    Terminal Theme Service (Bloomberg)   │
└─────────────────────────────────────────┘
```

### Component Architecture
- **41 UI components** (shadcn/ui with Radix UI)
- **30+ application tabs** for different features
- **5 predefined terminal themes**
- **6 monospace font options**
- **Full dark mode support**

### Styling Approach
- **Utility-first CSS** via Tailwind
- **CSS Variables** for theming
- **OKLCH color space** for better color manipulation
- **Component variants** via class-variance-authority
- **Class composition** via clsx + tailwind-merge

### Customization Flexibility
- Modify CSS variables for quick theme changes
- Add new terminal color themes easily
- Extend component variants
- Override component styles
- Custom icon integration
- Font and typography control

---

## File Reference

### Key Design Files
```
fincept-terminal-desktop/
├── components.json                    # shadcn/ui config
├── vite.config.ts                     # Build config with Tailwind
├── tsconfig.json                      # TypeScript paths
├── src/
│   ├── App.css                        # Main styles & theme variables
│   ├── lib/
│   │   └── utils.ts                   # cn() utility
│   ├── components/
│   │   ├── ui/                        # 41 shadcn/ui components
│   │   ├── common/                    # Shared components
│   │   ├── tabs/                      # Feature tabs
│   │   ├── charts/                    # Chart components
│   │   └── ...
│   ├── contexts/
│   │   └── ThemeContext.tsx          # Theme management
│   └── services/
│       └── terminalThemeService.ts   # Terminal themes
└── public/                            # Static assets
```

---

## Conclusion

Fincept Terminal uses a modern, professional design system built on industry-standard tools. The combination of React, Tailwind CSS v4, shadcn/ui, and Radix UI provides a robust foundation for building a Bloomberg Terminal-inspired financial application.

The design system is highly customizable through CSS variables, component variants, and a dedicated terminal theme service, making it easy to adapt the UI to different use cases while maintaining consistency and accessibility.
