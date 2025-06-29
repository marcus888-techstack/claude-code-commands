# Tailwind Spacing Margin

## Purpose
Dynamically adjust margin values and implement advanced spacing patterns in React/Next.js applications using Tailwind CSS v4

## Context
Use to create consistent margin spacing, implement layout rhythms, manage component separation, and handle complex margin scenarios like margin collapse. Built for Tailwind CSS v4's CSS-first approach with React components.

## Parameters
- `$MARGIN_SCALE` - Scale factor for margins
  - Required
  - Options: `tight`, `default`, `relaxed`, `loose`, `custom`
- `$RHYTHM` - Vertical rhythm multiplier
  - Optional
  - Default: `1.5`
  - Example: `1.5`, `1.618`, `2`
- `$COLLAPSE_BEHAVIOR` - How to handle margin collapse
  - Optional
  - Default: `smart`
  - Options: `allow`, `prevent`, `smart`
- `$FLOW_DIRECTION` - Primary flow direction
  - Optional
  - Default: `vertical`
  - Options: `vertical`, `horizontal`, `both`

## Steps

### 1. Configure Tailwind CSS v4 margin theme
```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  /* Margin scale variables */
  --margin-scale: 1;
  --margin-rhythm: 1.5;
  
  /* Negative margin support */
  --margin-negative-multiplier: -1;
  
  /* Flow spacing for consistent rhythm */
  --flow-space-xs: calc(0.5rem * var(--margin-scale));
  --flow-space-sm: calc(0.75rem * var(--margin-scale));
  --flow-space-md: calc(1.5rem * var(--margin-scale));
  --flow-space-lg: calc(2rem * var(--margin-scale));
  --flow-space-xl: calc(3rem * var(--margin-scale));
  --flow-space-2xl: calc(4rem * var(--margin-scale));
}

/* Custom margin utilities */
@layer utilities {
  /* Flow utilities for consistent spacing */
  .flow > * + * {
    margin-top: var(--flow-space-md);
  }
  
  .flow-xs > * + * {
    margin-top: var(--flow-space-xs);
  }
  
  .flow-sm > * + * {
    margin-top: var(--flow-space-sm);
  }
  
  .flow-lg > * + * {
    margin-top: var(--flow-space-lg);
  }
  
  .flow-xl > * + * {
    margin-top: var(--flow-space-xl);
  }
  
  /* Horizontal flow */
  .flow-horizontal > * + * {
    margin-left: var(--flow-space-md);
  }
  
  /* Margin trim for container edges */
  .margin-trim {
    margin-trim: block;
  }
  
  /* Auto margins for centering */
  .mx-auto {
    margin-inline: auto;
  }
  
  .my-auto {
    margin-block: auto;
  }
}
```
Configures margin theme with flow utilities.

