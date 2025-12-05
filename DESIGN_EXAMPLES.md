# Fincept Terminal - Design System Examples & Code Snippets

This document provides practical examples of using the design system components, styling patterns, and customization techniques.

---

## Table of Contents
1. [Button Variants](#button-variants)
2. [Color Theme Usage](#color-theme-usage)
3. [Card Layouts](#card-layouts)
4. [Form Components](#form-components)
5. [Typography Examples](#typography-examples)
6. [Icon Usage](#icon-usage)
7. [Layout Patterns](#layout-patterns)
8. [Dark Mode Implementation](#dark-mode-implementation)
9. [Custom Scrollbars](#custom-scrollbars)
10. [Component Composition](#component-composition)

---

## Button Variants

### Basic Button Usage

```tsx
import { Button } from "@/components/ui/button"

// Default button
<Button>Click me</Button>

// Variant buttons
<Button variant="default">Primary</Button>
<Button variant="destructive">Delete</Button>
<Button variant="outline">Outline</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>

// Size variants
<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
<Button size="icon">Icon</Button>

// With icons
import { Download } from 'lucide-react'

<Button>
  <Download className="mr-2 h-4 w-4" />
  Download
</Button>

// Custom styling
<Button className="bg-orange-500 hover:bg-orange-600">
  Custom Color
</Button>

// Disabled state
<Button disabled>Disabled</Button>

// As child (polymorphic)
<Button asChild>
  <a href="/download">Download Link</a>
</Button>
```

### Button Variant Definition
```typescript
// File: src/components/ui/button.tsx

const buttonVariants = cva(
  // Base classes
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline: "border bg-background shadow-xs hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md gap-1.5 px-3",
        lg: "h-10 rounded-md px-6",
        icon: "size-9",
      },
    },
  }
)
```

---

## Color Theme Usage

### Using Terminal Themes

```typescript
import { useTheme } from '@/contexts/ThemeContext'

function MyComponent() {
  const { theme, colors, updateTheme } = useTheme()
  
  // Access current colors
  const primaryColor = colors.primary      // e.g., '#FFA500'
  const backgroundColor = colors.background // e.g., '#000000'
  
  // Apply colors inline
  return (
    <div style={{ 
      color: colors.text,
      backgroundColor: colors.panel 
    }}>
      Content with theme colors
    </div>
  )
}
```

### Switching Color Themes

```typescript
import { terminalThemeService } from '@/services/terminalThemeService'

// Switch to a different theme
function ThemeSwitcher() {
  const { updateTheme } = useTheme()
  
  const switchTheme = (themeName: string) => {
    const newTheme = {
      ...terminalThemeService.getTheme(),
      colors: COLOR_THEMES[themeName]
    }
    updateTheme(newTheme)
  }
  
  return (
    <div>
      <button onClick={() => switchTheme('bloomberg-classic')}>
        Bloomberg Classic
      </button>
      <button onClick={() => switchTheme('matrix-green')}>
        Matrix Green
      </button>
      <button onClick={() => switchTheme('blue-terminal')}>
        Blue Terminal
      </button>
    </div>
  )
}
```

### Creating Custom Color Theme

```typescript
// File: src/services/terminalThemeService.ts

export const COLOR_THEMES: Record<string, ColorTheme> = {
  // ... existing themes ...
  
  'custom-sunset': {
    name: 'Sunset Theme',
    primary: '#FF6B35',      // Coral
    secondary: '#F7931E',    // Orange
    success: '#4CAF50',      // Green
    alert: '#F44336',        // Red
    warning: '#FFC107',      // Amber
    info: '#2196F3',         // Blue
    accent: '#E91E63',       // Pink
    purple: '#9C27B0',       // Purple
    text: '#FFFFFF',         // White
    textMuted: '#B0B0B0',    // Light Gray
    background: '#1A1A2E',   // Dark Blue
    panel: '#16213E'         // Darker Blue
  }
}
```

### Using CSS Variables

```tsx
// Using Tailwind with design tokens
<div className="bg-background text-foreground">
  <div className="bg-card text-card-foreground border border-border rounded-lg p-4">
    <h2 className="text-primary">Primary Color Heading</h2>
    <p className="text-muted-foreground">Muted text</p>
    <button className="bg-primary text-primary-foreground">
      Primary Button
    </button>
  </div>
</div>

// Using CSS variables directly in style
<div style={{
  backgroundColor: 'var(--background)',
  color: 'var(--foreground)',
  borderColor: 'var(--border)',
  borderRadius: 'var(--radius)'
}}>
  Content
</div>
```

---

## Card Layouts

### Basic Card

```tsx
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from "@/components/ui/card"
import { Button } from "@/components/ui/button"

function BasicCard() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Card Title</CardTitle>
        <CardDescription>Card description goes here</CardDescription>
      </CardHeader>
      <CardContent>
        <p>Main content of the card</p>
      </CardContent>
      <CardFooter>
        <Button>Action</Button>
      </CardFooter>
    </Card>
  )
}
```

### Card with Action Button

```tsx
import { CardAction } from "@/components/ui/card"
import { Settings } from "lucide-react"

function CardWithAction() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Settings</CardTitle>
        <CardDescription>Configure your preferences</CardDescription>
        <CardAction>
          <Button variant="ghost" size="icon">
            <Settings className="h-4 w-4" />
          </Button>
        </CardAction>
      </CardHeader>
      <CardContent>
        {/* Settings content */}
      </CardContent>
    </Card>
  )
}
```

### Grid of Cards

```tsx
function CardGrid() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <Card>
        <CardHeader>
          <CardTitle>Card 1</CardTitle>
        </CardHeader>
        <CardContent>Content 1</CardContent>
      </Card>
      <Card>
        <CardHeader>
          <CardTitle>Card 2</CardTitle>
        </CardHeader>
        <CardContent>Content 2</CardContent>
      </Card>
      <Card>
        <CardHeader>
          <CardTitle>Card 3</CardTitle>
        </CardHeader>
        <CardContent>Content 3</CardContent>
      </Card>
    </div>
  )
}
```

---

## Form Components

### Complete Form Example

```tsx
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Checkbox } from "@/components/ui/checkbox"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"
import { Textarea } from "@/components/ui/textarea"
import { Switch } from "@/components/ui/switch"
import { Button } from "@/components/ui/button"

function CompleteForm() {
  return (
    <form className="space-y-4">
      {/* Text Input */}
      <div>
        <Label htmlFor="name">Name</Label>
        <Input id="name" placeholder="Enter your name" />
      </div>
      
      {/* Select Dropdown */}
      <div>
        <Label htmlFor="theme">Theme</Label>
        <Select>
          <SelectTrigger>
            <SelectValue placeholder="Select theme" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="bloomberg">Bloomberg Classic</SelectItem>
            <SelectItem value="matrix">Matrix Green</SelectItem>
            <SelectItem value="blue">Blue Terminal</SelectItem>
          </SelectContent>
        </Select>
      </div>
      
      {/* Textarea */}
      <div>
        <Label htmlFor="description">Description</Label>
        <Textarea id="description" placeholder="Enter description" />
      </div>
      
      {/* Checkbox */}
      <div className="flex items-center space-x-2">
        <Checkbox id="terms" />
        <Label htmlFor="terms">Accept terms and conditions</Label>
      </div>
      
      {/* Switch */}
      <div className="flex items-center space-x-2">
        <Switch id="notifications" />
        <Label htmlFor="notifications">Enable notifications</Label>
      </div>
      
      {/* Submit Button */}
      <Button type="submit">Submit</Button>
    </form>
  )
}
```

### Input with Validation States

```tsx
function InputValidation() {
  return (
    <div className="space-y-4">
      {/* Default */}
      <Input placeholder="Default input" />
      
      {/* Disabled */}
      <Input placeholder="Disabled" disabled />
      
      {/* Error state (using aria-invalid) */}
      <Input 
        placeholder="Invalid input" 
        aria-invalid={true}
        className="border-destructive"
      />
      
      {/* Success state */}
      <Input 
        placeholder="Valid input" 
        className="border-green-500"
      />
    </div>
  )
}
```

---

## Typography Examples

### Using Terminal Typography

```tsx
import { useTheme } from '@/contexts/ThemeContext'

function TypographyExample() {
  const { fontSize } = useTheme()
  
  return (
    <div>
      <h1 style={{ fontSize: fontSize.heading }}>
        Heading (H1)
      </h1>
      
      <h2 style={{ fontSize: fontSize.subheading }}>
        Subheading (H2)
      </h2>
      
      <p style={{ fontSize: fontSize.body }}>
        Body text
      </p>
      
      <small style={{ fontSize: fontSize.small }}>
        Small text for labels
      </small>
      
      <span style={{ fontSize: fontSize.tiny }}>
        Tiny text
      </span>
    </div>
  )
}
```

### Typography with Tailwind

```tsx
function TailwindTypography() {
  return (
    <div className="space-y-4">
      <h1 className="text-4xl font-bold text-foreground">
        Large Heading
      </h1>
      
      <h2 className="text-2xl font-semibold text-foreground">
        Medium Heading
      </h2>
      
      <p className="text-base text-foreground">
        Regular paragraph text
      </p>
      
      <p className="text-sm text-muted-foreground">
        Smaller muted text
      </p>
      
      <p className="text-xs text-muted-foreground">
        Extra small text
      </p>
    </div>
  )
}
```

### Monospace Terminal Text

```tsx
function MonospaceText() {
  const { fontFamily, fontWeight, fontStyle } = useTheme()
  
  return (
    <div style={{ 
      fontFamily: fontFamily,
      fontWeight: fontWeight,
      fontStyle: fontStyle 
    }}>
      <pre>Terminal-style monospace text</pre>
      <code>Code snippet in terminal font</code>
    </div>
  )
}
```

---

## Icon Usage

### Lucide Icons

```tsx
import { 
  Home, Settings, User, Search, Bell, 
  Download, Upload, Edit, Trash2, Plus,
  ChevronRight, ChevronDown, X, Check
} from 'lucide-react'

function IconExamples() {
  return (
    <div className="space-y-4">
      {/* Basic icon */}
      <Home className="h-6 w-6" />
      
      {/* With color */}
      <Settings className="h-6 w-6 text-primary" />
      
      {/* Custom size and color */}
      <User size={24} color="#FFA500" />
      
      {/* In button */}
      <Button>
        <Download className="mr-2 h-4 w-4" />
        Download
      </Button>
      
      {/* Icon button */}
      <Button variant="ghost" size="icon">
        <Search className="h-4 w-4" />
      </Button>
      
      {/* With stroke width */}
      <Bell strokeWidth={2.5} className="h-6 w-6" />
      
      {/* Grid of icons */}
      <div className="flex gap-2">
        <Edit className="h-5 w-5" />
        <Trash2 className="h-5 w-5 text-destructive" />
        <Plus className="h-5 w-5 text-green-500" />
        <Check className="h-5 w-5 text-green-500" />
        <X className="h-5 w-5 text-destructive" />
      </div>
    </div>
  )
}
```

### Icon with Terminal Theme

```tsx
function ThemedIcon() {
  const { colors } = useTheme()
  
  return (
    <Settings 
      size={24} 
      color={colors.primary}
      strokeWidth={2}
    />
  )
}
```

---

## Layout Patterns

### Flex Layouts

```tsx
// Horizontal layout
<div className="flex items-center gap-2">
  <span>Item 1</span>
  <span>Item 2</span>
  <span>Item 3</span>
</div>

// Vertical layout
<div className="flex flex-col gap-4">
  <div>Row 1</div>
  <div>Row 2</div>
  <div>Row 3</div>
</div>

// Space between
<div className="flex items-center justify-between">
  <span>Left</span>
  <span>Right</span>
</div>

// Centered
<div className="flex items-center justify-center h-screen">
  <div>Centered content</div>
</div>
```

### Grid Layouts

```tsx
// Auto-fit grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
  {items.map(item => (
    <Card key={item.id}>
      {/* Card content */}
    </Card>
  ))}
</div>

// Fixed columns
<div className="grid grid-cols-3 gap-4">
  <div>Column 1</div>
  <div>Column 2</div>
  <div>Column 3</div>
</div>

// Unequal columns
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-8">Main content (8/12)</div>
  <div className="col-span-4">Sidebar (4/12)</div>
</div>
```

### Responsive Layout

```tsx
function ResponsiveLayout() {
  return (
    <div className="container mx-auto px-4">
      {/* Header */}
      <header className="py-4">
        <h1 className="text-2xl md:text-3xl lg:text-4xl">
          Responsive Title
        </h1>
      </header>
      
      {/* Main content */}
      <main className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Sidebar - hidden on mobile */}
        <aside className="hidden lg:block lg:col-span-1">
          Sidebar
        </aside>
        
        {/* Main content */}
        <div className="col-span-1 lg:col-span-2">
          Main content
        </div>
      </main>
    </div>
  )
}
```

---

## Dark Mode Implementation

### Toggle Dark Mode

```tsx
import { useEffect, useState } from 'react'

function DarkModeToggle() {
  const [isDark, setIsDark] = useState(false)
  
  useEffect(() => {
    // Check initial theme
    const isDarkMode = document.documentElement.classList.contains('dark')
    setIsDark(isDarkMode)
  }, [])
  
  const toggleDarkMode = () => {
    if (isDark) {
      document.documentElement.classList.remove('dark')
    } else {
      document.documentElement.classList.add('dark')
    }
    setIsDark(!isDark)
  }
  
  return (
    <Button onClick={toggleDarkMode}>
      {isDark ? 'Light Mode' : 'Dark Mode'}
    </Button>
  )
}
```

### Using next-themes

```tsx
import { ThemeProvider as NextThemeProvider } from 'next-themes'

function App() {
  return (
    <NextThemeProvider attribute="class" defaultTheme="dark">
      {/* Your app */}
    </NextThemeProvider>
  )
}

// In components
import { useTheme } from 'next-themes'

function ThemeToggle() {
  const { theme, setTheme } = useTheme()
  
  return (
    <Button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      Toggle theme
    </Button>
  )
}
```

---

## Custom Scrollbars

### Using Custom Scrollbar Class

```tsx
// Apply custom scrollbar to specific element
<div className="custom-scrollbar h-96 overflow-y-auto">
  {/* Scrollable content */}
  {longList.map(item => (
    <div key={item.id}>{item.content}</div>
  ))}
</div>
```

### Custom Scrollbar Styles

```css
/* File: src/App.css */

/* Custom transparent scrollbar for specific elements */
.custom-scrollbar::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 165, 0, 0.2);
  border-radius: 4px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 165, 0, 0.4);
}

/* Firefox scrollbar */
.custom-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: rgba(255, 165, 0, 0.2) transparent;
}
```

### Using ScrollArea Component

```tsx
import { ScrollArea } from "@/components/ui/scroll-area"

function ScrollableContent() {
  return (
    <ScrollArea className="h-96 w-full rounded-md border p-4">
      {/* Content that scrolls */}
      <div className="space-y-4">
        {items.map(item => (
          <div key={item.id}>{item.content}</div>
        ))}
      </div>
    </ScrollArea>
  )
}
```

---

## Component Composition

### Dialog with Form

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from "@/components/ui/dialog"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Button } from "@/components/ui/button"

function FormDialog({ open, onOpenChange }) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-[425px]">
        <DialogHeader>
          <DialogTitle>Edit Profile</DialogTitle>
        </DialogHeader>
        <div className="grid gap-4 py-4">
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="name" className="text-right">
              Name
            </Label>
            <Input id="name" className="col-span-3" />
          </div>
          <div className="grid grid-cols-4 items-center gap-4">
            <Label htmlFor="username" className="text-right">
              Username
            </Label>
            <Input id="username" className="col-span-3" />
          </div>
        </div>
        <DialogFooter>
          <Button type="submit">Save changes</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  )
}
```

### Tabs with Content

```tsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"

function TabbedInterface() {
  return (
    <Tabs defaultValue="overview" className="w-full">
      <TabsList>
        <TabsTrigger value="overview">Overview</TabsTrigger>
        <TabsTrigger value="analytics">Analytics</TabsTrigger>
        <TabsTrigger value="reports">Reports</TabsTrigger>
      </TabsList>
      
      <TabsContent value="overview">
        <Card>
          <CardHeader>
            <CardTitle>Overview</CardTitle>
            <CardDescription>Your account overview</CardDescription>
          </CardHeader>
          <CardContent>
            Overview content
          </CardContent>
        </Card>
      </TabsContent>
      
      <TabsContent value="analytics">
        <Card>
          <CardHeader>
            <CardTitle>Analytics</CardTitle>
          </CardHeader>
          <CardContent>
            Analytics content
          </CardContent>
        </Card>
      </TabsContent>
      
      <TabsContent value="reports">
        <Card>
          <CardHeader>
            <CardTitle>Reports</CardTitle>
          </CardHeader>
          <CardContent>
            Reports content
          </CardContent>
        </Card>
      </TabsContent>
    </Tabs>
  )
}
```

### Dropdown Menu

```tsx
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from "@/components/ui/dropdown-menu"
import { Button } from "@/components/ui/button"
import { Settings, User, LogOut } from "lucide-react"

function UserMenu() {
  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon">
          <User className="h-5 w-5" />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem>
          <User className="mr-2 h-4 w-4" />
          <span>Profile</span>
        </DropdownMenuItem>
        <DropdownMenuItem>
          <Settings className="mr-2 h-4 w-4" />
          <span>Settings</span>
        </DropdownMenuItem>
        <DropdownMenuItem>
          <LogOut className="mr-2 h-4 w-4" />
          <span>Log out</span>
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

### Toast Notifications

```tsx
import { toast } from "sonner"

function NotificationExamples() {
  return (
    <div className="space-y-2">
      <Button onClick={() => toast.success("Success message")}>
        Success Toast
      </Button>
      
      <Button onClick={() => toast.error("Error message")}>
        Error Toast
      </Button>
      
      <Button onClick={() => toast.info("Info message")}>
        Info Toast
      </Button>
      
      <Button onClick={() => toast("Simple message")}>
        Simple Toast
      </Button>
      
      <Button onClick={() => toast.promise(
        fetchData(),
        {
          loading: 'Loading...',
          success: 'Data loaded!',
          error: 'Error loading data',
        }
      )}>
        Promise Toast
      </Button>
    </div>
  )
}
```

---

## Advanced Patterns

### Custom Hook for Terminal Theme

```tsx
// Custom hook to access terminal colors easily
function useTerminalColors() {
  const { colors } = useTheme()
  
  return {
    primary: colors.primary,
    secondary: colors.secondary,
    background: colors.background,
    text: colors.text,
    success: colors.success,
    alert: colors.alert,
    warning: colors.warning,
    
    // Utility functions
    getPrimaryRgb: () => hexToRgb(colors.primary),
    withOpacity: (color: string, opacity: number) => `${color}${Math.round(opacity * 255).toString(16)}`
  }
}

// Usage
function MyComponent() {
  const colors = useTerminalColors()
  
  return (
    <div style={{ backgroundColor: colors.withOpacity(colors.primary, 0.1) }}>
      Content with 10% opacity primary color
    </div>
  )
}
```

### Composing Variants

```tsx
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

// Define custom component with variants
const panelVariants = cva(
  "rounded-lg border p-4",
  {
    variants: {
      variant: {
        default: "bg-card text-card-foreground",
        primary: "bg-primary text-primary-foreground",
        secondary: "bg-secondary text-secondary-foreground",
      },
      size: {
        sm: "p-2",
        md: "p-4",
        lg: "p-6",
      },
      border: {
        none: "border-0",
        default: "border",
        thick: "border-2",
      }
    },
    defaultVariants: {
      variant: "default",
      size: "md",
      border: "default",
    },
  }
)

interface PanelProps extends React.HTMLAttributes<HTMLDivElement>, VariantProps<typeof panelVariants> {}

function Panel({ className, variant, size, border, ...props }: PanelProps) {
  return (
    <div
      className={cn(panelVariants({ variant, size, border, className }))}
      {...props}
    />
  )
}

// Usage
<Panel variant="primary" size="lg" border="thick">
  Custom panel with variants
</Panel>
```

---

## Performance Tips

### Lazy Loading Components

```tsx
import { lazy, Suspense } from 'react'
import { Skeleton } from "@/components/ui/skeleton"

// Lazy load heavy components
const HeavyChart = lazy(() => import('@/components/charts/HeavyChart'))

function ChartContainer() {
  return (
    <Suspense fallback={<Skeleton className="h-96 w-full" />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### Memoized Components

```tsx
import { memo } from 'react'

const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  // Expensive rendering logic
  return (
    <Card>
      {/* Content */}
    </Card>
  )
})
```

---

## Accessibility Examples

### Focus Management

```tsx
import { useRef, useEffect } from 'react'

function AccessibleDialog({ isOpen }) {
  const firstInputRef = useRef<HTMLInputElement>(null)
  
  useEffect(() => {
    if (isOpen) {
      firstInputRef.current?.focus()
    }
  }, [isOpen])
  
  return (
    <Dialog open={isOpen}>
      <DialogContent>
        <Input ref={firstInputRef} placeholder="First input gets focus" />
      </DialogContent>
    </Dialog>
  )
}
```

### ARIA Labels

```tsx
function AccessibleButton() {
  return (
    <>
      {/* Icon button with aria-label */}
      <Button variant="ghost" size="icon" aria-label="Close dialog">
        <X className="h-4 w-4" />
      </Button>
      
      {/* Button with aria-describedby */}
      <div>
        <Button id="submit-btn" aria-describedby="submit-help">
          Submit
        </Button>
        <p id="submit-help" className="text-sm text-muted-foreground">
          This will submit your form
        </p>
      </div>
    </>
  )
}
```

---

## Conclusion

These examples demonstrate the flexibility and power of the Fincept Terminal design system. The combination of shadcn/ui components, Tailwind utilities, and custom terminal theming provides a solid foundation for building sophisticated financial UIs.

For more examples, explore the component files in:
- `fincept-terminal-desktop/src/components/ui/`
- `fincept-terminal-desktop/src/components/tabs/`
- `fincept-terminal-desktop/src/components/common/`
