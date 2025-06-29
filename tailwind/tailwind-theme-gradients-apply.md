# Tailwind Theme Gradients Apply

## Purpose
Apply gradient backgrounds to elements in your React/Next.js application using Tailwind CSS v4 gradient utilities and custom gradient presets

## Context
Use to add visually appealing gradient backgrounds to sections, buttons, cards, or any element. Supports linear, radial, and conic gradients with customizable directions, colors, and stops. Perfect for modern UI designs and hero sections. Built for Tailwind CSS v4 with React/Next.js.

## Parameters
- `$GRADIENT_TYPE` - Type of gradient to apply
  - Required
  - Options: `linear`, `radial`, `conic`, `preset`
- `$PRESET_NAME` - Name of preset gradient (if type is preset)
  - Optional
  - Options: `sunset`, `ocean`, `forest`, `fire`, `aurora`, `candy`, `metal`, `custom`
- `$DIRECTION` - Gradient direction
  - Optional
  - Default: `to-r` (to right)
  - Options: `to-t`, `to-tr`, `to-r`, `to-br`, `to-b`, `to-bl`, `to-l`, `to-tl`
- `$SELECTOR` - CSS selector for target elements
  - Optional
  - Default: `.gradient`
  - Example: `.hero`, `button`, `.card`

## Steps

### 1. Define gradient system with Tailwind CSS v4
```css
/* app/globals.css */
@import "tailwindcss";

/* Define gradient theme variables */
@theme {
  /* Gradient color stops */
  --gradient-from: var(--color-primary-500);
  --gradient-via: var(--color-secondary-500);
  --gradient-to: var(--color-accent-500);
  
  /* Gradient positions */
  --gradient-from-position: 0%;
  --gradient-via-position: 50%;
  --gradient-to-position: 100%;
}

/* Custom gradient utilities */
@utility gradient-radial {
  background-image: radial-gradient(
    ellipse at center,
    var(--gradient-from) var(--gradient-from-position),
    var(--gradient-via) var(--gradient-via-position),
    var(--gradient-to) var(--gradient-to-position)
  );
}

@utility gradient-conic {
  background-image: conic-gradient(
    from 180deg at 50% 50%,
    var(--gradient-from) var(--gradient-from-position),
    var(--gradient-via) var(--gradient-via-position),
    var(--gradient-to) var(--gradient-to-position)
  );
}

/* Animated gradient */
@utility gradient-animated {
  background-size: 200% 200%;
  animation: gradient-shift 3s ease infinite;
}

@keyframes gradient-shift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* Gradient text */
@utility gradient-text {
  background-image: linear-gradient(
    to right,
    var(--gradient-from),
    var(--gradient-via),
    var(--gradient-to)
  );
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}

/* Mesh gradient */
@utility gradient-mesh {
  background-image: 
    radial-gradient(at 20% 30%, var(--gradient-from) 0px, transparent 50%),
    radial-gradient(at 80% 20%, var(--gradient-via) 0px, transparent 50%),
    radial-gradient(at 40% 80%, var(--gradient-to) 0px, transparent 50%),
    radial-gradient(at 80% 70%, var(--gradient-from) 0px, transparent 50%);
}
```
Sets up comprehensive gradient system for Tailwind CSS v4.