### 2. Create margin context and provider
```typescript
// app/contexts/MarginContext.tsx
'use client';

import React, { createContext, useContext, useState, useEffect } from 'react';

interface MarginScale {
  name: string;
  scale: number;
  rhythm: number;
  description: string;
}

interface MarginContextValue {
  scale: MarginScale;
  setScale: (scale: MarginScale) => void;
  collapseMode: 'allow' | 'prevent' | 'smart';
  setCollapseMode: (mode: 'allow' | 'prevent' | 'smart') => void;
  flowDirection: 'vertical' | 'horizontal' | 'both';
  setFlowDirection: (direction: 'vertical' | 'horizontal' | 'both') => void;
}

const marginScales: Record<string, MarginScale> = {
  tight: { 
    name: 'tight', 
    scale: 0.75, 
    rhythm: 1.25,
    description: 'Compact spacing for dense interfaces' 
  },
  default: { 
    name: 'default', 
    scale: 1, 
    rhythm: 1.5,
    description: 'Standard spacing with good rhythm' 
  },
  relaxed: { 
    name: 'relaxed', 
    scale: 1.25, 
    rhythm: 1.618,
    description: 'Comfortable spacing with golden ratio' 
  },
  loose: { 
    name: 'loose', 
    scale: 1.5, 
    rhythm: 2,
    description: 'Generous spacing for readability' 
  },
};

const MarginContext = createContext<MarginContextValue | undefined>(undefined);

export function MarginProvider({ children }: { children: React.ReactNode }) {
  const [scale, setScale] = useState<MarginScale>(marginScales.default);
  const [collapseMode, setCollapseMode] = useState<'allow' | 'prevent' | 'smart'>('smart');
  const [flowDirection, setFlowDirection] = useState<'vertical' | 'horizontal' | 'both'>('vertical');

  useEffect(() => {
    // Load saved preferences
    const saved = localStorage.getItem('margin-preferences');
    if (saved) {
      const prefs = JSON.parse(saved);
      if (prefs.scale && marginScales[prefs.scale]) {
        setScale(marginScales[prefs.scale]);
      }
      if (prefs.collapseMode) setCollapseMode(prefs.collapseMode);
      if (prefs.flowDirection) setFlowDirection(prefs.flowDirection);
    }
  }, []);

  useEffect(() => {
    // Apply scale to CSS variables
    const root = document.documentElement;
    root.style.setProperty('--margin-scale', scale.scale.toString());
    root.style.setProperty('--margin-rhythm', scale.rhythm.toString());

    // Save preferences
    localStorage.setItem('margin-preferences', JSON.stringify({
      scale: scale.name,
      collapseMode,
      flowDirection
    }));
  }, [scale, collapseMode, flowDirection]);

  return (
    <MarginContext.Provider value={{
      scale,
      setScale,
      collapseMode,
      setCollapseMode,
      flowDirection,
      setFlowDirection
    }}>
      {children}
    </MarginContext.Provider>
  );
}

export function useMargin() {
  const context = useContext(MarginContext);
  if (!context) {
    throw new Error('useMargin must be used within MarginProvider');
  }
  return context;
}
```
Creates context for managing margin scales and behavior.

### 3. Create margin components
```typescript
// app/components/margin/MarginComponents.tsx
'use client';

import React from 'react';
import { cn } from '@/lib/utils';
import { useMargin } from '@/contexts/MarginContext';

// Flow component for consistent spacing
export function Flow({ 
  children, 
  space = 'md',
  direction = 'vertical',
  className,
  ...props 
}: {
  children: React.ReactNode;
  space?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl';
  direction?: 'vertical' | 'horizontal';
  className?: string;
}) {
  const flowClass = direction === 'horizontal' 
    ? `flow-horizontal-${space}`
    : `flow-${space}`;
    
  return (
    <div className={cn(flowClass, className)} {...props}>
      {children}
    </div>
  );
}

// Stack component with margin collapse control
export function Stack({ 
  children, 
  gap = '4',
  direction = 'vertical',
  collapse = 'smart',
  className,
  ...props 
}: {
  children: React.ReactNode;
  gap?: string;
  direction?: 'vertical' | 'horizontal';
  collapse?: 'allow' | 'prevent' | 'smart';
  className?: string;
}) {
  const { collapseMode } = useMargin();
  const shouldPreventCollapse = collapse === 'prevent' || 
    (collapse === 'smart' && collapseMode === 'prevent');

  const stackStyles = shouldPreventCollapse ? {
    display: 'flex',
    flexDirection: direction === 'horizontal' ? 'row' : 'column',
    gap: `var(--spacing-${gap})`
  } : {};

  const marginClass = !shouldPreventCollapse && direction === 'vertical' 
    ? `space-y-${gap}`
    : direction === 'horizontal' 
    ? `space-x-${gap}`
    : '';

  return (
    <div 
      className={cn(marginClass, className)} 
      style={stackStyles}
      {...props}
    >
      {children}
    </div>
  );
}

// Spacer component for explicit spacing
export function Spacer({ 
  size = '4',
  axis = 'vertical',
  className 
}: {
  size?: string;
  axis?: 'vertical' | 'horizontal' | 'both';
  className?: string;
}) {
  const classes = {
    vertical: `h-${size}`,
    horizontal: `w-${size}`,
    both: `w-${size} h-${size}`
  };

  return <div className={cn(classes[axis], className)} aria-hidden="true" />;
}

// Center component with auto margins
export function Center({ 
  children, 
  axis = 'horizontal',
  maxWidth,
  className,
  ...props 
}: {
  children: React.ReactNode;
  axis?: 'horizontal' | 'vertical' | 'both';
  maxWidth?: string;
  className?: string;
}) {
  const centerClasses = {
    horizontal: 'mx-auto',
    vertical: 'my-auto',
    both: 'mx-auto my-auto'
  };

  return (
    <div 
      className={cn(
        centerClasses[axis],
        maxWidth && `max-w-${maxWidth}`,
        className
      )} 
      {...props}
    >
      {children}
    </div>
  );
}

// Responsive margin component
export function ResponsiveMargin({ 
  children,
  base = '0',
  sm,
  md,
  lg,
  xl,
  className = ''
}: {
  children: React.ReactNode;
  base?: string;
  sm?: string;
  md?: string;
  lg?: string;
  xl?: string;
  className?: string;
}) {
  const classes = [
    `m-${base}`,
    sm && `sm:m-${sm}`,
    md && `md:m-${md}`,
    lg && `lg:m-${lg}`,
    xl && `xl:m-${xl}`
  ].filter(Boolean).join(' ');

  return <div className={cn(classes, className)}>{children}</div>;
}
```
Creates margin components for various spacing patterns.

