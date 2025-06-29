# Tailwind Spacing Padding

## Purpose
Dynamically adjust padding values across your React/Next.js application using Tailwind CSS v4's spacing system

## Context
Use to create consistent padding throughout your application, implement responsive padding scales, or adjust spacing for different themes or user preferences. Works with Tailwind CSS v4's new CSS-first configuration and React components.

## Parameters
- `$PADDING_SCALE` - Scale factor for padding
  - Required
  - Options: `compact`, `default`, `comfortable`, `spacious`, `custom`
- `$BASE_UNIT` - Base spacing unit
  - Optional
  - Default: `0.25rem` (4px)
  - Example: `0.25rem`, `4px`, `0.5rem`
- `$RESPONSIVE` - Apply responsive scaling
  - Optional
  - Default: `true`
  - Options: `true`, `false`
- `$SCOPE` - Component scope for padding
  - Optional
  - Default: `global`
  - Example: `global`, `cards`, `forms`, `buttons`

## Steps

### 1. Configure Tailwind CSS v4 spacing theme
```css
/* app/globals.css */
@import "tailwindcss";

/* Define custom spacing scale using CSS variables */
@theme {
  /* Base spacing units */
  --spacing-base: 0.25rem; /* 4px */
  --spacing-scale: 1;
  
  /* Generate spacing scale */
  --spacing-0: 0;
  --spacing-px: 1px;
  --spacing-0_5: calc(var(--spacing-base) * 0.5 * var(--spacing-scale));
  --spacing-1: calc(var(--spacing-base) * 1 * var(--spacing-scale));
  --spacing-1_5: calc(var(--spacing-base) * 1.5 * var(--spacing-scale));
  --spacing-2: calc(var(--spacing-base) * 2 * var(--spacing-scale));
  --spacing-2_5: calc(var(--spacing-base) * 2.5 * var(--spacing-scale));
  --spacing-3: calc(var(--spacing-base) * 3 * var(--spacing-scale));
  --spacing-3_5: calc(var(--spacing-base) * 3.5 * var(--spacing-scale));
  --spacing-4: calc(var(--spacing-base) * 4 * var(--spacing-scale));
  --spacing-5: calc(var(--spacing-base) * 5 * var(--spacing-scale));
  --spacing-6: calc(var(--spacing-base) * 6 * var(--spacing-scale));
  --spacing-7: calc(var(--spacing-base) * 7 * var(--spacing-scale));
  --spacing-8: calc(var(--spacing-base) * 8 * var(--spacing-scale));
  --spacing-9: calc(var(--spacing-base) * 9 * var(--spacing-scale));
  --spacing-10: calc(var(--spacing-base) * 10 * var(--spacing-scale));
  --spacing-11: calc(var(--spacing-base) * 11 * var(--spacing-scale));
  --spacing-12: calc(var(--spacing-base) * 12 * var(--spacing-scale));
  --spacing-14: calc(var(--spacing-base) * 14 * var(--spacing-scale));
  --spacing-16: calc(var(--spacing-base) * 16 * var(--spacing-scale));
  --spacing-20: calc(var(--spacing-base) * 20 * var(--spacing-scale));
  --spacing-24: calc(var(--spacing-base) * 24 * var(--spacing-scale));
  --spacing-28: calc(var(--spacing-base) * 28 * var(--spacing-scale));
  --spacing-32: calc(var(--spacing-base) * 32 * var(--spacing-scale));
  --spacing-36: calc(var(--spacing-base) * 36 * var(--spacing-scale));
  --spacing-40: calc(var(--spacing-base) * 40 * var(--spacing-scale));
  --spacing-44: calc(var(--spacing-base) * 44 * var(--spacing-scale));
  --spacing-48: calc(var(--spacing-base) * 48 * var(--spacing-scale));
  --spacing-52: calc(var(--spacing-base) * 52 * var(--spacing-scale));
  --spacing-56: calc(var(--spacing-base) * 56 * var(--spacing-scale));
  --spacing-60: calc(var(--spacing-base) * 60 * var(--spacing-scale));
  --spacing-64: calc(var(--spacing-base) * 64 * var(--spacing-scale));
  --spacing-72: calc(var(--spacing-base) * 72 * var(--spacing-scale));
  --spacing-80: calc(var(--spacing-base) * 80 * var(--spacing-scale));
  --spacing-96: calc(var(--spacing-base) * 96 * var(--spacing-scale));
}
```
Configures Tailwind CSS v4 spacing theme with dynamic scaling.

