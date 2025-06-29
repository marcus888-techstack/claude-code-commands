# Tailwind Layout Flex

## Purpose
Configure flexbox layouts for flexible, one-dimensional layouts with alignment and distribution controls in Tailwind CSS v4 for React/Next.js applications

## Context
Use to create flexible layouts for navigation bars, card groups, centering content, or any single-axis layout needs. More suitable than grid for one-dimensional layouts. Supports all flexbox properties including direction, wrap, alignment, and spacing. Optimized for Tailwind CSS v4's CSS-first configuration and React/Next.js component architecture.

## Parameters
- `$FLEX_DIRECTION` - Direction of flex items
  - Required
  - Options: `row`, `row-reverse`, `col`, `col-reverse`
- `$JUSTIFY` - Justification along main axis
  - Optional
  - Default: `start`
  - Options: `start`, `end`, `center`, `between`, `around`, `evenly`
- `$ALIGN` - Alignment along cross axis
  - Optional
  - Default: `stretch`
  - Options: `start`, `end`, `center`, `baseline`, `stretch`
- `$SELECTOR` - Target container selector
  - Optional
  - Default: `.flex`
  - Example: `.navbar`, `.card-group`, `.header`

## Steps

### 1. Create React component for flex configuration
```javascript
// FlexConfigurator.jsx (or .tsx for TypeScript)
import React, { useState, useEffect } from 'react';

const FlexConfigurator = ({ targetSelector = '.flex' }) => {
  const [flexConfig, setFlexConfig] = useState({
    direction: 'row',
    justify: 'start',
    align: 'stretch'
  });

  const flexMappings = {
    direction: {
      row: 'flex-row',
      'row-reverse': 'flex-row-reverse',
      col: 'flex-col',
      'col-reverse': 'flex-col-reverse'
    },
    justify: {
      start: 'justify-start',
      end: 'justify-end',
      center: 'justify-center',
      between: 'justify-between',
      around: 'justify-around',
      evenly: 'justify-evenly'
    },
    align: {
      start: 'items-start',
      end: 'items-end',
      center: 'items-center',
      baseline: 'items-baseline',
      stretch: 'items-stretch'
    }
  };

  return { flexConfig, setFlexConfig, flexMappings };
};
```
Defines flex configuration as a React component for Tailwind CSS v4.

### 2. Apply flex layout in React/Next.js
```javascript
// useFlexLayout.js - Custom hook for flex layouts
import { useEffect, useCallback } from 'react';

export const useFlexLayout = (config) => {
  const applyFlexClasses = useCallback((element, direction, justify, align) => {
    if (!element) return;
    
    // Build class string using Tailwind v4 classes
    const classes = [
      'flex',
      direction && `flex-${direction}`,
      justify && `justify-${justify}`,
      align && `items-${align}`
    ].filter(Boolean).join(' ');
    
    return classes;
  }, []);
  
  // For dynamic class application
  const FlexContainer = ({ children, direction = 'row', justify = 'start', align = 'stretch', className = '', ...props }) => {
    const flexClasses = applyFlexClasses({}, direction, justify, align);
    
    return (
      <div className={`${flexClasses} ${className}`} {...props}>
        {children}
      </div>
    );
  };
  
  // Save preferences to localStorage
  const saveFlexPreferences = useCallback((key, config) => {
    if (typeof window !== 'undefined') {
      const preferences = JSON.parse(localStorage.getItem('flex-preferences') || '{}');
      preferences[key] = config;
      localStorage.setItem('flex-preferences', JSON.stringify(preferences));
    }
  }, []);
  
  return { applyFlexClasses, FlexContainer, saveFlexPreferences };
};
```
Provides React hooks and components for applying flexbox layouts with Tailwind CSS v4.