### 4. Create margin utilities and hooks
```typescript
// app/hooks/useMarginUtilities.ts
import { useMemo } from 'react';
import { useMargin } from '@/contexts/MarginContext';

export function useMarginClasses(baseMargin: number = 4) {
  const { scale } = useMargin();
  
  return useMemo(() => {
    const scaledMargin = Math.round(baseMargin * scale.scale);
    
    return {
      margin: `m-${scaledMargin}`,
      marginX: `mx-${scaledMargin}`,
      marginY: `my-${scaledMargin}`,
      marginTop: `mt-${scaledMargin}`,
      marginRight: `mr-${scaledMargin}`,
      marginBottom: `mb-${scaledMargin}`,
      marginLeft: `ml-${scaledMargin}`,
      marginStart: `ms-${scaledMargin}`,
      marginEnd: `me-${scaledMargin}`,
      negativeMargin: `-m-${scaledMargin}`,
      negativeMarginX: `-mx-${scaledMargin}`,
      negativeMarginY: `-my-${scaledMargin}`,
    };
  }, [baseMargin, scale]);
}

// Hook for flow spacing
export function useFlowSpacing() {
  const { scale, flowDirection } = useMargin();
  
  return useMemo(() => ({
    getFlowClass: (size: 'xs' | 'sm' | 'md' | 'lg' | 'xl' = 'md') => {
      return flowDirection === 'horizontal' 
        ? `flow-horizontal-${size}`
        : `flow-${size}`;
    },
    applyFlow: (element: HTMLElement, size: string = 'md') => {
      const flowClass = flowDirection === 'horizontal' 
        ? `flow-horizontal-${size}`
        : `flow-${size}`;
      element.classList.add(flowClass);
    }
  }), [flowDirection]);
}

// Hook for margin collapse prevention
export function useMarginCollapse() {
  const { collapseMode } = useMargin();
  
  return {
    preventCollapse: collapseMode === 'prevent',
    collapseClass: collapseMode === 'prevent' ? 'flex flex-col' : '',
    getContainerProps: () => {
      if (collapseMode === 'prevent') {
        return {
          style: { display: 'flex', flexDirection: 'column' as const }
        };
      }
      return {};
    }
  };
}
```
Creates hooks for margin utilities.