### 2. Create padding context and hook
```typescript
// app/contexts/PaddingContext.tsx
'use client';

import React, { createContext, useContext, useState, useEffect } from 'react';

interface PaddingScale {
  name: string;
  scale: number;
  description: string;
}

interface PaddingContextValue {
  scale: PaddingScale;
  setScale: (scale: PaddingScale) => void;
  baseUnit: string;
  setBaseUnit: (unit: string) => void;
  responsive: boolean;
  setResponsive: (responsive: boolean) => void;
}

const paddingScales: Record<string, PaddingScale> = {
  compact: { name: 'compact', scale: 0.75, description: 'Reduced padding for dense layouts' },
  default: { name: 'default', scale: 1, description: 'Standard Tailwind spacing' },
  comfortable: { name: 'comfortable', scale: 1.25, description: 'Increased padding for touch' },
  spacious: { name: 'spacious', scale: 1.5, description: 'Extra padding for readability' },
};

const PaddingContext = createContext<PaddingContextValue | undefined>(undefined);

export function PaddingProvider({ children }: { children: React.ReactNode }) {
  const [scale, setScale] = useState<PaddingScale>(paddingScales.default);
  const [baseUnit, setBaseUnit] = useState('0.25rem');
  const [responsive, setResponsive] = useState(true);

  useEffect(() => {
    // Load saved preferences
    const saved = localStorage.getItem('padding-preferences');
    if (saved) {
      const prefs = JSON.parse(saved);
      if (prefs.scale && paddingScales[prefs.scale]) {
        setScale(paddingScales[prefs.scale]);
      }
      if (prefs.baseUnit) setBaseUnit(prefs.baseUnit);
      if (prefs.responsive !== undefined) setResponsive(prefs.responsive);
    }
  }, []);

  useEffect(() => {
    // Apply scale to CSS variables
    const root = document.documentElement;
    root.style.setProperty('--spacing-scale', scale.scale.toString());
    root.style.setProperty('--spacing-base', baseUnit);

    // Apply responsive scaling
    if (responsive) {
      const applyResponsiveScale = () => {
        const width = window.innerWidth;
        let responsiveScale = scale.scale;
        
        if (width < 640) responsiveScale *= 0.875; // sm
        else if (width >= 1536) responsiveScale *= 1.125; // 2xl
        
        root.style.setProperty('--spacing-scale', responsiveScale.toString());
      };

      applyResponsiveScale();
      window.addEventListener('resize', applyResponsiveScale);
      return () => window.removeEventListener('resize', applyResponsiveScale);
    }
  }, [scale, baseUnit, responsive]);

  const updateScale = (newScale: PaddingScale) => {
    setScale(newScale);
    localStorage.setItem('padding-preferences', JSON.stringify({
      scale: newScale.name,
      baseUnit,
      responsive
    }));
  };

  return (
    <PaddingContext.Provider value={{
      scale,
      setScale: updateScale,
      baseUnit,
      setBaseUnit,
      responsive,
      setResponsive
    }}>
      {children}
    </PaddingContext.Provider>
  );
}

export function usePadding() {
  const context = useContext(PaddingContext);
  if (!context) {
    throw new Error('usePadding must be used within PaddingProvider');
  }
  return context;
}
```
Creates React context for managing padding scale dynamically.

