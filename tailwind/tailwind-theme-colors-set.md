# Tailwind Theme Colors Set

## Purpose
Set primary, secondary, and accent colors throughout your React/Next.js application using Tailwind CSS v4's CSS-first configuration

## Context
Use to dynamically change the color scheme of your application without rebuilding CSS. Enables theme customization, brand color updates, and user-personalized color preferences. Works with Tailwind CSS v4's new @theme directive and CSS custom properties.

## Parameters
- `$COLOR_TYPE` - Type of color to set
  - Required
  - Options: `primary`, `secondary`, `accent`, `neutral`, `success`, `warning`, `error`
- `$COLOR_VALUE` - Color value to apply
  - Required
  - Format: Hex (#3B82F6), RGB (59 130 246), HSL (217 91% 60%)
- `$SHADES` - Generate color shades automatically
  - Optional
  - Default: `yes`
  - Options: `yes`, `no`

## Steps

### 1. Set up Tailwind CSS v4 color system
```css
/* app/globals.css */
@import "tailwindcss";

/* Define color system using @theme */
@theme {
  /* Primary colors with shades */
  --color-primary-50: #eff6ff;
  --color-primary-100: #dbeafe;
  --color-primary-200: #bfdbfe;
  --color-primary-300: #93c5fd;
  --color-primary-400: #60a5fa;
  --color-primary-500: #3b82f6;
  --color-primary-600: #2563eb;
  --color-primary-700: #1d4ed8;
  --color-primary-800: #1e40af;
  --color-primary-900: #1e3a8a;
  --color-primary-950: #172554;
  
  /* Secondary colors */
  --color-secondary-50: #f5f3ff;
  --color-secondary-100: #ede9fe;
  --color-secondary-200: #ddd6fe;
  --color-secondary-300: #c4b5fd;
  --color-secondary-400: #a78bfa;
  --color-secondary-500: #8b5cf6;
  --color-secondary-600: #7c3aed;
  --color-secondary-700: #6d28d9;
  --color-secondary-800: #5b21b6;
  --color-secondary-900: #4c1d95;
  --color-secondary-950: #2e1065;
  
  /* Semantic colors */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  
  /* Neutral colors */
  --color-neutral-50: #f9fafb;
  --color-neutral-100: #f3f4f6;
  --color-neutral-200: #e5e7eb;
  --color-neutral-300: #d1d5db;
  --color-neutral-400: #9ca3af;
  --color-neutral-500: #6b7280;
  --color-neutral-600: #4b5563;
  --color-neutral-700: #374151;
  --color-neutral-800: #1f2937;
  --color-neutral-900: #111827;
  --color-neutral-950: #030712;
}

/* Use colors in utilities */
@utility text-primary {
  color: var(--color-primary-500);
}

@utility bg-primary {
  background-color: var(--color-primary-500);
}

@utility border-primary {
  border-color: var(--color-primary-500);
}
```
Sets up Tailwind CSS v4 color system with @theme directive.

### 2. Create Color Management Hook (TypeScript)
```typescript
// app/hooks/use-theme-colors.ts
'use client'

import { useEffect, useState } from 'react'

interface ColorShades {
  50: string
  100: string
  200: string
  300: string
  400: string
  500: string
  600: string
  700: string
  800: string
  900: string
  950: string
}

type ColorType = 'primary' | 'secondary' | 'accent' | 'neutral' | 'success' | 'warning' | 'error'

export function useThemeColors() {
  const [colors, setColors] = useState<Record<ColorType, string>>({
    primary: '#3b82f6',
    secondary: '#8b5cf6',
    accent: '#f59e0b',
    neutral: '#6b7280',
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
  })

  // Helper functions for color conversion
  const hexToRgb = (hex: string): { r: number; g: number; b: number } => {
    const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
    return result
      ? {
          r: parseInt(result[1], 16),
          g: parseInt(result[2], 16),
          b: parseInt(result[3], 16),
        }
      : { r: 0, g: 0, b: 0 }
  }

  const rgbToHsl = (r: number, g: number, b: number) => {
    r /= 255
    g /= 255
    b /= 255

    const max = Math.max(r, g, b)
    const min = Math.min(r, g, b)
    let h = 0
    let s = 0
    const l = (max + min) / 2

    if (max !== min) {
      const d = max - min
      s = l > 0.5 ? d / (2 - max - min) : d / (max + min)

      switch (max) {
        case r:
          h = ((g - b) / d + (g < b ? 6 : 0)) / 6
          break
        case g:
          h = ((b - r) / d + 2) / 6
          break
        case b:
          h = ((r - g) / d + 4) / 6
          break
      }
    }

    return {
      h: Math.round(h * 360),
      s: Math.round(s * 100),
      l: Math.round(l * 100),
    }
  }

  const hslToRgb = (h: number, s: number, l: number) => {
    h /= 360
    s /= 100
    l /= 100

    let r, g, b

    if (s === 0) {
      r = g = b = l
    } else {
      const hue2rgb = (p: number, q: number, t: number) => {
        if (t < 0) t += 1
        if (t > 1) t -= 1
        if (t < 1 / 6) return p + (q - p) * 6 * t
        if (t < 1 / 2) return q
        if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6
        return p
      }

      const q = l < 0.5 ? l * (1 + s) : l + s - l * s
      const p = 2 * l - q
      r = hue2rgb(p, q, h + 1 / 3)
      g = hue2rgb(p, q, h)
      b = hue2rgb(p, q, h - 1 / 3)
    }

    return {
      r: Math.round(r * 255),
      g: Math.round(g * 255),
      b: Math.round(b * 255),
    }
  }

  const generateShades = (hex: string): ColorShades => {
    const rgb = hexToRgb(hex)
    const hsl = rgbToHsl(rgb.r, rgb.g, rgb.b)

    const shades: ColorShades = {
      50: '',
      100: '',
      200: '',
      300: '',
      400: '',
      500: hex,
      600: '',
      700: '',
      800: '',
      900: '',
      950: '',
    }

    // Generate lighter shades
    const lightnessSteps = [95, 90, 80, 70, 60]
    lightnessSteps.forEach((lightness, index) => {
      const newRgb = hslToRgb(hsl.h, hsl.s, lightness)
      const shade = `#${((1 << 24) + (newRgb.r << 16) + (newRgb.g << 8) + newRgb.b).toString(16).slice(1)}`
      const key = [50, 100, 200, 300, 400][index] as keyof ColorShades
      shades[key] = shade
    })

    // Generate darker shades
    const darknessSteps = [50, 40, 30, 20, 10]
    darknessSteps.forEach((lightness, index) => {
      const newRgb = hslToRgb(hsl.h, hsl.s, lightness)
      const shade = `#${((1 << 24) + (newRgb.r << 16) + (newRgb.g << 8) + newRgb.b).toString(16).slice(1)}`
      const key = [600, 700, 800, 900, 950][index] as keyof ColorShades
      shades[key] = shade
    })

    return shades
  }

  const setThemeColor = (type: ColorType, value: string, generateAllShades: boolean = true) => {
    const root = document.documentElement

    if (generateAllShades && value.startsWith('#')) {
      const shades = generateShades(value)
      
      Object.entries(shades).forEach(([shade, color]) => {
        root.style.setProperty(`--color-${type}-${shade}`, color)
      })
    } else {
      root.style.setProperty(`--color-${type}-500`, value)
    }

    // Update state
    setColors(prev => ({ ...prev, [type]: value }))

    // Save to localStorage
    const savedColors = JSON.parse(localStorage.getItem('theme-colors') || '{}')
    savedColors[type] = value
    localStorage.setItem('theme-colors', JSON.stringify(savedColors))
  }

  // Load saved colors on mount
  useEffect(() => {
    const savedColors = JSON.parse(localStorage.getItem('theme-colors') || '{}')
    Object.entries(savedColors).forEach(([type, value]) => {
      setThemeColor(type as ColorType, value as string)
    })
  }, [])

  return { colors, setThemeColor, generateShades }
}
```
Provides hooks for dynamic color management.

### 3. Create Color Picker Component
```typescript
// app/components/color-picker.tsx
'use client'