### 3. Configure Tailwind CSS v4 theme
```css
/* In your global CSS file (app/globals.css or styles/globals.css) */
@import "tailwindcss";

/* Define custom flex theme variables */
@theme {
  /* Custom spacing for flex gaps */
  --spacing-flex-sm: 0.5rem;
  --spacing-flex-md: 1rem;
  --spacing-flex-lg: 1.5rem;
  --spacing-flex-xl: 2rem;
  
  /* Custom flex basis values */
  --flex-basis-1: 8.333333%;
  --flex-basis-2: 16.666667%;
  --flex-basis-3: 25%;
  --flex-basis-4: 33.333333%;
  --flex-basis-5: 41.666667%;
  --flex-basis-6: 50%;
  --flex-basis-7: 58.333333%;
  --flex-basis-8: 66.666667%;
  --flex-basis-9: 75%;
  --flex-basis-10: 83.333333%;
  --flex-basis-11: 91.666667%;
  --flex-basis-12: 100%;
}

/* Flex utilities are built-in with Tailwind v4
   These are available automatically:
  /* Display */
  .flex { display: flex; }
  .inline-flex { display: inline-flex; }
  
  /* Flex direction */
  .flex-row { flex-direction: row; }
  .flex-row-reverse { flex-direction: row-reverse; }
  .flex-col { flex-direction: column; }
  .flex-col-reverse { flex-direction: column-reverse; }
  
  /* Flex wrap */
  .flex-wrap { flex-wrap: wrap; }
  .flex-wrap-reverse { flex-wrap: wrap-reverse; }
  .flex-nowrap { flex-wrap: nowrap; }
  
  /* Justify content */
  .justify-start { justify-content: flex-start; }
  .justify-end { justify-content: flex-end; }
  .justify-center { justify-content: center; }
  .justify-between { justify-content: space-between; }
  .justify-around { justify-content: space-around; }
  .justify-evenly { justify-content: space-evenly; }
  
  /* Align items */
  .items-start { align-items: flex-start; }
  .items-end { align-items: flex-end; }
  .items-center { align-items: center; }
  .items-baseline { align-items: baseline; }
  .items-stretch { align-items: stretch; }
  
  /* Align self */
  .self-auto { align-self: auto; }
  .self-start { align-self: flex-start; }
  .self-end { align-self: flex-end; }
  .self-center { align-self: center; }
  .self-stretch { align-self: stretch; }
  .self-baseline { align-self: baseline; }
  
  /* Align content */
  .content-start { align-content: flex-start; }
  .content-end { align-content: flex-end; }
  .content-center { align-content: center; }
  .content-between { align-content: space-between; }
  .content-around { align-content: space-around; }
  .content-evenly { align-content: space-evenly; }
  
  /* Flex grow and shrink */
  .flex-1 { flex: 1 1 0%; }
  .flex-auto { flex: 1 1 auto; }
  .flex-initial { flex: 0 1 auto; }
  .flex-none { flex: none; }
  
  .grow { flex-grow: 1; }
  .grow-0 { flex-grow: 0; }
  
  .shrink { flex-shrink: 1; }
  .shrink-0 { flex-shrink: 0; }
  
  /* Flex basis */
  .basis-auto { flex-basis: auto; }
  .basis-0 { flex-basis: 0; }
  .basis-1 { flex-basis: 0.25rem; }
  .basis-2 { flex-basis: 0.5rem; }
  .basis-3 { flex-basis: 0.75rem; }
  .basis-4 { flex-basis: 1rem; }
  .basis-5 { flex-basis: 1.25rem; }
  .basis-6 { flex-basis: 1.5rem; }
  .basis-8 { flex-basis: 2rem; }
  .basis-10 { flex-basis: 2.5rem; }
  .basis-12 { flex-basis: 3rem; }
  .basis-16 { flex-basis: 4rem; }
  .basis-20 { flex-basis: 5rem; }
  .basis-24 { flex-basis: 6rem; }
  .basis-32 { flex-basis: 8rem; }
  .basis-40 { flex-basis: 10rem; }
  .basis-48 { flex-basis: 12rem; }
  .basis-56 { flex-basis: 14rem; }
  .basis-64 { flex-basis: 16rem; }
  .basis-1\\/2 { flex-basis: 50%; }
  .basis-1\\/3 { flex-basis: 33.333333%; }
  .basis-2\\/3 { flex-basis: 66.666667%; }
  .basis-1\\/4 { flex-basis: 25%; }
  .basis-2\\/4 { flex-basis: 50%; }
  .basis-3\\/4 { flex-basis: 75%; }
  .basis-full { flex-basis: 100%; }
  
  /* Order */
  .order-first { order: -9999; }
  .order-last { order: 9999; }
  .order-none { order: 0; }
  .order-1 { order: 1; }
  .order-2 { order: 2; }
  .order-3 { order: 3; }
  .order-4 { order: 4; }
  .order-5 { order: 5; }
  .order-6 { order: 6; }
  .order-7 { order: 7; }
  .order-8 { order: 8; }
  .order-9 { order: 9; }
  .order-10 { order: 10; }
  .order-11 { order: 11; }
  .order-12 { order: 12; }
}
```
Creates comprehensive flex utilities.