### 5. Create margin patterns
```typescript
// app/components/margin/MarginPatterns.tsx
'use client';

import React from 'react';
import { cn } from '@/lib/utils';

// Section with consistent vertical rhythm
export function RhythmSection({ 
  children, 
  className,
  ...props 
}: {
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <section className={cn('flow-lg', className)} {...props}>
      {children}
    </section>
  );
}

// Article with typography rhythm
export function Article({ 
  children, 
  className,
  ...props 
}: {
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <article className={cn('flow prose prose-lg mx-auto', className)} {...props}>
      {children}
    </article>
  );
}

// Card grid with consistent gaps
export function CardGrid({ 
  children, 
  columns = { base: 1, md: 2, lg: 3 },
  gap = '6',
  className,
  ...props 
}: {
  children: React.ReactNode;
  columns?: { base: number; md?: number; lg?: number; xl?: number };
  gap?: string;
  className?: string;
}) {
  const gridClasses = [
    `grid`,
    `grid-cols-${columns.base}`,
    columns.md && `md:grid-cols-${columns.md}`,
    columns.lg && `lg:grid-cols-${columns.lg}`,
    columns.xl && `xl:grid-cols-${columns.xl}`,
    `gap-${gap}`
  ].filter(Boolean).join(' ');

  return (
    <div className={cn(gridClasses, className)} {...props}>
      {children}
    </div>
  );
}

// Offset component for negative margins
export function Offset({ 
  children, 
  offset = '4',
  direction = 'all',
  className,
  ...props 
}: {
  children: React.ReactNode;
  offset?: string;
  direction?: 'all' | 'x' | 'y' | 'top' | 'right' | 'bottom' | 'left';
  className?: string;
}) {
  const offsetClasses = {
    all: `-m-${offset}`,
    x: `-mx-${offset}`,
    y: `-my-${offset}`,
    top: `-mt-${offset}`,
    right: `-mr-${offset}`,
    bottom: `-mb-${offset}`,
    left: `-ml-${offset}`,
  };

  return (
    <div className={cn(offsetClasses[direction], className)} {...props}>
      {children}
    </div>
  );
}

// Bleed component for full-width within containers
export function Bleed({ 
  children, 
  className,
  ...props 
}: {
  children: React.ReactNode;
  className?: string;
}) {
  return (
    <div className={cn('-mx-4 sm:-mx-6 lg:-mx-8', className)} {...props}>
      {children}
    </div>
  );
}
```
Creates common margin patterns.

