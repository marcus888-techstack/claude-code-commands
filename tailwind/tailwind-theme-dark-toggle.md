# Tailwind Theme Dark Toggle

## Purpose
Toggle dark mode on/off for your React/Next.js application using Tailwind CSS v4 dark mode utilities

## Context
Use to enable or disable dark mode across your entire application. Supports both class-based and media-query based dark mode strategies. Essential for providing users with theme preferences and reducing eye strain in low-light conditions. Built for Tailwind CSS v4 with React/Next.js.

## Parameters
- `$MODE` - Dark mode toggle action
  - Required
  - Options: `on`, `off`, `toggle`, `auto`
- `$STRATEGY` - Dark mode strategy to use
  - Optional
  - Default: `class`
  - Options: `class`, `media`
- `$SELECTOR` - Root element selector
  - Optional
  - Default: `html`
  - Example: `html`, `body`, `.app-root`

## Steps

### 1. Set up Tailwind CSS v4 configuration
```css
/* app/globals.css */
@import "tailwindcss";

/* Define theme variables for light and dark modes */
@theme {
  --color-background: #ffffff;
  --color-foreground: #000000;
  --color-card: #f9fafb;
  --color-card-foreground: #111827;
  --color-primary: #3b82f6;
  --color-primary-foreground: #ffffff;
  --color-muted: #f3f4f6;
  --color-muted-foreground: #6b7280;
}

/* Dark mode theme overrides */
.dark {
  --color-background: #0a0a0a;
  --color-foreground: #ededed;
  --color-card: #141414;
  --color-card-foreground: #e5e5e5;
  --color-primary: #60a5fa;
  --color-primary-foreground: #000000;
  --color-muted: #262626;
  --color-muted-foreground: #a3a3a3;
}

/* Media query strategy */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: #0a0a0a;
    --color-foreground: #ededed;
    /* ... other dark mode variables */
  }
}
```
Configures Tailwind CSS v4 with theme variables for dark mode.

### 2. Create Theme Provider component (TypeScript)
```typescript
// app/components/theme-provider.tsx
'use client'

import { createContext, useContext, useEffect, useState } from 'react'

type Theme = 'light' | 'dark' | 'system'

interface ThemeContextType {
  theme: Theme
  setTheme: (theme: Theme) => void
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

export function ThemeProvider({
  children,
  defaultTheme = 'system',
  storageKey = 'theme',
  enableSystem = true,
  disableTransitionOnChange = true,
}: {
  children: React.ReactNode
  defaultTheme?: Theme
  storageKey?: string
  enableSystem?: boolean
  disableTransitionOnChange?: boolean
}) {
  const [theme, setTheme] = useState<Theme>(defaultTheme)
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
    const stored = localStorage.getItem(storageKey) as Theme
    if (stored) {
      setTheme(stored)
    }
  }, [storageKey])

  useEffect(() => {
    if (!mounted) return

    const root = window.document.documentElement

    // Remove old theme classes
    root.classList.remove('light', 'dark')

    if (theme === 'system' && enableSystem) {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark'
        : 'light'
      root.classList.add(systemTheme)
    } else if (theme !== 'system') {
      root.classList.add(theme)
    }

    // Disable transitions temporarily
    if (disableTransitionOnChange) {
      root.classList.add('theme-transition-disabled')
      setTimeout(() => {
        root.classList.remove('theme-transition-disabled')
      }, 0)
    }
  }, [theme, enableSystem, mounted, disableTransitionOnChange])

  const value = {
    theme,
    setTheme: (newTheme: Theme) => {
      localStorage.setItem(storageKey, newTheme)
      setTheme(newTheme)
    },
  }

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  )
}

export const useTheme = () => {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider')
  }
  return context
}
```
Provides theme context and management for the application.

### 3. Create Theme Toggle component
```typescript
// app/components/theme-toggle.tsx
'use client'

import { useTheme } from './theme-provider'
import { Moon, Sun, Monitor } from 'lucide-react'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <div className="flex items-center gap-2">
      <button
        onClick={() => setTheme('light')}
        className={`p-2 rounded-lg transition-colors ${
          theme === 'light'
            ? 'bg-primary text-primary-foreground'
            : 'bg-muted text-muted-foreground hover:bg-muted/80'
        }`}
        aria-label="Light mode"
      >
        <Sun className="h-5 w-5" />
      </button>
      <button
        onClick={() => setTheme('dark')}
        className={`p-2 rounded-lg transition-colors ${
          theme === 'dark'
            ? 'bg-primary text-primary-foreground'
            : 'bg-muted text-muted-foreground hover:bg-muted/80'
        }`}
        aria-label="Dark mode"
      >
        <Moon className="h-5 w-5" />
      </button>
      <button
        onClick={() => setTheme('system')}
        className={`p-2 rounded-lg transition-colors ${
          theme === 'system'
            ? 'bg-primary text-primary-foreground'
            : 'bg-muted text-muted-foreground hover:bg-muted/80'
        }`}
        aria-label="System theme"
      >
        <Monitor className="h-5 w-5" />
      </button>
    </div>
  )
}
```
Creates a theme toggle component with icon buttons.

