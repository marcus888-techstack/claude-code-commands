# Tailwind Theme Colors Palette

## Purpose
Apply pre-defined color palette presets to quickly theme your React/Next.js application with Tailwind CSS v4

## Context
Use to instantly apply cohesive color schemes like Material Design, corporate themes, or seasonal palettes. Ideal for rapid prototyping, A/B testing different color schemes, or providing users with preset theme options. Built for Tailwind CSS v4 with React/Next.js.

## Parameters
- `$PALETTE` - Name of the color palette to apply
  - Required
  - Options: `default`, `corporate`, `playful`, `dark`, `pastel`, `earth`, `ocean`, `sunset`, `forest`, `custom`
- `$VARIANT` - Palette variant
  - Optional
  - Default: `standard`
  - Options: `standard`, `vivid`, `muted`
- `$APPLY_TO` - Scope of palette application  
  - Optional
  - Default: `all`
  - Options: `all`, `primary-only`, `ui-colors`, `backgrounds`

## Steps

### 1. Define palette presets with Tailwind CSS v4
```typescript
// app/lib/color-palettes.ts
export interface ColorPalette {
  primary: string
  secondary: string
  accent: string
  neutral: string
  success: string
  warning: string
  error: string
  info?: string
}

export const colorPalettes: Record<string, ColorPalette> = {
  default: {
    primary: '#3b82f6',    // Blue
    secondary: '#8b5cf6',  // Purple
    accent: '#f59e0b',     // Amber
    neutral: '#6b7280',    // Gray
    success: '#10b981',    // Green
    warning: '#f59e0b',    // Amber
    error: '#ef4444',      // Red
    info: '#3b82f6',       // Blue
  },
  
  corporate: {
    primary: '#1e40af',    // Dark Blue
    secondary: '#64748b',  // Slate
    accent: '#0ea5e9',     // Sky
    neutral: '#475569',    // Slate
    success: '#059669',    // Emerald
    warning: '#d97706',    // Amber
    error: '#dc2626',      // Red
    info: '#0891b2',       // Cyan
  },
  
  playful: {
    primary: '#ec4899',    // Pink
    secondary: '#8b5cf6',  // Violet
    accent: '#f59e0b',     // Amber
    neutral: '#6b7280',    // Gray
    success: '#10b981',    // Emerald
    warning: '#f97316',    // Orange
    error: '#ef4444',      // Red
    info: '#06b6d4',       // Cyan
  },
  
  earth: {
    primary: '#92400e',    // Brown
    secondary: '#78716c',  // Stone
    accent: '#f97316',     // Orange
    neutral: '#57534e',    // Stone
    success: '#65a30d',    // Lime
    warning: '#ea580c',    // Orange
    error: '#b91c1c',      // Red
    info: '#0891b2',       // Cyan
  },
  
  ocean: {
    primary: '#0891b2',    // Cyan
    secondary: '#0e7490',  // Dark Cyan
    accent: '#06b6d4',     // Cyan
    neutral: '#64748b',    // Slate
    success: '#059669',    // Emerald
    warning: '#f59e0b',    // Amber
    error: '#e11d48',      // Rose
    info: '#0284c7',       // Sky
  },
  
  sunset: {
    primary: '#f97316',    // Orange
    secondary: '#dc2626',  // Red
    accent: '#fbbf24',     // Amber
    neutral: '#78716c',    // Stone
    success: '#84cc16',    // Lime
    warning: '#f59e0b',    // Amber
    error: '#e11d48',      // Rose
    info: '#7c3aed',       // Violet
  },
  
  forest: {
    primary: '#059669',    // Emerald
    secondary: '#047857',  // Dark Emerald
    accent: '#84cc16',     // Lime
    neutral: '#374151',    // Gray
    success: '#10b981',    // Emerald
    warning: '#f59e0b',    // Amber
    error: '#dc2626',      // Red
    info: '#0891b2',       // Cyan
  },
  
  pastel: {
    primary: '#c084fc',    // Light Purple
    secondary: '#f9a8d4',  // Light Pink
    accent: '#fde047',     // Light Yellow
    neutral: '#e5e7eb',    // Light Gray
    success: '#86efac',    // Light Green
    warning: '#fed7aa',    // Light Orange
    error: '#fecaca',      // Light Red
    info: '#bfdbfe',       // Light Blue
  },
  
  dark: {
    primary: '#818cf8',    // Indigo
    secondary: '#a78bfa',  // Purple
    accent: '#fbbf24',     // Amber
    neutral: '#374151',    // Dark Gray
    success: '#34d399',    // Emerald
    warning: '#fbbf24',    // Amber
    error: '#f87171',      // Light Red
    info: '#60a5fa',       // Light Blue
  },
}
```
Defines comprehensive color palette presets.