### 2. Create gradient presets library
```typescript
// app/lib/gradient-presets.ts
export interface GradientPreset {
  name: string
  type: 'linear' | 'radial' | 'conic'
  direction?: string
  colors: {
    from: string
    via?: string
    to: string
  }
  positions?: {
    from?: string
    via?: string
    to?: string
  }
}

export const gradientPresets: Record<string, GradientPreset> = {
  sunset: {
    name: 'Sunset',
    type: 'linear',
    direction: 'to-r',
    colors: {
      from: '#f97316', // orange-500
      via: '#ef4444',  // red-500
      to: '#ec4899'    // pink-500
    }
  },
  
  ocean: {
    name: 'Ocean',
    type: 'linear',
    direction: 'to-br',
    colors: {
      from: '#3b82f6', // blue-500
      via: '#06b6d4',  // cyan-500
      to: '#14b8a6'    // teal-500
    }
  },
  
  forest: {
    name: 'Forest',
    type: 'linear',
    direction: 'to-b',
    colors: {
      from: '#22c55e', // green-500
      via: '#10b981',  // emerald-500
      to: '#14b8a6'    // teal-500
    }
  },
  
  fire: {
    name: 'Fire',
    type: 'linear',
    direction: 'to-t',
    colors: {
      from: '#facc15', // yellow-400
      via: '#f97316',  // orange-500
      to: '#dc2626'    // red-600
    }
  },
  
  aurora: {
    name: 'Aurora',
    type: 'linear',
    direction: 'to-tr',
    colors: {
      from: '#a855f7', // purple-500
      via: '#ec4899',  // pink-500
      to: '#6366f1'    // indigo-500
    }
  },
  
  candy: {
    name: 'Candy',
    type: 'linear',
    direction: 'to-r',
    colors: {
      from: '#f9a8d4', // pink-300
      via: '#c084fc',  // purple-400
      to: '#818cf8'    // indigo-400
    }
  },
  
  metal: {
    name: 'Metal',
    type: 'linear',
    direction: 'to-b',
    colors: {
      from: '#d1d5db', // gray-300
      via: '#9ca3af',  // gray-400
      to: '#4b5563'    // gray-600
    }
  },
  
  midnight: {
    name: 'Midnight',
    type: 'radial',
    colors: {
      from: '#1e293b', // slate-800
      via: '#0f172a',  // slate-900
      to: '#020617'    // slate-950
    }
  },
  
  rainbow: {
    name: 'Rainbow',
    type: 'conic',
    colors: {
      from: '#ef4444', // red
      via: '#3b82f6',  // blue
      to: '#ef4444'    // red (full circle)
    }
  }
}
```
Defines comprehensive gradient presets.

### 3. Create Gradient Provider component
```typescript
// app/components/gradient-provider.tsx
'use client'

import { createContext, useContext, useState, ReactNode } from 'react'
import { gradientPresets, GradientPreset } from '@/lib/gradient-presets'

interface GradientContextType {
  applyGradient: (
    element: HTMLElement | null,
    type: string,
    preset?: string,
    direction?: string
  ) => void
  applyGradientToClass: (
    className: string,
    type: string,
    preset?: string,
    direction?: string
  ) => void
  createGradientClass: (
    type: string,
    preset?: string,
    direction?: string
  ) => string
}

const GradientContext = createContext<GradientContextType | undefined>(undefined)

export function GradientProvider({ children }: { children: ReactNode }) {
  const createGradientClass = (
    type: string,
    preset?: string,
    direction: string = 'to-r'
  ): string => {
    if (type === 'preset' && preset && gradientPresets[preset]) {
      const presetConfig = gradientPresets[preset]
      const gradientDirection = presetConfig.direction || direction
      
      switch (presetConfig.type) {
        case 'radial':
          return 'gradient-radial'
        case 'conic':
          return 'gradient-conic'
        default:
          return `bg-gradient-${gradientDirection} from-[${presetConfig.colors.from}] ${
            presetConfig.colors.via ? `via-[${presetConfig.colors.via}]` : ''
          } to-[${presetConfig.colors.to}]`
      }
    }
    
    switch (type) {
      case 'radial':
        return 'gradient-radial'
      case 'conic':
        return 'gradient-conic'
      default:
        return `bg-gradient-${direction}`
    }
  }

  const applyGradient = (
    element: HTMLElement | null,
    type: string,
    preset?: string,
    direction?: string
  ) => {
    if (!element) return
    
    // Remove existing gradient classes
    element.className = element.className
      .split(' ')
      .filter(cls => 
        !cls.includes('bg-gradient') && 
        !cls.includes('gradient-') &&
        !cls.includes('from-') && 
        !cls.includes('via-') && 
        !cls.includes('to-')
      )
      .join(' ')
    
    // Add new gradient classes
    const gradientClass = createGradientClass(type, preset, direction)
    element.className += ` ${gradientClass}`
    
    // Apply custom CSS variables if using preset
    if (type === 'preset' && preset && gradientPresets[preset]) {
      const presetConfig = gradientPresets[preset]
      element.style.setProperty('--gradient-from', presetConfig.colors.from)
      if (presetConfig.colors.via) {
        element.style.setProperty('--gradient-via', presetConfig.colors.via)
      }
      element.style.setProperty('--gradient-to', presetConfig.colors.to)
    }
  }

  const applyGradientToClass = (
    className: string,
    type: string,
    preset?: string,
    direction?: string
  ) => {
    const elements = document.querySelectorAll(`.${className}`)
    elements.forEach(element => {
      applyGradient(element as HTMLElement, type, preset, direction)
    })
  }

  return (
    <GradientContext.Provider value={{
      applyGradient,
      applyGradientToClass,
      createGradientClass
    }}>
      {children}
    </GradientContext.Provider>
  )
}

export const useGradient = () => {
  const context = useContext(GradientContext)
  if (!context) {
    throw new Error('useGradient must be used within GradientProvider')
  }
  return context
}
```
Provides gradient management functionality.