### 4. Create responsive flex components
```javascript
// ResponsiveFlex.jsx - Responsive flex component for React/Next.js
import React from 'react';
import { cn } from '@/lib/utils'; // Utility for combining classes

export const ResponsiveFlex = ({ 
  children,
  direction = { base: 'col', sm: 'row' },
  justify = { base: 'start', md: 'between' },
  align = { base: 'start', lg: 'center' },
  gap = { base: '2', md: '4', lg: '6' },
  wrap = false,
  className,
  ...props 
}) => {
  // Build responsive class string
  const buildResponsiveClasses = () => {
    const classes = ['flex'];
    
    // Direction classes
    if (typeof direction === 'string') {
      classes.push(`flex-${direction}`);
    } else {
      Object.entries(direction).forEach(([breakpoint, value]) => {
        const prefix = breakpoint === 'base' ? '' : `${breakpoint}:`;
        classes.push(`${prefix}flex-${value}`);
      });
    }
    
    // Justify classes
    if (typeof justify === 'string') {
      classes.push(`justify-${justify}`);
    } else {
      Object.entries(justify).forEach(([breakpoint, value]) => {
        const prefix = breakpoint === 'base' ? '' : `${breakpoint}:`;
        classes.push(`${prefix}justify-${value}`);
      });
    }
    
    // Align classes
    if (typeof align === 'string') {
      classes.push(`items-${align}`);
    } else {
      Object.entries(align).forEach(([breakpoint, value]) => {
        const prefix = breakpoint === 'base' ? '' : `${breakpoint}:`;
        classes.push(`${prefix}items-${value}`);
      });
    }
    
    // Gap classes
    if (typeof gap === 'string') {
      classes.push(`gap-${gap}`);
    } else {
      Object.entries(gap).forEach(([breakpoint, value]) => {
        const prefix = breakpoint === 'base' ? '' : `${breakpoint}:`;
        classes.push(`${prefix}gap-${value}`);
      });
    }
    
    // Wrap
    if (wrap) {
      classes.push('flex-wrap');
    }
    
    return classes.join(' ');
  };
  
  return (
    <div className={cn(buildResponsiveClasses(), className)} {...props}>
      {children}
    </div>
  );
};

/* Usage examples in Next.js:
<ResponsiveFlex
  direction={{ base: 'col', md: 'row' }}
  justify={{ base: 'start', lg: 'between' }}
  align="center"
  gap={{ base: '2', md: '4' }}
>
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</ResponsiveFlex>
*/
    /* Responsive flex direction */
    @media (min-width: 640px) {
      .sm\\:flex-row { flex-direction: row; }
      .sm\\:flex-row-reverse { flex-direction: row-reverse; }
      .sm\\:flex-col { flex-direction: column; }
      .sm\\:flex-col-reverse { flex-direction: column-reverse; }
    }
    
    @media (min-width: 768px) {
      .md\\:flex-row { flex-direction: row; }
      .md\\:flex-row-reverse { flex-direction: row-reverse; }
      .md\\:flex-col { flex-direction: column; }
      .md\\:flex-col-reverse { flex-direction: column-reverse; }
    }
    
    @media (min-width: 1024px) {
      .lg\\:flex-row { flex-direction: row; }
      .lg\\:flex-row-reverse { flex-direction: row-reverse; }
      .lg\\:flex-col { flex-direction: column; }
      .lg\\:flex-col-reverse { flex-direction: column-reverse; }
    }
    
    /* Responsive justify content */
    @media (min-width: 640px) {
      .sm\\:justify-start { justify-content: flex-start; }
      .sm\\:justify-end { justify-content: flex-end; }
      .sm\\:justify-center { justify-content: center; }
      .sm\\:justify-between { justify-content: space-between; }
      .sm\\:justify-around { justify-content: space-around; }
      .sm\\:justify-evenly { justify-content: space-evenly; }
    }
    
    @media (min-width: 768px) {
      .md\\:justify-start { justify-content: flex-start; }
      .md\\:justify-end { justify-content: flex-end; }
      .md\\:justify-center { justify-content: center; }
      .md\\:justify-between { justify-content: space-between; }
      .md\\:justify-around { justify-content: space-around; }
      .md\\:justify-evenly { justify-content: space-evenly; }
    }
    
    @media (min-width: 1024px) {
      .lg\\:justify-start { justify-content: flex-start; }
      .lg\\:justify-end { justify-content: flex-end; }
      .lg\\:justify-center { justify-content: center; }
      .lg\\:justify-between { justify-content: space-between; }
      .lg\\:justify-around { justify-content: space-around; }
      .lg\\:justify-evenly { justify-content: space-evenly; }
    }
    
    /* Responsive align items */
    @media (min-width: 640px) {
      .sm\\:items-start { align-items: flex-start; }
      .sm\\:items-end { align-items: flex-end; }
      .sm\\:items-center { align-items: center; }
      .sm\\:items-baseline { align-items: baseline; }
      .sm\\:items-stretch { align-items: stretch; }
    }
    
    @media (min-width: 768px) {
      .md\\:items-start { align-items: flex-start; }
      .md\\:items-end { align-items: flex-end; }
      .md\\:items-center { align-items: center; }
      .md\\:items-baseline { align-items: baseline; }
      .md\\:items-stretch { align-items: stretch; }
    }
    
    @media (min-width: 1024px) {
      .lg\\:items-start { align-items: flex-start; }
      .lg\\:items-end { align-items: flex-end; }
      .lg\\:items-center { align-items: center; }
      .lg\\:items-baseline { align-items: baseline; }
      .lg\\:items-stretch { align-items: stretch; }
    }
    
    /* Responsive order */
    @media (min-width: 640px) {
      .sm\\:order-first { order: -9999; }
      .sm\\:order-last { order: 9999; }
      .sm\\:order-none { order: 0; }
      .sm\\:order-1 { order: 1; }
      .sm\\:order-2 { order: 2; }
      .sm\\:order-3 { order: 3; }
      .sm\\:order-4 { order: 4; }
      .sm\\:order-5 { order: 5; }
      .sm\\:order-6 { order: 6; }
    }
    
    @media (min-width: 768px) {
      .md\\:order-first { order: -9999; }
      .md\\:order-last { order: 9999; }
      .md\\:order-none { order: 0; }
      .md\\:order-1 { order: 1; }
      .md\\:order-2 { order: 2; }
      .md\\:order-3 { order: 3; }
      .md\\:order-4 { order: 4; }
      .md\\:order-5 { order: 5; }
      .md\\:order-6 { order: 6; }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = responsiveStyles;
  document.head.appendChild(style);
}
```
Adds responsive flexbox layouts.