import { useState } from 'react'
import { useThemeColors } from '@/hooks/use-theme-colors'

type ColorType = 'primary' | 'secondary' | 'accent' | 'neutral' | 'success' | 'warning' | 'error'

interface ColorPickerProps {
  type: ColorType
  label: string
  generateShades?: boolean
}

export function ColorPicker({ type, label, generateShades = true }: ColorPickerProps) {
  const { colors, setThemeColor } = useThemeColors()
  const [colorValue, setColorValue] = useState(colors[type])

  const handleColorChange = (value: string) => {
    setColorValue(value)
    setThemeColor(type, value, generateShades)
  }

  return (
    <div className="flex items-center gap-4">
      <label htmlFor={`color-${type}`} className="text-sm font-medium">
        {label}
      </label>
      <div className="flex items-center gap-2">
        <input
          id={`color-${type}`}
          type="color"
          value={colorValue}
          onChange={(e) => handleColorChange(e.target.value)}
          className="h-10 w-10 cursor-pointer rounded border border-neutral-300"
        />
        <input
          type="text"
          value={colorValue}
          onChange={(e) => handleColorChange(e.target.value)}
          className="w-28 px-2 py-1 text-sm border border-neutral-300 rounded"
          placeholder="#000000"
        />
      </div>
    </div>
  )
}
```
Creates an interactive color picker component.

### 4. Create Theme Customizer Panel
```typescript
// app/components/theme-customizer.tsx
'use client'

import { ColorPicker } from './color-picker'
import { useState } from 'react'