### 4. Create Gradient component
```typescript
// app/components/gradient.tsx
'use client'

import { useGradient } from './gradient-provider'
import { ReactNode, HTMLAttributes } from 'react'

interface GradientProps extends HTMLAttributes<HTMLDivElement> {
  type?: 'linear' | 'radial' | 'conic' | 'preset'
  preset?: string
  direction?: string
  animated?: boolean
  children?: ReactNode
}

export function Gradient({
  type = 'linear',
  preset,
  direction = 'to-r',
  animated = false,
  children,
  className = '',
  ...props
}: GradientProps) {
  const { createGradientClass } = useGradient()
  
  const gradientClass = createGradientClass(type, preset, direction)
  const animatedClass = animated ? 'gradient-animated' : ''
  
  return (
    <div
      className={`${gradientClass} ${animatedClass} ${className}`}
      {...props}
    >
      {children}
    </div>
  )
}
```
Creates a reusable gradient component.

### 5. Create Gradient Text component
```typescript
// app/components/gradient-text.tsx
'use client'

import { ReactNode } from 'react'

interface GradientTextProps {
  children: ReactNode
  from?: string
  via?: string
  to?: string
  direction?: string
  className?: string
}

export function GradientText({
  children,
  from = '#3b82f6',
  via,
  to = '#ec4899',
  direction = 'to-r',
  className = ''
}: GradientTextProps) {
  const gradientStyle = {
    backgroundImage: `linear-gradient(${direction}, ${from}${via ? `, ${via}` : ''}, ${to})`,
    backgroundClip: 'text',
    WebkitBackgroundClip: 'text',
    color: 'transparent'
  }
  
  return (
    <span className={`inline-block ${className}`} style={gradientStyle}>
      {children}
    </span>
  )
}
```
Creates gradient text effects.