### 5. Create flex pattern components
```javascript
// FlexPatterns.jsx - Common flex layout patterns for React/Next.js
import React from 'react';

// Center content pattern
export const FlexCenter = ({ children, className = '', ...props }) => (
  <div className={`flex justify-center items-center ${className}`} {...props}>
    {children}
  </div>
);

// Navigation pattern
export const FlexNav = ({ logo, links, actions, className = '' }) => (
  <nav className={`flex justify-between items-center flex-wrap gap-4 ${className}`}>
    <div className="flex-shrink-0">{logo}</div>
    <div className="flex items-center gap-6">{links}</div>
    <div className="flex items-center gap-2">{actions}</div>
  </nav>
);

// Card group pattern
export const FlexCards = ({ children, className = '' }) => (
  <div className={`flex flex-wrap gap-6 ${className}`}>
    {React.Children.map(children, (child) => (
      <div className="flex-1 basis-80">{child}</div>
    ))}
  </div>
);

// Sidebar layout pattern
export const FlexSidebar = ({ sidebar, children, sidebarPosition = 'left', className = '' }) => (
  <div className={`flex gap-8 ${className}`}>
    {sidebarPosition === 'left' && (
      <aside className="flex-none w-64">{sidebar}</aside>
    )}
    <main className="flex-1">{children}</main>
    {sidebarPosition === 'right' && (
      <aside className="flex-none w-64">{sidebar}</aside>
    )}
  </div>
);

// Media object pattern
export const FlexMedia = ({ media, children, className = '' }) => (
  <div className={`flex items-start gap-4 ${className}`}>
    <div className="flex-none">{media}</div>
    <div className="flex-1">{children}</div>
  </div>
);

// Sticky footer pattern (for Next.js layouts)
export const FlexStickyFooter = ({ header, children, footer }) => (
  <div className="flex flex-col min-h-screen">
    {header && <header className="flex-none">{header}</header>}
    <main className="flex-1">{children}</main>
    <footer className="flex-none">{footer}</footer>
  </div>
);

// Button group pattern
export const FlexButtonGroup = ({ children, className = '' }) => (
  <div className={`inline-flex gap-2 ${className}`}>
    {children}
  </div>
);

// Equal columns pattern
export const FlexEqualColumns = ({ children, className = '' }) => (
  <div className={`flex gap-4 ${className}`}>
    {React.Children.map(children, (child) => (
      <div className="flex-1">{child}</div>
    ))}
  </div>
);
    /* Common flex patterns */
    
    /* Center content */
    .flex-center {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    
    /* Space between with centered items */
    .flex-between-center {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    
    /* Navigation pattern */
    .flex-nav {
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
      gap: 1rem;
    }
    
    /* Card group pattern */
    .flex-cards {
      display: flex;
      flex-wrap: wrap;
      gap: 1.5rem;
    }
    .flex-cards > * {
      flex: 1 1 300px;
    }
    
    /* Sidebar layout */
    .flex-sidebar {
      display: flex;
      gap: 2rem;
    }
    .flex-sidebar-main {
      flex: 1 1 0%;
    }
    .flex-sidebar-aside {
      flex: 0 0 300px;
    }
    
    /* Equal columns */
    .flex-equal > * {
      flex: 1 1 0%;
    }
    
    /* Sticky footer pattern */
    .flex-sticky-footer {
      display: flex;
      flex-direction: column;
      min-height: 100vh;
    }
    .flex-sticky-footer-content {
      flex: 1 0 auto;
    }
    
    /* Split screen */
    .flex-split {
      display: flex;
      height: 100vh;
    }
    .flex-split > * {
      flex: 1 1 0%;
      overflow: auto;
    }
    
    /* Media object pattern */
    .flex-media {
      display: flex;
      align-items: flex-start;
      gap: 1rem;
    }
    .flex-media-figure {
      flex: 0 0 auto;
    }
    .flex-media-body {
      flex: 1 1 0%;
    }
    
    /* Button group */
    .flex-button-group {
      display: inline-flex;
      gap: 0.5rem;
    }
    
    /* Responsive stack */
    .flex-stack {
      display: flex;
      flex-direction: column;
      gap: 1rem;
    }
    @media (min-width: 768px) {
      .flex-stack-md {
        flex-direction: row;
      }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = patterns;
  document.head.appendChild(style);
}
```
Creates common flex patterns.