### 2. Create Palette Provider component
```typescript
// app/components/palette-provider.tsx
'use client'

import { createContext, useContext, useEffect, useState, ReactNode } from 'react'
import { colorPalettes, ColorPalette } from '@/lib/color-palettes'

type PaletteVariant = 'standard' | 'vivid' | 'muted'
type ApplyScope = 'all' | 'primary-only' | 'ui-colors' | 'backgrounds'

interface PaletteContextType {
  currentPalette: string
  variant: PaletteVariant
  applyPalette: (palette: string, variant?: PaletteVariant, scope?: ApplyScope) => void
  resetPalette: () => void
}

const PaletteContext = createContext<PaletteContextType | undefined>(undefined)

export function PaletteProvider({ children }: { children: ReactNode }) {
  const [currentPalette, setCurrentPalette] = useState('default')
  const [variant, setVariant] = useState<PaletteVariant>('standard')

  // Color manipulation functions
  const adjustColorVariant = (hex: string, variant: PaletteVariant): string => {
    const rgb = hexToRgb(hex)
    const hsl = rgbToHsl(rgb.r, rgb.g, rgb.b)

    switch (variant) {
      case 'vivid':
        hsl.s = Math.min(hsl.s * 1.3, 100)
        break
      case 'muted':
        hsl.s = hsl.s * 0.6
        break
    }

    const newRgb = hslToRgb(hsl.h, hsl.s, hsl.l)
    return rgbToHex(newRgb.r, newRgb.g, newRgb.b)
  }

  const generateColorShades = (baseColor: string): Record<string, string> => {
    const rgb = hexToRgb(baseColor)
    const hsl = rgbToHsl(rgb.r, rgb.g, rgb.b)
    
    const shades = {
      50: '', 100: '', 200: '', 300: '', 400: '',
      500: baseColor,
      600: '', 700: '', 800: '', 900: '', 950: ''
    }

    // Light shades
    const lightSteps = [95, 90, 80, 70, 60]
    lightSteps.forEach((lightness, i) => {
      const newRgb = hslToRgb(hsl.h, hsl.s, lightness)
      shades[[50, 100, 200, 300, 400][i] as keyof typeof shades] = 
        rgbToHex(newRgb.r, newRgb.g, newRgb.b)
    })

    // Dark shades
    const darkSteps = [45, 35, 25, 15, 5]
    darkSteps.forEach((lightness, i) => {
      const newRgb = hslToRgb(hsl.h, hsl.s, lightness)
      shades[[600, 700, 800, 900, 950][i] as keyof typeof shades] = 
        rgbToHex(newRgb.r, newRgb.g, newRgb.b)
    })

    return shades
  }

  const applyPalette = (
    paletteName: string, 
    paletteVariant: PaletteVariant = 'standard',
    scope: ApplyScope = 'all'
  ) => {
    const palette = colorPalettes[paletteName]
    if (!palette) return

    const root = document.documentElement
    
    // Determine which colors to apply based on scope
    const colorsToApply = 
      scope === 'primary-only' ? ['primary'] :
      scope === 'ui-colors' ? ['primary', 'secondary', 'accent'] :
      scope === 'backgrounds' ? ['neutral', 'primary'] :
      Object.keys(palette)

    // Apply each color with shades
    colorsToApply.forEach(colorType => {
      const baseColor = palette[colorType as keyof ColorPalette]
      if (!baseColor) return

      const adjustedColor = adjustColorVariant(baseColor, paletteVariant)
      const shades = generateColorShades(adjustedColor)

      Object.entries(shades).forEach(([shade, color]) => {
        root.style.setProperty(`--color-${colorType}-${shade}`, hexToRgb(color).toString())
      })
    })

    // Save selection
    setCurrentPalette(paletteName)
    setVariant(paletteVariant)
    
    localStorage.setItem('selected-palette', JSON.stringify({
      name: paletteName,
      variant: paletteVariant,
      scope
    }))
  }

  const resetPalette = () => {
    localStorage.removeItem('selected-palette')
    applyPalette('default', 'standard', 'all')
  }

  // Load saved palette on mount
  useEffect(() => {
    const saved = localStorage.getItem('selected-palette')
    if (saved) {
      const { name, variant, scope } = JSON.parse(saved)
      applyPalette(name, variant, scope)
    }
  }, [])

  return (
    <PaletteContext.Provider value={{
      currentPalette,
      variant,
      applyPalette,
      resetPalette
    }}>
      {children}
    </PaletteContext.Provider>
  )
}

export const usePalette = () => {
  const context = useContext(PaletteContext)
  if (!context) {
    throw new Error('usePalette must be used within PaletteProvider')
  }
  return context
}

// Helper functions
function hexToRgb(hex: string): { r: number; g: number; b: number } {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
  return result ? {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16)
  } : { r: 0, g: 0, b: 0 }
}

function rgbToHex(r: number, g: number, b: number): string {
  return '#' + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1)
}

function rgbToHsl(r: number, g: number, b: number) {
  r /= 255
  g /= 255
  b /= 255

  const max = Math.max(r, g, b)
  const min = Math.min(r, g, b)
  let h = 0, s = 0
  const l = (max + min) / 2

  if (max !== min) {
    const d = max - min
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
    
    switch (max) {
      case r: h = ((g - b) / d + (g < b ? 6 : 0)) / 6; break
      case g: h = ((b - r) / d + 2) / 6; break
      case b: h = ((r - g) / d + 4) / 6; break
    }
  }

  return {
    h: Math.round(h * 360),
    s: Math.round(s * 100),
    l: Math.round(l * 100)
  }
}

function hslToRgb(h: number, s: number, l: number) {
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
      if (t < 1/6) return p + (q - p) * 6 * t
      if (t < 1/2) return q
      if (t < 2/3) return p + (q - p) * (2/3 - t) * 6
      return p
    }

    const q = l < 0.5 ? l * (1 + s) : l + s - l * s
    const p = 2 * l - q
    r = hue2rgb(p, q, h + 1/3)
    g = hue2rgb(p, q, h)
    b = hue2rgb(p, q, h - 1/3)
  }

  return {
    r: Math.round(r * 255),
    g: Math.round(g * 255),
    b: Math.round(b * 255)
  }
}
```
Provides palette management with React context.