### 6. Create Gradient Builder component
```typescript
// app/components/gradient-builder.tsx
'use client'

import { useState } from 'react'
import { gradientPresets } from '@/lib/gradient-presets'
import { Gradient } from './gradient'

export function GradientBuilder() {
  const [gradientType, setGradientType] = useState<'linear' | 'radial' | 'conic' | 'preset'>('preset')
  const [selectedPreset, setSelectedPreset] = useState('sunset')
  const [direction, setDirection] = useState('to-r')
  const [animated, setAnimated] = useState(false)

  const directions = [
    { value: 'to-t', label: '↑' },
    { value: 'to-tr', label: '↗' },
    { value: 'to-r', label: '→' },
    { value: 'to-br', label: '↘' },
    { value: 'to-b', label: '↓' },
    { value: 'to-bl', label: '↙' },
    { value: 'to-l', label: '←' },
    { value: 'to-tl', label: '↖' }
  ]

  return (
    <div className="space-y-6">
      {/* Preview */}
      <div className="rounded-lg overflow-hidden shadow-lg">
        <Gradient
          type={gradientType}
          preset={gradientType === 'preset' ? selectedPreset : undefined}
          direction={direction}
          animated={animated}
          className="h-48 flex items-center justify-center"
        >
          <h3 className="text-2xl font-bold text-white drop-shadow-lg">
            Gradient Preview
          </h3>
        </Gradient>
      </div>

      {/* Controls */}
      <div className="space-y-4">
        {/* Gradient Type */}
        <div>
          <label className="text-sm font-medium mb-2 block">Gradient Type</label>
          <div className="flex gap-2">
            {(['linear', 'radial', 'conic', 'preset'] as const).map(type => (
              <button
                key={type}
                onClick={() => setGradientType(type)}
                className={`px-4 py-2 rounded-lg capitalize ${
                  gradientType === type
                    ? 'bg-primary-500 text-white'
                    : 'bg-neutral-100 hover:bg-neutral-200'
                }`}
              >
                {type}
              </button>
            ))}
          </div>
        </div>

        {/* Preset Selection */}
        {gradientType === 'preset' && (
          <div>
            <label className="text-sm font-medium mb-2 block">Preset</label>
            <div className="grid grid-cols-3 gap-2">
              {Object.entries(gradientPresets).map(([key, preset]) => (
                <button
                  key={key}
                  onClick={() => setSelectedPreset(key)}
                  className={`p-3 rounded-lg border-2 transition-all ${
                    selectedPreset === key
                      ? 'border-primary-500 shadow-md'
                      : 'border-neutral-200 hover:border-neutral-300'
                  }`}
                >
                  <div className="text-sm font-medium mb-1">{preset.name}</div>
                  <div
                    className="h-8 rounded"
                    style={{
                      background: `linear-gradient(to right, ${preset.colors.from}, ${
                        preset.colors.via || preset.colors.to
                      }, ${preset.colors.to})`
                    }}
                  />
                </button>
              ))}
            </div>
          </div>
        )}

        {/* Direction (for linear gradients) */}
        {(gradientType === 'linear' || (gradientType === 'preset' && gradientPresets[selectedPreset]?.type === 'linear')) && (
          <div>
            <label className="text-sm font-medium mb-2 block">Direction</label>
            <div className="grid grid-cols-4 gap-2">
              {directions.map(dir => (
                <button
                  key={dir.value}
                  onClick={() => setDirection(dir.value)}
                  className={`p-3 rounded-lg text-xl ${
                    direction === dir.value
                      ? 'bg-primary-500 text-white'
                      : 'bg-neutral-100 hover:bg-neutral-200'
                  }`}
                >
                  {dir.label}
                </button>
              ))}
            </div>
          </div>
        )}

        {/* Animation Toggle */}
        <div className="flex items-center gap-2">
          <input
            id="animated"
            type="checkbox"
            checked={animated}
            onChange={(e) => setAnimated(e.target.checked)}
            className="rounded"
          />
          <label htmlFor="animated" className="text-sm font-medium">
            Animate Gradient
          </label>
        </div>
      </div>

      {/* Code Preview */}
      <div className="bg-neutral-100 rounded-lg p-4">
        <h4 className="text-sm font-medium mb-2">React Code</h4>
        <pre className="text-sm overflow-x-auto">
          <code>{`<Gradient
  type="${gradientType}"${gradientType === 'preset' ? `\n  preset="${selectedPreset}"` : ''}
  direction="${direction}"${animated ? '\n  animated' : ''}
>
  Your content here
</Gradient>`}</code>
        </pre>
      </div>
    </div>
  )
}
```
Creates an interactive gradient builder.