export function ThemeCustomizer() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <>
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="fixed bottom-4 right-4 p-3 bg-primary text-white rounded-full shadow-lg hover:shadow-xl transition-shadow"
        aria-label="Open theme customizer"
      >
        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01" />
        </svg>
      </button>

      {isOpen && (
        <div className="fixed bottom-20 right-4 w-80 bg-white dark:bg-neutral-900 rounded-lg shadow-xl p-6 space-y-4">
          <h3 className="text-lg font-semibold mb-4">Theme Colors</h3>
          
          <ColorPicker type="primary" label="Primary" />
          <ColorPicker type="secondary" label="Secondary" />
          <ColorPicker type="accent" label="Accent" />
          <ColorPicker type="neutral" label="Neutral" />
          
          <div className="pt-4 border-t">
            <h4 className="text-sm font-medium mb-2">Semantic Colors</h4>
            <ColorPicker type="success" label="Success" generateShades={false} />
            <ColorPicker type="warning" label="Warning" generateShades={false} />
            <ColorPicker type="error" label="Error" generateShades={false} />
          </div>

          <button
            onClick={() => {
              localStorage.removeItem('theme-colors')
              window.location.reload()
            }}
            className="w-full mt-4 px-4 py-2 text-sm bg-neutral-100 dark:bg-neutral-800 rounded hover:bg-neutral-200 dark:hover:bg-neutral-700 transition-colors"
          >
            Reset to Defaults
          </button>
        </div>
      )}
    </>
  )
}
```
Creates a floating theme customization panel.

### 5. Example usage in components
```typescript
// app/components/button.tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'accent'
  children: React.ReactNode
  onClick?: () => void
}

export function Button({ variant = 'primary', children, onClick }: ButtonProps) {
  const variantClasses = {
    primary: 'bg-primary-500 hover:bg-primary-600 text-white',
    secondary: 'bg-secondary-500 hover:bg-secondary-600 text-white',
    accent: 'bg-accent-500 hover:bg-accent-600 text-white',
  }

  return (
    <button
      onClick={onClick}
      className={`px-4 py-2 rounded-lg font-medium transition-colors ${variantClasses[variant]}`}
    >
      {children}
    </button>
  )
}
```
Example button component using theme colors.

### 6. Color palette display component
```typescript
// app/components/color-palette.tsx
'use client'

import { useThemeColors } from '@/hooks/use-theme-colors'

export function ColorPalette() {
  const { colors } = useThemeColors()

  const shades = [50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950]

  return (
    <div className="space-y-8">
      {Object.entries(colors).map(([type, baseColor]) => (
        <div key={type}>
          <h3 className="text-lg font-semibold capitalize mb-2">{type}</h3>
          <div className="grid grid-cols-11 gap-2">
            {shades.map((shade) => (
              <div
                key={shade}
                className="aspect-square rounded"
                style={{
                  backgroundColor: `var(--color-${type}-${shade}, ${baseColor})`,
                }}
                title={`${type}-${shade}`}
              />
            ))}
          </div>
        </div>
      ))}
    </div>
  )
}
```
Displays the complete color palette.

### 7. Integration with Next.js
```typescript
// app/layout.tsx
import { ThemeCustomizer } from '@/components/theme-customizer'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        {children}
        <ThemeCustomizer />
      </body>
    </html>
  )
}
```
Adds theme customizer to the app layout.

## Validation
- CSS custom properties are updated dynamically
- Color shades are generated with proper contrast
- Changes persist across page reloads
- All components using color variables update instantly
- Color picker reflects current values

## Error Handling
- **"Invalid color format"** - Validate hex/rgb/hsl format before applying
- **"Colors not updating"** - Check CSS variable names and specificity
- **"Shades too similar"** - Adjust HSL generation algorithm
- **"localStorage not available"** - Add fallback for SSR

## Safety Notes
- Test color contrast for accessibility (WCAG standards)
- Ensure text remains readable on all backgrounds
- Consider colorblind users when choosing colors
- Provide a reset option to default colors
- Validate colors before applying

## Examples
- **Basic color setting**
  ```tsx
  import { useThemeColors } from '@/hooks/use-theme-colors'
  
  function MyComponent() {
    const { setThemeColor } = useThemeColors()
    
    return (
      <button onClick={() => setThemeColor('primary', '#3b82f6')}>
        Set Blue Theme
      </button>
    )
  }
  ```

- **Custom brand colors**
  ```tsx
  // Set brand colors on app initialization
  useEffect(() => {
    setThemeColor('primary', '#FF6B6B', true)
    setThemeColor('secondary', '#4ECDC4', true)
  }, [])
  ```

- **Theme presets**
  ```tsx
  const themePresets = {
    ocean: { primary: '#006BA6', secondary: '#0496FF' },
    forest: { primary: '#2D6A4F', secondary: '#40916C' },
    sunset: { primary: '#F77F00', secondary: '#FCBF49' }
  }
  
  function applyPreset(presetName: keyof typeof themePresets) {
    const preset = themePresets[presetName]
    Object.entries(preset).forEach(([type, color]) => {
      setThemeColor(type as any, color)
    })
  }
  ```