# Fincept Terminal - Design Quick Reference

A quick reference guide for the Fincept Terminal design system. For comprehensive documentation, see [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) and [DESIGN_EXAMPLES.md](DESIGN_EXAMPLES.md).

---

## 🎨 Design Stack

```
React 19 + TypeScript 5.8
↓
shadcn/ui (New York) + Radix UI
↓
Tailwind CSS v4 (CSS-first)
↓
CSS Variables (OKLCH colors)
↓
Terminal Theme Service
```

---

## 📦 Key Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| **react** | 19.1.0 | UI Framework |
| **tailwindcss** | 4.1.13 | Utility CSS |
| **@radix-ui/react-*** | Various | Headless UI components |
| **lucide-react** | 0.544.0 | Icon library |
| **recharts** | 2.15.4 | Charts |
| **react-grid-layout** | 1.5.2 | Draggable grids |
| **class-variance-authority** | 0.7.1 | Variant management |

---

## 🎨 Color System

### Design Tokens (CSS Variables)

#### Light Mode
```css
--background: oklch(1 0 0)
--foreground: oklch(0.147 0.004 49.25)
--primary: oklch(0.216 0.006 56.043)
--border: oklch(0.923 0.003 48.717)
--radius: 0.625rem
```

#### Dark Mode
```css
--background: oklch(0.147 0.004 49.25)
--foreground: oklch(0.985 0.001 106.423)
--primary: oklch(0.923 0.003 48.717)
--border: oklch(1 0 0 / 10%)
```

### Terminal Color Themes

**File:** `src/services/terminalThemeService.ts`

1. **Bloomberg Classic** (Default)
   - Orange primary, green secondary, black background
2. **Matrix Green**
   - Bright green on black
3. **Blue Terminal**
   - Cyan/blue with dark blue background
4. **Amber Retro**
   - Amber/orange vintage terminal
5. **Purple Neon**
   - Purple neon cyberpunk aesthetic

---

## 🧩 Component Library (41 Components)

**Location:** `src/components/ui/`

### Essential Components
- **button** - Primary actions with 6 variants
- **card** - Content containers
- **dialog** - Modal overlays
- **dropdown-menu** - Context menus
- **input** - Text inputs
- **select** - Dropdowns
- **tabs** - Tab navigation
- **table** - Data tables
- **tooltip** - Hover hints

### Form Components
- checkbox, radio-group, switch, slider, textarea, label

### Layout Components
- separator, scroll-area, sheet, sidebar, collapsible, accordion

### Feedback Components
- alert, alert-dialog, toast (sonner), progress, skeleton

### Special Components
- calendar, carousel, chart, command (palette), context-menu

---

## 🎯 Common Patterns

### Using Components

```tsx
import { Button } from "@/components/ui/button"
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card"
import { Input } from "@/components/ui/input"
import { Download } from "lucide-react"

// Basic usage
<Button variant="default">Click me</Button>
<Button variant="destructive" size="lg">Delete</Button>
<Button variant="outline">
  <Download className="mr-2 h-4 w-4" />
  Download
</Button>

// Card layout
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>
```

### Using Theme

```tsx
import { useTheme } from '@/contexts/ThemeContext'

function MyComponent() {
  const { colors, fontSize, fontFamily } = useTheme()
  
  return (
    <div style={{
      color: colors.text,
      backgroundColor: colors.background,
      fontFamily: fontFamily,
      fontSize: fontSize.body
    }}>
      Content
    </div>
  )
}
```

### Styling with Tailwind

```tsx
// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => <Card key={item.id}>...</Card>)}
</div>

// Flex layout
<div className="flex items-center justify-between gap-4">
  <span>Left</span>
  <span>Right</span>
</div>

// Using design tokens
<div className="bg-background text-foreground border border-border rounded-lg p-4">
  Content with theme colors
</div>
```

---

## 📐 Layout System

### Responsive Breakpoints
- **sm:** 640px
- **md:** 768px
- **lg:** 1024px
- **xl:** 1280px
- **2xl:** 1536px

### Grid Layout
Uses `react-grid-layout` for draggable/resizable dashboard panels.

```tsx
import GridLayout from 'react-grid-layout'

<GridLayout cols={12} rowHeight={30}>
  {/* Grid items */}
</GridLayout>
```

---

## 🖼️ Icons & Assets

### Lucide Icons
```tsx
import { Home, Settings, User, Download } from 'lucide-react'

<Home className="h-6 w-6 text-primary" />
<Settings size={24} color="#FFA500" strokeWidth={2} />
```