### 7. Example usage components
```typescript
// app/components/gradient-examples.tsx
import { Gradient } from './gradient'
import { GradientText } from './gradient-text'

export function GradientExamples() {
  return (
    <div className="space-y-8">
      {/* Hero Section */}
      <Gradient type="preset" preset="aurora" className="rounded-lg p-12">
        <div className="text-center">
          <h1 className="text-4xl font-bold text-white mb-4">
            Beautiful Gradients
          </h1>
          <p className="text-white/90 text-lg">
            Create stunning gradient backgrounds with ease
          </p>
        </div>
      </Gradient>

      {/* Gradient Cards */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <Gradient type="preset" preset="ocean" className="rounded-lg p-6">
          <h3 className="text-xl font-semibold text-white mb-2">Ocean</h3>
          <p className="text-white/80">Calming blue gradient</p>
        </Gradient>
        
        <Gradient type="preset" preset="sunset" className="rounded-lg p-6">
          <h3 className="text-xl font-semibold text-white mb-2">Sunset</h3>
          <p className="text-white/80">Warm sunset colors</p>
        </Gradient>
        
        <Gradient type="preset" preset="forest" className="rounded-lg p-6">
          <h3 className="text-xl font-semibold text-white mb-2">Forest</h3>
          <p className="text-white/80">Natural green tones</p>
        </Gradient>
      </div>

      {/* Gradient Buttons */}
      <div className="flex gap-4 flex-wrap">
        <button className="px-6 py-3 rounded-lg font-medium text-white bg-gradient-to-r from-blue-500 via-blue-600 to-blue-700 hover:shadow-lg transition-shadow">
          Primary Action
        </button>
        
        <button className="px-6 py-3 rounded-lg font-medium text-white bg-gradient-to-r from-purple-500 via-pink-500 to-red-500 hover:shadow-lg transition-shadow">
          Secondary Action
        </button>
        
        <button className="px-6 py-3 rounded-lg font-medium gradient-animated bg-gradient-to-r from-green-400 via-blue-500 to-purple-600 text-white">
          Animated Button
        </button>
      </div>

      {/* Gradient Text */}
      <div className="space-y-4">
        <h2 className="text-3xl font-bold">
          <GradientText from="#3b82f6" to="#ec4899">
            Gradient Text Example
          </GradientText>
        </h2>
        
        <p className="text-lg">
          Create beautiful{' '}
          <GradientText from="#10b981" via="#3b82f6" to="#8b5cf6" className="font-semibold">
            gradient text effects
          </GradientText>{' '}
          with custom colors
        </p>
      </div>

      {/* Mesh Gradient */}
      <div className="gradient-mesh rounded-lg p-12 text-center">
        <h3 className="text-2xl font-bold mb-2">Mesh Gradient</h3>
        <p className="text-neutral-600">
          Complex multi-point gradient backgrounds
        </p>
      </div>
    </div>
  )
}
```
Shows various gradient implementations.

### 8. Integration with layout
```typescript
// app/layout.tsx
import { GradientProvider } from '@/components/gradient-provider'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <GradientProvider>
          {children}
        </GradientProvider>
      </body>
    </html>
  )
}

// app/page.tsx
import { GradientBuilder } from '@/components/gradient-builder'
import { GradientExamples } from '@/components/gradient-examples'

export default function GradientPage() {
  return (
    <div className="container mx-auto py-8 px-4">
      <h1 className="text-3xl font-bold mb-8">Gradient System</h1>
      
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-12">
        <div>
          <h2 className="text-2xl font-semibold mb-4">Gradient Builder</h2>
          <GradientBuilder />
        </div>
        
        <div>
          <h2 className="text-2xl font-semibold mb-4">Live Examples</h2>
          <GradientExamples />
        </div>
      </div>
    </div>
  )
}
```
Shows complete integration example.

## Validation
- Gradient classes are applied correctly
- Preset gradients render as expected
- Animation works smoothly
- Custom gradients display properly
- Text gradients are visible

## Error Handling
- **"Gradient not showing"** - Check for conflicting background classes
- **"Invalid preset"** - Verify preset name in gradientPresets
- **"Animation not working"** - Ensure gradient-animated class is applied
- **"Text gradient invisible"** - Check text color and background-clip

## Safety Notes
- Test gradients on different screen sizes
- Ensure text remains readable on gradients
- Consider performance with animated gradients
- Provide fallback solid colors
- Test in different browsers for compatibility

## Examples
- **Quick gradient application**
  ```tsx
  <Gradient type="preset" preset="sunset">
    <h1>Sunset Gradient Background</h1>
  </Gradient>
  ```

- **Custom gradient**
  ```tsx
  <div className="bg-gradient-to-br from-indigo-500 via-purple-500 to-pink-500 p-8">
    Custom gradient content
  </div>
  ```

- **Animated gradient button**
  ```tsx
  <button className="gradient-animated bg-gradient-to-r from-blue-500 to-purple-600 text-white px-6 py-3 rounded-lg">
    Animated Button
  </button>
  ```