### 6. Create margin configurator
```typescript
// app/components/margin/MarginConfigurator.tsx
'use client';

import React, { useState } from 'react';
import { useMargin } from '@/contexts/MarginContext';
import { cn } from '@/lib/utils';

export function MarginConfigurator() {
  const { scale, setScale, collapseMode, setCollapseMode, flowDirection, setFlowDirection } = useMargin();
  const [isOpen, setIsOpen] = useState(false);

  const marginScales = [
    { name: 'tight', scale: 0.75, rhythm: 1.25, description: 'Compact spacing' },
    { name: 'default', scale: 1, rhythm: 1.5, description: 'Standard spacing' },
    { name: 'relaxed', scale: 1.25, rhythm: 1.618, description: 'Golden ratio' },
    { name: 'loose', scale: 1.5, rhythm: 2, description: 'Generous spacing' },
  ];

  if (process.env.NODE_ENV !== 'development') return null;

  return (
    <>
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="fixed bottom-20 right-4 bg-purple-500 text-white p-3 rounded-full shadow-lg z-50 hover:bg-purple-600"
        aria-label="Configure margins"
      >
        <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>

      {isOpen && (
        <div className="fixed bottom-36 right-4 bg-white dark:bg-gray-800 rounded-lg shadow-xl p-6 z-50 w-96">
          <h3 className="text-lg font-semibold mb-4">Margin Configuration</h3>
          
          <div className="space-y-6">
            {/* Scale selector */}
            <div>
              <label className="block text-sm font-medium mb-2">Margin Scale</label>
              <div className="grid grid-cols-2 gap-2">
                {marginScales.map((s) => (
                  <button
                    key={s.name}
                    onClick={() => setScale(s)}
                    className={cn(
                      'p-3 rounded-lg border text-sm transition-colors',
                      scale.name === s.name
                        ? 'border-purple-500 bg-purple-50 dark:bg-purple-900/20'
                        : 'border-gray-200 hover:border-gray-300 dark:border-gray-700'
                    )}
                  >
                    <div className="font-medium">{s.name}</div>
                    <div className="text-xs text-gray-600 dark:text-gray-400">
                      {s.scale}x / {s.rhythm} rhythm
                    </div>
                  </button>
                ))}
              </div>
            </div>

            {/* Collapse mode */}
            <div>
              <label className="block text-sm font-medium mb-2">Margin Collapse</label>
              <select
                value={collapseMode}
                onChange={(e) => setCollapseMode(e.target.value as any)}
                className="w-full border rounded-lg px-3 py-2"
              >
                <option value="allow">Allow (default CSS)</option>
                <option value="prevent">Prevent (use flexbox)</option>
                <option value="smart">Smart (context-aware)</option>
              </select>
            </div>

            {/* Flow direction */}
            <div>
              <label className="block text-sm font-medium mb-2">Flow Direction</label>
              <div className="flex gap-2">
                {(['vertical', 'horizontal', 'both'] as const).map((dir) => (
                  <button
                    key={dir}
                    onClick={() => setFlowDirection(dir)}
                    className={cn(
                      'flex-1 py-2 rounded-lg border transition-colors',
                      flowDirection === dir
                        ? 'border-purple-500 bg-purple-50 dark:bg-purple-900/20'
                        : 'border-gray-200 hover:border-gray-300 dark:border-gray-700'
                    )}
                  >
                    {dir}
                  </button>
                ))}
              </div>
            </div>

            {/* Preview */}
            <div className="pt-4 border-t">
              <p className="text-sm font-medium mb-3">Preview</p>
              <div className="space-y-2 p-4 bg-gray-50 dark:bg-gray-900 rounded-lg">
                <div className="flow-sm">
                  <div className="p-3 bg-purple-100 dark:bg-purple-900/30 rounded">Item 1</div>
                  <div className="p-3 bg-purple-100 dark:bg-purple-900/30 rounded">Item 2</div>
                  <div className="p-3 bg-purple-100 dark:bg-purple-900/30 rounded">Item 3</div>
                </div>
              </div>
              <p className="text-xs text-gray-600 dark:text-gray-400 mt-2">
                Current rhythm: {scale.rhythm} • Scale: {scale.scale}x
              </p>
            </div>
          </div>
        </div>
      )}
    </>
  );
}
```
Creates interactive margin configurator.

### 7. Add advanced margin utilities
```css
/* Advanced margin utilities for Tailwind v4 */
@layer utilities {
  /* Optical alignment adjustments */
  .mt-optical {
    margin-top: calc(var(--spacing-4) * 0.8);
  }
  
  .mb-optical {
    margin-bottom: calc(var(--spacing-4) * 0.8);
  }
  
  /* First/last child margin removal */
  .trim-start > :first-child {
    margin-top: 0;
  }
  
  .trim-end > :last-child {
    margin-bottom: 0;
  }
  
  .trim-both > :first-child {
    margin-top: 0;
  }
  
  .trim-both > :last-child {
    margin-bottom: 0;
  }
  
  /* Consistent component spacing */
  .component-spacing > * {
    margin-bottom: var(--flow-space-lg);
  }
  
  .component-spacing > :last-child {
    margin-bottom: 0;
  }
  
  /* Responsive flow spacing */
  @media (min-width: 768px) {
    .md\:flow > * + * {
      margin-top: calc(var(--flow-space-md) * 1.25);
    }
  }
  
  @media (min-width: 1024px) {
    .lg\:flow > * + * {
      margin-top: calc(var(--flow-space-md) * 1.5);
    }
  }
}
```
Adds advanced margin utilities.