### 4. Simple toggle button variant
```typescript
// app/components/theme-toggle-simple.tsx
'use client'

import { useTheme } from './theme-provider'

export function ThemeToggleSimple() {
  const { theme, setTheme } = useTheme()

  const toggleTheme = () => {
    if (theme === 'system') {
      setTheme('light')
    } else if (theme === 'light') {
      setTheme('dark')
    } else {
      setTheme('system')
    }
  }

  return (
    <button
      onClick={toggleTheme}
      className="px-4 py-2 rounded-lg bg-muted text-muted-foreground hover:bg-muted/80 transition-colors"
    >
      {theme === 'system' ? 'System' : theme === 'light' ? 'Light' : 'Dark'}
    </button>
  )
}
```
Simple toggle button that cycles through themes.

### 5. Add to Next.js layout
```typescript
// app/layout.tsx
import { ThemeProvider } from '@/components/theme-provider'
import { ThemeToggle } from '@/components/theme-toggle'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="bg-background text-foreground">
        <ThemeProvider
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          <header className="p-4 border-b">
            <nav className="flex justify-between items-center">
              <h1 className="text-xl font-bold">My App</h1>
              <ThemeToggle />
            </nav>
          </header>
          <main>{children}</main>
        </ThemeProvider>
      </body>
    </html>
  )
}
```
Integrates theme provider into the app layout.

### 6. Use with custom hooks
```typescript
// app/hooks/use-dark-mode.ts
import { useTheme } from '@/components/theme-provider'
import { useEffect, useState } from 'react'

export function useDarkMode() {
  const { theme } = useTheme()
  const [isDark, setIsDark] = useState(false)

  useEffect(() => {
    if (theme === 'dark') {
      setIsDark(true)
    } else if (theme === 'light') {
      setIsDark(false)
    } else {
      // System theme
      const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
      setIsDark(mediaQuery.matches)

      const handler = (e: MediaQueryListEvent) => setIsDark(e.matches)
      mediaQuery.addEventListener('change', handler)
      return () => mediaQuery.removeEventListener('change', handler)
    }
  }, [theme])

  return isDark
}
```
Custom hook to check if dark mode is active.

### 7. Add smooth transitions CSS
```css
/* app/globals.css - add after imports */

/* Smooth theme transitions */
* {
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* Disable transitions during theme change */
.theme-transition-disabled * {
  transition: none !important;
}

/* Update meta theme color with CSS */
@media (prefers-color-scheme: dark) {
  meta[name="theme-color"] {
    content: "#0a0a0a";
  }
}

.dark meta[name="theme-color"] {
  content: "#0a0a0a";
}

/* Custom scrollbar for dark mode */
.dark::-webkit-scrollbar-track {
  background: var(--color-muted);
}

.dark::-webkit-scrollbar-thumb {
  background: var(--color-muted-foreground);
}
```
Adds smooth transitions and dark mode scrollbar styling.

## Validation
- Theme provider correctly applies classes
- localStorage persists theme preference
- System preference detection works
- Transitions are smooth without flashing
- All components respond to theme changes

## Error Handling
- **"useTheme must be used within ThemeProvider"** - Wrap component in ThemeProvider
- **"localStorage not defined"** - Check for client-side rendering
- **"Hydration mismatch"** - Use suppressHydrationWarning on html element
- **"Theme not applying"** - Verify CSS imports and class names

## Safety Notes
- Always use suppressHydrationWarning on html element
- Test color contrast in both modes for accessibility
- Ensure images and icons work in both themes
- Consider using CSS custom properties for theme colors
- Respect system preferences when possible

## Examples
- **Basic implementation**
  ```tsx
  // Simple dark mode toggle in any component
  import { ThemeToggle } from '@/components/theme-toggle'
  
  export function Header() {
    return <ThemeToggle />
  }
  ```

- **Conditional rendering based on theme**
  ```tsx
  import { useDarkMode } from '@/hooks/use-dark-mode'
  
  export function Logo() {
    const isDark = useDarkMode()
    return (
      <img 
        src={isDark ? '/logo-dark.svg' : '/logo-light.svg'} 
        alt="Logo"
      />
    )
  }
  ```

- **Theme-aware component**
  ```tsx
  export function Card({ children }: { children: React.ReactNode }) {
    return (
      <div className="bg-card text-card-foreground p-6 rounded-lg shadow-sm">
        {children}
      </div>
    )
  }
  ```