### 3. Create Palette Selector component
```typescript
// app/components/palette-selector.tsx
'use client'

import { usePalette } from './palette-provider'
import { colorPalettes } from '@/lib/color-palettes'

export function PaletteSelector() {
  const { currentPalette, variant, applyPalette, resetPalette } = usePalette()

  return (
    <div className="space-y-6">
      <div>
        <h3 className="text-lg font-semibold mb-4">Color Palettes</h3>
        <div className="grid grid-cols-2 md:grid-cols-3 gap-4">
          {Object.entries(colorPalettes).map(([name, palette]) => (
            <button
              key={name}
              onClick={() => applyPalette(name, variant)}
              className={`p-4 rounded-lg border-2 transition-all ${
                currentPalette === name
                  ? 'border-primary-500 shadow-lg'
                  : 'border-neutral-200 hover:border-neutral-300'
              }`}
            >
              <h4 className="font-medium capitalize mb-2">{name}</h4>
              <div className="grid grid-cols-4 gap-1">
                <div
                  className="h-8 rounded"
                  style={{ backgroundColor: palette.primary }}
                  title="Primary"
                />
                <div
                  className="h-8 rounded"
                  style={{ backgroundColor: palette.secondary }}
                  title="Secondary"
                />
                <div
                  className="h-8 rounded"
                  style={{ backgroundColor: palette.accent }}
                  title="Accent"
                />
                <div
                  className="h-8 rounded"
                  style={{ backgroundColor: palette.neutral }}
                  title="Neutral"
                />
              </div>
            </button>
          ))}
        </div>
      </div>

      <div>
        <h4 className="text-sm font-medium mb-2">Variant</h4>
        <div className="flex gap-2">
          {(['standard', 'vivid', 'muted'] as const).map((v) => (
            <button
              key={v}
              onClick={() => applyPalette(currentPalette, v)}
              className={`px-4 py-2 rounded-lg capitalize ${
                variant === v
                  ? 'bg-primary-500 text-white'
                  : 'bg-neutral-100 hover:bg-neutral-200'
              }`}
            >
              {v}
            </button>
          ))}
        </div>
      </div>

      <button
        onClick={resetPalette}
        className="w-full px-4 py-2 bg-neutral-100 hover:bg-neutral-200 rounded-lg transition-colors"
      >
        Reset to Default
      </button>
    </div>
  )
}
```
Creates an interactive palette selector UI.