### 8. Complete implementation example
```typescript
// app/layout.tsx
import './globals.css';
import { MarginProvider } from '@/contexts/MarginContext';
import { MarginConfigurator } from '@/components/margin/MarginConfigurator';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <MarginProvider>
          {children}
          <MarginConfigurator />
        </MarginProvider>
      </body>
    </html>
  );
}

// app/page.tsx
import { 
  Flow, 
  Stack, 
  Center, 
  RhythmSection,
  CardGrid,
  Bleed 
} from '@/components/margin';

export default function Home() {
  return (
    <main>
      {/* Hero section with vertical rhythm */}
      <RhythmSection className="px-4 py-16">
        <Center maxWidth="4xl">
          <h1 className="text-5xl font-bold">Dynamic Margin System</h1>
          <p className="text-xl text-gray-600">
            Consistent spacing with Tailwind CSS v4
          </p>
        </Center>
      </RhythmSection>

      {/* Content with flow spacing */}
      <Flow space="lg" className="px-4">
        <section>
          <h2 className="text-3xl font-semibold mb-6">Features</h2>
          <CardGrid columns={{ base: 1, md: 2, lg: 3 }} gap="6">
            {[1, 2, 3, 4, 5, 6].map((i) => (
              <div key={i} className="p-6 bg-white shadow-lg rounded-lg">
                <h3 className="text-xl font-semibold mb-2">Feature {i}</h3>
                <p className="text-gray-600">
                  Automatically spaced with consistent margins.
                </p>
              </div>
            ))}
          </CardGrid>
        </section>

        <section>
          <h2 className="text-3xl font-semibold mb-6">Full Width Content</h2>
          <Bleed>
            <img 
              src="/api/placeholder/1200/400" 
              alt="Full width" 
              className="w-full"
            />
          </Bleed>
        </section>

        <Stack gap="8" collapse="prevent">
          <div>Content without margin collapse</div>
          <div>Each item maintains its spacing</div>
          <div>Using flexbox gap instead</div>
        </Stack>
      </Flow>
    </main>
  );
}
```
Complete example showing margin system usage.

## Validation
- Margin scales apply correctly
- Flow spacing maintains rhythm
- Margin collapse behavior works as expected
- Responsive margins adjust properly
- CSS variables update dynamically

## Error Handling
- **"Margins not updating"** - Check MarginProvider is present
- **"Flow not working"** - Ensure using correct flow classes
- **"Collapse not prevented"** - Verify collapse mode settings
- **"Negative margins breaking layout"** - Check container overflow

## Safety Notes
- Test margin collapse behavior across browsers
- Ensure adequate spacing for touch targets
- Verify layout stability with dynamic margins
- Test with different writing modes (RTL)
- Monitor CLS (Cumulative Layout Shift)

## Examples
- **Consistent vertical rhythm**
  ```jsx
  <Flow space="lg">
    <h1>Title</h1>
    <p>Paragraph</p>
    <Button>Action</Button>
  </Flow>
  ```

- **Prevent margin collapse**
  ```jsx
  <Stack gap="6" collapse="prevent">
    <Card />
    <Card />
  </Stack>
  ```

- **Center with max width**
  ```jsx
  <Center maxWidth="4xl" axis="horizontal">
    <Content />
  </Center>
  ```

- **Full-width bleed**
  ```jsx
  <article className="px-6">
    <h1>Article Title</h1>
    <Bleed>
      <img src="hero.jpg" className="w-full" />
    </Bleed>
    <p>Article content...</p>
  </article>
  ```