### 3. Create padding components
```typescript
// app/components/padding/PaddingBox.tsx
'use client';

import React from 'react';
import { cn } from '@/lib/utils';

interface PaddingBoxProps {
  children: React.ReactNode;
  padding?: string | { top?: string; right?: string; bottom?: string; left?: string; x?: string; y?: string };
  className?: string;
  responsive?: boolean;
}

export function PaddingBox({ 
  children, 
  padding = '4',
  className,
  responsive = true,
  ...props 
}: PaddingBoxProps) {
  const getPaddingClasses = () => {
    if (typeof padding === 'string') {
      return responsive 
        ? `p-${padding} sm:p-${parseInt(padding) * 1.25} lg:p-${parseInt(padding) * 1.5}`
        : `p-${padding}`;
    }
    
    const classes = [];
    if (padding.top) classes.push(`pt-${padding.top}`);
    if (padding.right) classes.push(`pr-${padding.right}`);
    if (padding.bottom) classes.push(`pb-${padding.bottom}`);
    if (padding.left) classes.push(`pl-${padding.left}`);
    if (padding.x) classes.push(`px-${padding.x}`);
    if (padding.y) classes.push(`py-${padding.y}`);
    
    return classes.join(' ');
  };

  return (
    <div className={cn(getPaddingClasses(), className)} {...props}>
      {children}
    </div>
  );
}

// Section component with consistent padding
export function Section({ 
  children, 
  size = 'default',
  className,
  ...props 
}: {
  children: React.ReactNode;
  size?: 'sm' | 'default' | 'lg' | 'xl';
  className?: string;
}) {
  const sizeMap = {
    sm: 'py-8 px-4 sm:py-12 sm:px-6',
    default: 'py-12 px-4 sm:py-16 sm:px-6 lg:py-20 lg:px-8',
    lg: 'py-16 px-4 sm:py-20 sm:px-6 lg:py-24 lg:px-8',
    xl: 'py-20 px-4 sm:py-24 sm:px-6 lg:py-32 lg:px-8'
  };

  return (
    <section className={cn(sizeMap[size], className)} {...props}>
      {children}
    </section>
  );
}

// Container with padding
export function PaddedContainer({ 
  children, 
  maxWidth = 'lg',
  className,
  ...props 
}: {
  children: React.ReactNode;
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | '2xl' | 'full';
  className?: string;
}) {
  const maxWidthMap = {
    sm: 'max-w-sm',
    md: 'max-w-md',
    lg: 'max-w-lg',
    xl: 'max-w-xl',
    '2xl': 'max-w-2xl',
    full: 'max-w-full'
  };

  return (
    <div className={cn(
      'mx-auto px-4 sm:px-6 lg:px-8',
      maxWidthMap[maxWidth],
      className
    )} {...props}>
      {children}
    </div>
  );
}
```
Creates reusable padding components with responsive support.

### 4. Create padding configurator
```typescript
// app/components/padding/PaddingConfigurator.tsx
'use client';

import React, { useState } from 'react';
import { usePadding } from '@/contexts/PaddingContext';

export function PaddingConfigurator() {
  const { scale, setScale, baseUnit, setBaseUnit, responsive, setResponsive } = usePadding();
  const [isOpen, setIsOpen] = useState(false);

  const paddingScales = [
    { name: 'compact', scale: 0.75, description: 'Dense layouts' },
    { name: 'default', scale: 1, description: 'Standard spacing' },
    { name: 'comfortable', scale: 1.25, description: 'Touch-friendly' },
    { name: 'spacious', scale: 1.5, description: 'Maximum readability' },
  ];

  if (process.env.NODE_ENV !== 'development') return null;

  return (
    <>
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="fixed bottom-4 right-4 bg-blue-500 text-white p-3 rounded-full shadow-lg z-50 hover:bg-blue-600"
        aria-label="Configure padding"
      >
        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4" />
        </svg>
      </button>

      {isOpen && (
        <div className="fixed bottom-20 right-4 bg-white dark:bg-gray-800 rounded-lg shadow-xl p-6 z-50 w-80">
          <h3 className="text-lg font-semibold mb-4">Padding Configuration</h3>
          
          <div className="space-y-4">
            {/* Scale selector */}
            <div>
              <label className="block text-sm font-medium mb-2">Padding Scale</label>
              <div className="space-y-2">
                {paddingScales.map((s) => (
                  <button
                    key={s.name}
                    onClick={() => setScale(s)}
                    className={cn(
                      'w-full text-left p-3 rounded-lg border transition-colors',
                      scale.name === s.name
                        ? 'border-blue-500 bg-blue-50 dark:bg-blue-900/20'
                        : 'border-gray-200 hover:border-gray-300 dark:border-gray-700'
                    )}
                  >
                    <div className="font-medium">{s.name}</div>
                    <div className="text-sm text-gray-600 dark:text-gray-400">{s.description}</div>
                    <div className="text-xs text-gray-500 mt-1">Scale: {s.scale}x</div>
                  </button>
                ))}
              </div>
            </div>

            {/* Base unit */}
            <div>
              <label className="block text-sm font-medium mb-2">
                Base Unit: {baseUnit}
              </label>
              <input
                type="range"
                min="0.125"
                max="0.5"
                step="0.0625"
                value={parseFloat(baseUnit)}
                onChange={(e) => setBaseUnit(`${e.target.value}rem`)}
                className="w-full"
              />
            </div>

            {/* Responsive toggle */}
            <div>
              <label className="flex items-center">
                <input
                  type="checkbox"
                  checked={responsive}
                  onChange={(e) => setResponsive(e.target.checked)}
                  className="mr-2"
                />
                <span className="text-sm font-medium">Responsive scaling</span>
              </label>
              <p className="text-xs text-gray-600 dark:text-gray-400 mt-1">
                Automatically adjust padding based on viewport size
              </p>
            </div>

            {/* Preview */}
            <div className="pt-4 border-t">
              <p className="text-sm font-medium mb-2">Preview</p>
              <div className="space-y-2">
                <div className="p-2 bg-blue-100 dark:bg-blue-900/30 rounded">p-2</div>
                <div className="p-4 bg-blue-100 dark:bg-blue-900/30 rounded">p-4</div>
                <div className="p-8 bg-blue-100 dark:bg-blue-900/30 rounded">p-8</div>
              </div>
            </div>
          </div>
        </div>
      )}
    </>
  );
}
```
Creates interactive padding configurator for development.