### 4. Create Preview component
```typescript
// app/components/palette-preview.tsx
'use client'

export function PalettePreview() {
  return (
    <div className="space-y-8">
      {/* Color swatches */}
      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        <div className="space-y-2">
          <h4 className="text-sm font-medium">Primary</h4>
          <div className="grid grid-cols-6 gap-1">
            {[50, 100, 300, 500, 700, 900].map(shade => (
              <div
                key={shade}
                className={`aspect-square rounded bg-primary-${shade}`}
                title={`primary-${shade}`}
              />
            ))}
          </div>
        </div>
        
        <div className="space-y-2">
          <h4 className="text-sm font-medium">Secondary</h4>
          <div className="grid grid-cols-6 gap-1">
            {[50, 100, 300, 500, 700, 900].map(shade => (
              <div
                key={shade}
                className={`aspect-square rounded bg-secondary-${shade}`}
                title={`secondary-${shade}`}
              />
            ))}
          </div>
        </div>
        
        <div className="space-y-2">
          <h4 className="text-sm font-medium">Accent</h4>
          <div className="grid grid-cols-6 gap-1">
            {[50, 100, 300, 500, 700, 900].map(shade => (
              <div
                key={shade}
                className={`aspect-square rounded bg-accent-${shade}`}
                title={`accent-${shade}`}
              />
            ))}
          </div>
        </div>
        
        <div className="space-y-2">
          <h4 className="text-sm font-medium">Neutral</h4>
          <div className="grid grid-cols-6 gap-1">
            {[50, 100, 300, 500, 700, 900].map(shade => (
              <div
                key={shade}
                className={`aspect-square rounded bg-neutral-${shade}`}
                title={`neutral-${shade}`}
              />
            ))}
          </div>
        </div>
      </div>

      {/* UI Examples */}
      <div className="space-y-4">
        <h4 className="text-lg font-semibold">UI Preview</h4>
        
        <div className="flex gap-2">
          <button className="px-4 py-2 bg-primary-500 text-white rounded-lg hover:bg-primary-600">
            Primary Button
          </button>
          <button className="px-4 py-2 bg-secondary-500 text-white rounded-lg hover:bg-secondary-600">
            Secondary Button
          </button>
          <button className="px-4 py-2 bg-accent-500 text-white rounded-lg hover:bg-accent-600">
            Accent Button
          </button>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div className="p-6 bg-primary-50 border border-primary-200 rounded-lg">
            <h5 className="font-semibold text-primary-900 mb-2">Primary Card</h5>
            <p className="text-primary-700">This card uses primary color variants.</p>
          </div>
          
          <div className="p-6 bg-secondary-50 border border-secondary-200 rounded-lg">
            <h5 className="font-semibold text-secondary-900 mb-2">Secondary Card</h5>
            <p className="text-secondary-700">This card uses secondary color variants.</p>
          </div>
          
          <div className="p-6 bg-accent-50 border border-accent-200 rounded-lg">
            <h5 className="font-semibold text-accent-900 mb-2">Accent Card</h5>
            <p className="text-accent-700">This card uses accent color variants.</p>
          </div>
        </div>

        <div className="flex gap-2">
          <span className="px-3 py-1 bg-success text-white rounded-full text-sm">Success</span>
          <span className="px-3 py-1 bg-warning text-white rounded-full text-sm">Warning</span>
          <span className="px-3 py-1 bg-error text-white rounded-full text-sm">Error</span>
          <span className="px-3 py-1 bg-info text-white rounded-full text-sm">Info</span>
        </div>
      </div>
    </div>
  )
}
```
Shows a live preview of the applied palette.