### 6. Create dynamic flex configurator component
```javascript
// FlexConfigurator.jsx - Interactive flex configuration for development
import React, { useState, useEffect } from 'react';

export const FlexConfigurator = ({ children, showControls = false }) => {
  const [config, setConfig] = useState({
    direction: 'row',
    justify: 'start',
    align: 'stretch',
    wrap: false,
    gap: '4'
  });
  
  const [isOpen, setIsOpen] = useState(false);
  
  // Only show in development
  if (process.env.NODE_ENV !== 'development' || !showControls) {
    return (
      <div className={`flex flex-${config.direction} justify-${config.justify} items-${config.align} gap-${config.gap} ${config.wrap ? 'flex-wrap' : ''}`}>
        {children}
      </div>
    );
  }
  
  return (
    <div className="relative">
      <div className={`flex flex-${config.direction} justify-${config.justify} items-${config.align} gap-${config.gap} ${config.wrap ? 'flex-wrap' : ''} border-2 border-dashed border-blue-500 p-4`}>
        {children}
      </div>
      
      {/* Configuration Panel */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="absolute top-2 right-2 bg-blue-500 text-white px-3 py-1 rounded text-sm"
      >
        {isOpen ? 'Close' : 'Configure'}
      </button>
      
      {isOpen && (
        <div className="absolute top-10 right-2 bg-white shadow-lg rounded-lg p-4 z-50 w-64">
          <h3 className="font-bold mb-3">Flex Configuration</h3>
          
          <div className="space-y-3">
            <div>
              <label className="block text-sm font-medium mb-1">Direction</label>
              <select
                value={config.direction}
                onChange={(e) => setConfig({ ...config, direction: e.target.value })}
                className="w-full border rounded px-2 py-1"
              >
                <option value="row">Row</option>
                <option value="row-reverse">Row Reverse</option>
                <option value="col">Column</option>
                <option value="col-reverse">Column Reverse</option>
              </select>
            </div>
            
            <div>
              <label className="block text-sm font-medium mb-1">Justify</label>
              <select
                value={config.justify}
                onChange={(e) => setConfig({ ...config, justify: e.target.value })}
                className="w-full border rounded px-2 py-1"
              >
                <option value="start">Start</option>
                <option value="end">End</option>
                <option value="center">Center</option>
                <option value="between">Between</option>
                <option value="around">Around</option>
                <option value="evenly">Evenly</option>
              </select>
            </div>
            
            <div>
              <label className="block text-sm font-medium mb-1">Align</label>
              <select
                value={config.align}
                onChange={(e) => setConfig({ ...config, align: e.target.value })}
                className="w-full border rounded px-2 py-1"
              >
                <option value="start">Start</option>
                <option value="end">End</option>
                <option value="center">Center</option>
                <option value="baseline">Baseline</option>
                <option value="stretch">Stretch</option>
              </select>
            </div>
            
            <div>
              <label className="block text-sm font-medium mb-1">Gap</label>
              <input
                type="range"
                min="0"
                max="20"
                value={config.gap}
                onChange={(e) => setConfig({ ...config, gap: e.target.value })}
                className="w-full"
              />
              <span className="text-sm text-gray-600">gap-{config.gap}</span>
            </div>
            
            <div>
              <label className="flex items-center">
                <input
                  type="checkbox"
                  checked={config.wrap}
                  onChange={(e) => setConfig({ ...config, wrap: e.target.checked })}
                  className="mr-2"
                />
                <span className="text-sm font-medium">Flex Wrap</span>
              </label>
            </div>
          </div>
          
          <div className="mt-4 p-2 bg-gray-100 rounded">
            <p className="text-xs font-mono">
              flex flex-{config.direction} justify-{config.justify} items-{config.align} gap-{config.gap} {config.wrap ? 'flex-wrap' : ''}
            </p>
          </div>
        </div>
      )}
    </div>
  );
};
    /* Flex gap utilities */
    .gap-0 { gap: 0px; }
    .gap-px { gap: 1px; }
    .gap-0\\.5 { gap: 0.125rem; }
    .gap-1 { gap: 0.25rem; }
    .gap-1\\.5 { gap: 0.375rem; }
    .gap-2 { gap: 0.5rem; }
    .gap-2\\.5 { gap: 0.625rem; }
    .gap-3 { gap: 0.75rem; }
    .gap-3\\.5 { gap: 0.875rem; }
    .gap-4 { gap: 1rem; }
    .gap-5 { gap: 1.25rem; }
    .gap-6 { gap: 1.5rem; }
    .gap-7 { gap: 1.75rem; }
    .gap-8 { gap: 2rem; }
    .gap-9 { gap: 2.25rem; }
    .gap-10 { gap: 2.5rem; }
    .gap-11 { gap: 2.75rem; }
    .gap-12 { gap: 3rem; }
    .gap-14 { gap: 3.5rem; }
    .gap-16 { gap: 4rem; }
    .gap-20 { gap: 5rem; }
    
    /* Directional gaps */
    .gap-x-0 { column-gap: 0px; }
    .gap-x-px { column-gap: 1px; }
    .gap-x-0\\.5 { column-gap: 0.125rem; }
    .gap-x-1 { column-gap: 0.25rem; }
    .gap-x-2 { column-gap: 0.5rem; }
    .gap-x-3 { column-gap: 0.75rem; }
    .gap-x-4 { column-gap: 1rem; }
    .gap-x-5 { column-gap: 1.25rem; }
    .gap-x-6 { column-gap: 1.5rem; }
    .gap-x-8 { column-gap: 2rem; }
    
    .gap-y-0 { row-gap: 0px; }
    .gap-y-px { row-gap: 1px; }
    .gap-y-0\\.5 { row-gap: 0.125rem; }
    .gap-y-1 { row-gap: 0.25rem; }
    .gap-y-2 { row-gap: 0.5rem; }
    .gap-y-3 { row-gap: 0.75rem; }
    .gap-y-4 { row-gap: 1rem; }
    .gap-y-5 { row-gap: 1.25rem; }
    .gap-y-6 { row-gap: 1.5rem; }
    .gap-y-8 { row-gap: 2rem; }
    
    /* Responsive gaps */
    @media (min-width: 640px) {
      .sm\\:gap-0 { gap: 0px; }
      .sm\\:gap-1 { gap: 0.25rem; }
      .sm\\:gap-2 { gap: 0.5rem; }
      .sm\\:gap-3 { gap: 0.75rem; }
      .sm\\:gap-4 { gap: 1rem; }
      .sm\\:gap-5 { gap: 1.25rem; }
      .sm\\:gap-6 { gap: 1.5rem; }
      .sm\\:gap-8 { gap: 2rem; }
    }
    
    @media (min-width: 768px) {
      .md\\:gap-0 { gap: 0px; }
      .md\\:gap-1 { gap: 0.25rem; }
      .md\\:gap-2 { gap: 0.5rem; }
      .md\\:gap-3 { gap: 0.75rem; }
      .md\\:gap-4 { gap: 1rem; }
      .md\\:gap-5 { gap: 1.25rem; }
      .md\\:gap-6 { gap: 1.5rem; }
      .md\\:gap-8 { gap: 2rem; }
    }
    
    @media (min-width: 1024px) {
      .lg\\:gap-0 { gap: 0px; }
      .lg\\:gap-1 { gap: 0.25rem; }
      .lg\\:gap-2 { gap: 0.5rem; }
      .lg\\:gap-3 { gap: 0.75rem; }
      .lg\\:gap-4 { gap: 1rem; }
      .lg\\:gap-5 { gap: 1.25rem; }
      .lg\\:gap-6 { gap: 1.5rem; }
      .lg\\:gap-8 { gap: 2rem; }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = gapStyles;
  document.head.appendChild(style);
}
```
Adds gap utilities for flexbox.