### 5. Create padding utilities
```typescript
// app/components/padding/PaddingUtilities.tsx
'use client';

import React from 'react';

// Responsive padding component
export function ResponsivePadding({ 
  children,
  base = '4',
  sm = null,
  md = null,
  lg = null,
  xl = null,
  className = ''
}: {
  children: React.ReactNode;
  base?: string;
  sm?: string | null;
  md?: string | null;
  lg?: string | null;
  xl?: string | null;
  className?: string;
}) {
  const classes = [
    `p-${base}`,
    sm && `sm:p-${sm}`,
    md && `md:p-${md}`,
    lg && `lg:p-${lg}`,
    xl && `xl:p-${xl}`,
    className
  ].filter(Boolean).join(' ');

  return <div className={classes}>{children}</div>;
}

// Directional padding component
export function DirectionalPadding({ 
  children,
  top = '0',
  right = '0',
  bottom = '0',
  left = '0',
  className = ''
}: {
  children: React.ReactNode;
  top?: string;
  right?: string;
  bottom?: string;
  left?: string;
  className?: string;
}) {
  return (
    <div className={`pt-${top} pr-${right} pb-${bottom} pl-${left} ${className}`}>
      {children}
    </div>
  );
}

// Padding preset patterns
export const paddingPresets = {
  card: 'p-4 sm:p-6',
  section: 'py-12 px-4 sm:py-16 sm:px-6 lg:py-20 lg:px-8',
  compact: 'p-2 sm:p-3',
  comfortable: 'p-6 sm:p-8',
  hero: 'py-16 px-4 sm:py-24 sm:px-6 lg:py-32 lg:px-8',
  form: 'p-4 sm:p-6 lg:p-8',
  modal: 'p-6 sm:p-8',
  button: 'py-2 px-4',
  'button-sm': 'py-1 px-3',
  'button-lg': 'py-3 px-6',
};

// Padding preset component
export function PaddingPreset({ 
  preset,
  children,
  className = ''
}: {
  preset: keyof typeof paddingPresets;
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <div className={`${paddingPresets[preset]} ${className}`}>
      {children}
    </div>
  );
}
```
Creates utility components for common padding patterns.

### 6. Create padding hooks
```typescript
// app/hooks/useDynamicPadding.ts
import { useState, useEffect } from 'react';
import { usePadding } from '@/contexts/PaddingContext';

export function useDynamicPadding(basePadding: number = 4) {
  const { scale } = usePadding();
  const [padding, setPadding] = useState(basePadding);

  useEffect(() => {
    setPadding(Math.round(basePadding * scale.scale));
  }, [basePadding, scale]);

  return {
    padding,
    paddingClass: `p-${padding}`,
    paddingX: `px-${padding}`,
    paddingY: `py-${padding}`,
    paddingTop: `pt-${padding}`,
    paddingRight: `pr-${padding}`,
    paddingBottom: `pb-${padding}`,
    paddingLeft: `pl-${padding}`,
  };
}

// Hook for responsive padding
export function useResponsivePadding(config: {
  base: number;
  sm?: number;
  md?: number;
  lg?: number;
  xl?: number;
}) {
  const [paddingClass, setPaddingClass] = useState('');

  useEffect(() => {
    const classes = [
      `p-${config.base}`,
      config.sm && `sm:p-${config.sm}`,
      config.md && `md:p-${config.md}`,
      config.lg && `lg:p-${config.lg}`,
      config.xl && `xl:p-${config.xl}`,
    ].filter(Boolean).join(' ');

    setPaddingClass(classes);
  }, [config]);

  return paddingClass;
}
```
Creates custom hooks for dynamic padding management.