### 5. Create CSS for Tailwind v4
```css
/* app/globals.css */
@import "tailwindcss";

/* Define theme with dynamic color values */
@theme {
  /* Colors will be dynamically set via CSS custom properties */
  --color-primary-*: initial;
  --color-secondary-*: initial;
  --color-accent-*: initial;
  --color-neutral-*: initial;
  --color-success: initial;
  --color-warning: initial;
  --color-error: initial;
  --color-info: initial;
}

/* Smooth color transitions */
* {
  transition: background-color 0.3s ease, 
              color 0.3s ease, 
              border-color 0.3s ease;
}

/* Disable transitions during palette change */
.palette-transitioning * {
  transition: none !important;
}

/* Custom utilities for semantic colors */
@utility bg-success {
  background-color: var(--color-success);
}

@utility bg-warning {
  background-color: var(--color-warning);
}

@utility bg-error {
  background-color: var(--color-error);
}

@utility bg-info {
  background-color: var(--color-info);
}

@utility text-success {
  color: var(--color-success);
}

@utility text-warning {
  color: var(--color-warning);
}

@utility text-error {
  color: var(--color-error);
}

@utility text-info {
  color: var(--color-info);
}
```
Sets up Tailwind CSS v4 with dynamic palette support.

### 6. Integration example
```typescript
// app/page.tsx
import { PaletteSelector } from '@/components/palette-selector'
import { PalettePreview } from '@/components/palette-preview'

export default function PalettePage() {
  return (
    <div className="container mx-auto py-8 px-4">
      <h1 className="text-3xl font-bold mb-8">Theme Palette Selector</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div>
          <PaletteSelector />
        </div>
        
        <div>
          <PalettePreview />
        </div>
      </div>
    </div>
  )
}

// app/layout.tsx
import { PaletteProvider } from '@/components/palette-provider'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <PaletteProvider>
          {children}
        </PaletteProvider>
      </body>
    </html>
  )
}
```
Shows how to integrate palette system into a Next.js app.

## Validation
- All color variables are updated correctly
- Palette changes apply smoothly with transitions
- Variant modifications work as expected
- Scope limitations are respected
- Selection persists on reload

## Error Handling
- **"Palette not found"** - Check palette name in colorPalettes object
- **"Colors not applying"** - Verify CSS variable names and React context
- **"Invalid variant"** - Ensure variant is one of: standard, vivid, muted
- **"Hydration mismatch"** - Load saved palette after mount

## Safety Notes
- Test all palettes for accessibility compliance
- Ensure sufficient contrast for text/backgrounds
- Provide option to revert to default palette
- Consider color blindness when selecting palettes
- Preview before applying to production

## Examples
- **Quick palette switch**
  ```tsx
  import { usePalette } from '@/components/palette-provider'
  
  function QuickSwitch() {
    const { applyPalette } = usePalette()
    
    return (
      <div className="flex gap-2">
        <button onClick={() => applyPalette('ocean')}>Ocean</button>
        <button onClick={() => applyPalette('sunset')}>Sunset</button>
        <button onClick={() => applyPalette('forest')}>Forest</button>
      </div>
    )
  }
  ```

- **Programmatic palette application**
  ```tsx
  // Apply palette based on user preference
  useEffect(() => {
    const userPreference = getUserThemePreference()
    if (userPreference) {
      applyPalette(userPreference, 'standard', 'all')
    }
  }, [])
  ```

- **Seasonal themes**
  ```tsx
  const seasonalPalettes = {
    spring: 'pastel',
    summer: 'ocean',
    autumn: 'earth',
    winter: 'corporate'
  }
  
  const currentSeason = getCurrentSeason()
  applyPalette(seasonalPalettes[currentSeason])
  ```