### 7. Create flex debugging component
```javascript
// FlexDebug.jsx - Flex debugging component for development
import React, { useEffect, useState } from 'react';

export const FlexDebug = ({ children, showInfo = true }) => {
  const [flexInfo, setFlexInfo] = useState({});
  const [itemInfo, setItemInfo] = useState([]);
  
  useEffect(() => {
    if (process.env.NODE_ENV !== 'development' || !showInfo) return;
    
    const updateFlexInfo = () => {
      const container = document.querySelector('.flex-debug-container');
      if (!container) return;
      
      const computedStyle = window.getComputedStyle(container);
      setFlexInfo({
        direction: computedStyle.flexDirection,
        justify: computedStyle.justifyContent,
        align: computedStyle.alignItems,
        wrap: computedStyle.flexWrap,
        gap: computedStyle.gap
      });
      
      const items = Array.from(container.children).map((child, index) => {
        const childStyle = window.getComputedStyle(child);
        return {
          index: index + 1,
          grow: childStyle.flexGrow,
          shrink: childStyle.flexShrink,
          basis: childStyle.flexBasis,
          order: childStyle.order
        };
      });
      setItemInfo(items);
    };
    
    updateFlexInfo();
    window.addEventListener('resize', updateFlexInfo);
    
    return () => window.removeEventListener('resize', updateFlexInfo);
  }, [showInfo]);
  
  if (process.env.NODE_ENV !== 'development' || !showInfo) {
    return <>{children}</>;
  }
  
  return (
    <div className="relative">
      <div className="flex-debug-container flex outline-2 outline-dashed outline-blue-500 relative">
        {React.Children.map(children, (child, index) => (
          <div className="outline-1 outline-dashed outline-red-500 relative">
            <span className="absolute top-0 left-0 bg-red-500 text-white text-xs px-1 z-10">
              {index + 1}
            </span>
            {child}
            {itemInfo[index] && (
              <div className="absolute bottom-0 left-0 right-0 bg-black bg-opacity-75 text-white text-xs p-1 z-10">
                grow: {itemInfo[index].grow} | shrink: {itemInfo[index].shrink}
              </div>
            )}
          </div>
        ))}
      </div>
      
      {flexInfo.direction && (
        <div className="absolute top-0 right-0 bg-blue-500 text-white text-xs px-2 py-1 z-20">
          {flexInfo.direction} | {flexInfo.justify} | {flexInfo.align}
        </div>
      )}
    </div>
  );
};

// CSS for debugging (add to globals.css)
const debugStyles = `
/* Flex debugging styles for Tailwind v4 */
@layer utilities {
  .flex-debug {
    @apply relative outline-2 outline-dashed outline-blue-500;
  }
  
  .flex-debug > * {
    @apply outline-1 outline-dashed outline-red-500 relative;
  }
  
  .flex-debug-info {
    @apply absolute top-0 right-0 bg-blue-500 text-white text-xs px-2 py-1 z-20;
  }
  
  .flex-debug-item-info {
    @apply absolute bottom-0 left-0 right-0 bg-black bg-opacity-75 text-white text-xs p-1 z-10;
  }
}`;
    /* Flex debugging */
    .flex-debug {
      position: relative;
      outline: 2px dashed rgba(59, 130, 246, 0.5);
    }
    
    .flex-debug > * {
      outline: 1px dashed rgba(239, 68, 68, 0.5);
      position: relative;
    }
    
    .flex-debug > *::before {
      content: attr(data-flex-item);
      position: absolute;
      top: 2px;
      left: 2px;
      background: rgba(239, 68, 68, 0.8);
      color: white;
      font-size: 10px;
      padding: 2px 4px;
      pointer-events: none;
      z-index: 1;
    }
    
    .flex-debug::after {
      content: attr(data-flex-info);
      position: absolute;
      top: 2px;
      right: 2px;
      background: rgba(59, 130, 246, 0.8);
      color: white;
      font-size: 10px;
      padding: 2px 4px;
      pointer-events: none;
      z-index: 1;
    }
    
    /* Show flex properties */
    .flex-debug-verbose > * {
      min-height: 80px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 12px;
      text-align: center;
    }
    
    .flex-debug-verbose > *::after {
      content: attr(data-flex-properties);
      opacity: 0.7;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = debugStyles;
  document.head.appendChild(style);
  
  // Update flex info
  function updateFlexInfo() {
    document.querySelectorAll('.flex-debug').forEach(container => {
      const direction = window.getComputedStyle(container).flexDirection;
      const justify = window.getComputedStyle(container).justifyContent;
      const align = window.getComputedStyle(container).alignItems;
      container.dataset.flexInfo = `${direction} | ${justify} | ${align}`;
      
      // Number flex items
      Array.from(container.children).forEach((child, index) => {
        child.dataset.flexItem = index + 1;
        
        if (container.classList.contains('flex-debug-verbose')) {
          const grow = window.getComputedStyle(child).flexGrow;
          const shrink = window.getComputedStyle(child).flexShrink;
          const basis = window.getComputedStyle(child).flexBasis;
          child.dataset.flexProperties = `grow: ${grow} | shrink: ${shrink} | basis: ${basis}`;
        }
      });
    });
  }
  
  updateFlexInfo();
  window.addEventListener('resize', updateFlexInfo);
}
```
Adds flex debugging features.