### 7. Add CSS utilities for padding
```css
/* Additional CSS for custom padding utilities */
@layer utilities {
  /* Fluid padding using clamp */
  .p-fluid-sm {
    padding: clamp(0.5rem, 2vw, 1rem);
  }
  
  .p-fluid-md {
    padding: clamp(1rem, 3vw, 2rem);
  }
  
  .p-fluid-lg {
    padding: clamp(2rem, 5vw, 4rem);
  }
  
  /* Logical properties */
  .p-block-4 {
    padding-block: var(--spacing-4);
  }
  
  .p-inline-4 {
    padding-inline: var(--spacing-4);
  }
  
  /* Custom padding patterns */
  .p-safe {
    padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
  }
}
```
Adds additional CSS utilities for advanced padding.

### 8. Complete setup example
```typescript
// app/layout.tsx
import './globals.css';
import { PaddingProvider } from '@/contexts/PaddingContext';
import { PaddingConfigurator } from '@/components/padding/PaddingConfigurator';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <PaddingProvider>
          {children}
          <PaddingConfigurator />
        </PaddingProvider>
      </body>
    </html>
  );
}

// app/page.tsx
import { Section, PaddedContainer, PaddingBox } from '@/components/padding';

export default function Home() {
  return (
    <>
      <Section size="lg">
        <PaddedContainer maxWidth="xl">
          <h1 className="text-4xl font-bold mb-4">Dynamic Padding System</h1>
          <p>Padding adjusts based on user preferences and viewport size.</p>
        </PaddedContainer>
      </Section>

      <Section>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <PaddingBox padding="6" className="bg-white shadow rounded-lg">
            <h2 className="text-xl font-semibold">Card 1</h2>
            <p>Content with dynamic padding</p>
          </PaddingBox>
          
          <PaddingBox padding={{ x: '6', y: '8' }} className="bg-white shadow rounded-lg">
            <h2 className="text-xl font-semibold">Card 2</h2>
            <p>Different padding values</p>
          </PaddingBox>
          
          <PaddingBox padding="6" responsive className="bg-white shadow rounded-lg">
            <h2 className="text-xl font-semibold">Card 3</h2>
            <p>Responsive padding enabled</p>
          </PaddingBox>
        </div>
      </Section>
    </>
  );
}
```
Complete implementation example with Next.js.

## Validation
- Padding scales apply correctly across components
- Responsive padding adjusts at breakpoints
- CSS variables update dynamically
- LocalStorage persistence works
- TypeScript types are correct

## Error Handling
- **"Padding not updating"** - Check PaddingProvider is wrapping app
- **"Invalid padding value"** - Ensure using valid Tailwind spacing values
- **"Hydration error"** - Check client-only components use 'use client'
- **"Types error"** - Install @types/react and configure tsconfig.json

## Safety Notes
- Test with different viewport sizes and devices
- Ensure padding doesn't break layouts
- Consider accessibility with sufficient touch targets
- Test with RTL languages
- Monitor performance with many dynamic updates

## Examples
- **Apply consistent padding**
  ```jsx
  <PaddingBox padding="6">
    <Card />
  </PaddingBox>
  ```
  
- **Responsive section padding**
  ```jsx
  <Section size="lg">
    <Content />
  </Section>
  ```

- **Custom padding configuration**
  ```jsx
  <PaddingBox 
    padding={{ top: '8', x: '6', bottom: '4' }}
    responsive
  >
    <CustomComponent />
  </PaddingBox>
  ```

- **Use padding hook**
  ```jsx
  function MyComponent() {
    const { paddingClass } = useDynamicPadding(4);
    return <div className={paddingClass}>Content</div>;
  }
  ```