**1000+ icons available**, all tree-shakeable

### Application Assets
- **Icons:** `/fincept-terminal-desktop/src-tauri/icons/`
- **Images:** `/images/` (screenshots)
- **SVG:** `/fincept-terminal-desktop/public/` (logos)

---

## 🎨 Typography

### Font Options
- Consolas (default)
- Courier New
- Monaco
- Fira Code
- JetBrains Mono
- Source Code Pro

### Type Scale (Dynamic)
```typescript
heading: baseSize × 1.8    // H1
subheading: baseSize × 1.4 // H2
body: baseSize × 1         // Normal
small: baseSize × 0.85     // Labels
tiny: baseSize × 0.7       // Metadata
```

Default base size: 11px (Bloomberg Terminal style)

---

## 🔧 Configuration Files

| File | Purpose |
|------|---------|
| `components.json` | shadcn/ui config |
| `src/App.css` | Tailwind + CSS variables |
| `vite.config.ts` | Build config |
| `tsconfig.json` | TypeScript paths |
| `src/services/terminalThemeService.ts` | Theme definitions |

---

## 🛠️ Quick Customization

### 1. Change Primary Color
**File:** `src/App.css`
```css
:root {
  --primary: oklch(0.216 0.006 56.043); /* Your color here */
}
```

### 2. Add Terminal Theme
**File:** `src/services/terminalThemeService.ts`
```typescript
export const COLOR_THEMES = {
  'my-theme': {
    name: 'My Theme',
    primary: '#YOUR_COLOR',
    // ... other colors
  }
}
```

### 3. Extend Button Variant
**File:** `src/components/ui/button.tsx`
```typescript
const buttonVariants = cva("...", {
  variants: {
    variant: {
      // Add your variant
      myVariant: "bg-custom text-custom hover:bg-custom/90"
    }
  }
})
```

### 4. Change Border Radius
**File:** `src/App.css`
```css
:root {
  --radius: 0.5rem; /* Your radius */
}
```

---

## 🌙 Dark Mode

Toggle via class on root element:
```typescript
// Enable dark mode
document.documentElement.classList.add('dark')

// Disable dark mode
document.documentElement.classList.remove('dark')
```

Or use `next-themes` package (already included):
```tsx
import { useTheme } from 'next-themes'

const { theme, setTheme } = useTheme()
setTheme('dark') // or 'light'
```

---

## 📁 Project Structure

```
fincept-terminal-desktop/
├── src/
│   ├── App.css              # Main styles + CSS variables
│   ├── components/
│   │   ├── ui/              # 41 shadcn/ui components
│   │   ├── common/          # Shared components
│   │   ├── tabs/            # 30+ feature tabs
│   │   ├── charts/          # Chart components
│   │   └── ...
│   ├── contexts/
│   │   └── ThemeContext.tsx # Theme state
│   ├── services/
│   │   └── terminalThemeService.ts # Theme logic
│   └── lib/
│       └── utils.ts         # cn() utility
├── components.json          # shadcn/ui config
├── vite.config.ts           # Vite + Tailwind
└── public/                  # Static assets
```

---

## 🔍 Path Aliases

```typescript
"@/*" → "./src/*"
```

Usage:
```tsx
import { Button } from "@/components/ui/button"
import { cn } from "@/lib/utils"
import { useTheme } from "@/contexts/ThemeContext"
```

---

## 🚀 Commands

```bash
npm run dev      # Dev server (port 1420)
npm run build    # Production build
npm run preview  # Preview build
npm run tauri    # Desktop app commands
```

---

## 📚 Learn More

- **Full Documentation:** [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md)
- **Code Examples:** [DESIGN_EXAMPLES.md](DESIGN_EXAMPLES.md)
- **shadcn/ui:** https://ui.shadcn.com/
- **Radix UI:** https://www.radix-ui.com/
- **Tailwind CSS:** https://tailwindcss.com/
- **Lucide Icons:** https://lucide.dev/

---

## 💡 Pro Tips

1. **Use `cn()` utility** for combining classes
2. **Leverage CSS variables** for consistent theming
3. **Use Radix UI states** (data-state, aria-*) for styling
4. **Import icons individually** to reduce bundle size
5. **Extend variants** instead of creating new components
6. **Test dark mode** for all custom colors
7. **Follow existing patterns** in codebase
8. **Use shadcn/ui CLI** to add new components

---

**Last Updated:** 2024-12-05  
**Version:** Fincept Terminal 3.0.11