### 8. Complete flex system setup for Next.js
```javascript
// app/components/flex/index.js - Export all flex components
export * from './FlexPatterns';
export * from './ResponsiveFlex';
export * from './FlexConfigurator';
export * from './FlexDebug';
export { useFlexLayout } from './useFlexLayout';

// app/layout.js (or layout.tsx) - Root layout setup
import './globals.css';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}

// app/globals.css - Tailwind v4 configuration
@import "tailwindcss";

/* Custom theme for flex layouts */
@theme {
  /* Custom breakpoints if needed */
  --breakpoint-xs: 475px;
  --breakpoint-3xl: 1920px;
  
  /* Custom spacing for flex patterns */
  --spacing-flex-nav: clamp(1rem, 2vw, 2rem);
  --spacing-flex-section: clamp(2rem, 4vw, 4rem);
}

// Usage in a Next.js page
// app/page.js
import { FlexNav, FlexCards, FlexCenter, ResponsiveFlex } from '@/components/flex';

export default function Home() {
  return (
    <>
      {/* Navigation */}
      <FlexNav
        logo={<div className="text-xl font-bold">Logo</div>}
        links={
          <>
            <a href="#" className="hover:text-blue-500">Home</a>
            <a href="#" className="hover:text-blue-500">About</a>
            <a href="#" className="hover:text-blue-500">Contact</a>
          </>
        }
        actions={
          <button className="bg-blue-500 text-white px-4 py-2 rounded">
            Sign In
          </button>
        }
      />
      
      {/* Hero Section */}
      <FlexCenter className="min-h-[60vh] bg-gray-100">
        <div className="text-center">
          <h1 className="text-4xl font-bold mb-4">Welcome to Flex Layouts</h1>
          <p className="text-gray-600">Built with Tailwind CSS v4</p>
        </div>
      </FlexCenter>
      
      {/* Responsive Cards */}
      <ResponsiveFlex
        direction={{ base: 'col', md: 'row' }}
        gap={{ base: '4', lg: '8' }}
        className="p-8"
      >
        {[1, 2, 3].map((i) => (
          <div key={i} className="flex-1 bg-white shadow-lg rounded-lg p-6">
            <h3 className="text-xl font-semibold mb-2">Card {i}</h3>
            <p className="text-gray-600">Flexible card content that responds to screen size.</p>
          </div>
        ))}
      </ResponsiveFlex>
    </>
  );
}

// Environment variable setup for debugging
// .env.local
NEXT_PUBLIC_SHOW_FLEX_DEBUG=true

// Utility function for conditional classes (optional)
// lib/utils.js
import { clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```
Complete setup for flex layouts in Next.js with Tailwind CSS v4.

## Validation
- Components render with correct Tailwind v4 classes
- Responsive breakpoints work in Next.js
- SSR/hydration works correctly
- Dynamic class application functions
- TypeScript types are correct (if using TypeScript)

## Error Handling
- **"Hydration mismatch"** - Ensure conditional rendering is consistent
- **"Classes not applying"** - Check Tailwind v4 configuration
- **"Gap not working"** - Tailwind v4 includes gap utilities by default
- **"Component not found"** - Check import paths and exports

## Safety Notes
- Use `cn()` utility for conditional classes to avoid conflicts
- Test with Next.js SSR and client-side navigation
- Ensure responsive utilities work with Tailwind v4's new breakpoints
- Consider using TypeScript for better type safety
- Test with React DevTools and Next.js debugging

## Examples
- **Navigation component**
  ```jsx
  <FlexNav
    logo={<Logo />}
    links={<NavLinks />}
    actions={<Button>Sign In</Button>}
  />
  ```
  Responsive navigation with Tailwind v4 flex utilities

- **Responsive layout**
  ```jsx
  <ResponsiveFlex
    direction={{ base: 'col', md: 'row' }}
    justify={{ base: 'start', lg: 'between' }}
    gap={{ base: '4', xl: '8' }}
  >
    <Card />
    <Card />
    <Card />
  </ResponsiveFlex>
  ```
  Mobile-first responsive flex layout

- **Center content**
  ```jsx
  <FlexCenter className="min-h-screen">
    <div>Perfectly centered content</div>
  </FlexCenter>
  ```
  Centers content using Tailwind v4 flex utilities

- **Sidebar layout**
  ```jsx
  <FlexSidebar 
    sidebar={<Sidebar />}
    sidebarPosition="left"
  >
    <MainContent />
  </FlexSidebar>
  ```
  Flexible sidebar layout with fixed width aside