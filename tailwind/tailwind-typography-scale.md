# Tailwind Typography Scale

## Purpose
Change text sizes across your application using Tailwind's typography scale system

## Context
Use to adjust the overall text size scale for better readability, accessibility, or design requirements. Can apply different scales for desktop and mobile, or adjust specific text size categories while maintaining proportional relationships.

## Parameters
- `$SCALE` - Typography scale to apply
  - Required
  - Options: `xs`, `sm`, `base`, `lg`, `xl`, `2xl`, `custom`
- `$SCALE_FACTOR` - Custom scale multiplier (if scale is custom)
  - Optional
  - Default: `1.0`
  - Example: `0.875`, `1.125`, `1.25`
- `$APPLY_TO` - Which text elements to scale
  - Optional
  - Default: `all`
  - Options: `all`, `headings`, `body`, `ui`
- `$RESPONSIVE` - Apply different scales for breakpoints
  - Optional
  - Default: `no`
  - Options: `yes`, `no`

## Steps

### 1. Define typography scale presets
```javascript
const typographyScales = {
  xs: {
    factor: 0.75,
    base: '0.75rem',  // 12px
    scale: {
      xs: '0.625rem',   // 10px
      sm: '0.6875rem',  // 11px
      base: '0.75rem',  // 12px
      lg: '0.875rem',   // 14px
      xl: '1rem',       // 16px
      '2xl': '1.125rem', // 18px
      '3xl': '1.375rem', // 22px
      '4xl': '1.625rem', // 26px
      '5xl': '2rem',    // 32px
    }
  },
  sm: {
    factor: 0.875,
    base: '0.875rem', // 14px
    scale: {
      xs: '0.75rem',    // 12px
      sm: '0.8125rem',  // 13px
      base: '0.875rem', // 14px
      lg: '1rem',       // 16px
      xl: '1.125rem',   // 18px
      '2xl': '1.375rem', // 22px
      '3xl': '1.625rem', // 26px
      '4xl': '2rem',    // 32px
      '5xl': '2.5rem',  // 40px
    }
  },
  base: {
    factor: 1.0,
    base: '1rem',     // 16px
    scale: {
      xs: '0.75rem',    // 12px
      sm: '0.875rem',   // 14px
      base: '1rem',     // 16px
      lg: '1.125rem',   // 18px
      xl: '1.25rem',    // 20px
      '2xl': '1.5rem',  // 24px
      '3xl': '1.875rem', // 30px
      '4xl': '2.25rem', // 36px
      '5xl': '3rem',    // 48px
    }
  },
  lg: {
    factor: 1.125,
    base: '1.125rem', // 18px
    scale: {
      xs: '0.875rem',   // 14px
      sm: '1rem',       // 16px
      base: '1.125rem', // 18px
      lg: '1.25rem',    // 20px
      xl: '1.5rem',     // 24px
      '2xl': '1.875rem', // 30px
      '3xl': '2.25rem', // 36px
      '4xl': '3rem',    // 48px
      '5xl': '3.75rem', // 60px
    }
  }
};
```
Defines preset typography scales.

### 2. Apply scale to CSS variables
```javascript
function applyTypographyScale(scale, customFactor = 1.0, applyTo = 'all') {
  const root = document.documentElement;
  let scaleConfig;
  
  if (scale === 'custom') {
    // Generate custom scale based on factor
    scaleConfig = generateCustomScale(customFactor);
  } else {
    scaleConfig = typographyScales[scale] || typographyScales.base;
  }
  
  // Apply scale based on target
  if (applyTo === 'all' || applyTo === 'body') {
    Object.entries(scaleConfig.scale).forEach(([size, value]) => {
      root.style.setProperty(`--text-${size}`, value);
    });
  }
  
  if (applyTo === 'all' || applyTo === 'headings') {
    // Apply larger scale to headings
    root.style.setProperty('--heading-scale', scaleConfig.factor);
    root.style.setProperty('--text-h6', `calc(${scaleConfig.scale.lg} * 1.1)`);
    root.style.setProperty('--text-h5', `calc(${scaleConfig.scale.xl} * 1.1)`);
    root.style.setProperty('--text-h4', `calc(${scaleConfig.scale['2xl']} * 1.1)`);
    root.style.setProperty('--text-h3', `calc(${scaleConfig.scale['3xl']} * 1.1)`);
    root.style.setProperty('--text-h2', `calc(${scaleConfig.scale['4xl']} * 1.1)`);
    root.style.setProperty('--text-h1', `calc(${scaleConfig.scale['5xl']} * 1.1)`);
  }
  
  if (applyTo === 'all' || applyTo === 'ui') {
    // UI elements often need specific sizes
    root.style.setProperty('--text-ui-xs', scaleConfig.scale.xs);
    root.style.setProperty('--text-ui-sm', scaleConfig.scale.sm);
    root.style.setProperty('--text-ui-base', scaleConfig.scale.base);
  }
  
  // Save preference
  localStorage.setItem('typography-scale', JSON.stringify({
    scale: scale,
    factor: customFactor,
    applyTo: applyTo
  }));
}
```
Applies the selected typography scale.

### 3. Generate custom scale
```javascript
function generateCustomScale(factor) {
  const baseSize = 16; // Base pixel size
  const ratio = 1.25; // Scale ratio between sizes
  
  const scale = {
    xs: `${(baseSize * factor * Math.pow(ratio, -2)) / 16}rem`,
    sm: `${(baseSize * factor * Math.pow(ratio, -1)) / 16}rem`,
    base: `${(baseSize * factor) / 16}rem`,
    lg: `${(baseSize * factor * ratio) / 16}rem`,
    xl: `${(baseSize * factor * Math.pow(ratio, 2)) / 16}rem`,
    '2xl': `${(baseSize * factor * Math.pow(ratio, 3)) / 16}rem`,
    '3xl': `${(baseSize * factor * Math.pow(ratio, 4)) / 16}rem`,
    '4xl': `${(baseSize * factor * Math.pow(ratio, 5)) / 16}rem`,
    '5xl': `${(baseSize * factor * Math.pow(ratio, 6)) / 16}rem`,
  };
  
  return {
    factor: factor,
    base: scale.base,
    scale: scale
  };
}
```
Creates custom scale based on factor.

### 4. Update Tailwind utilities
```css
/* Override Tailwind text sizes with CSS variables */
@layer utilities {
  .text-xs { font-size: var(--text-xs, 0.75rem); }
  .text-sm { font-size: var(--text-sm, 0.875rem); }
  .text-base { font-size: var(--text-base, 1rem); }
  .text-lg { font-size: var(--text-lg, 1.125rem); }
  .text-xl { font-size: var(--text-xl, 1.25rem); }
  .text-2xl { font-size: var(--text-2xl, 1.5rem); }
  .text-3xl { font-size: var(--text-3xl, 1.875rem); }
  .text-4xl { font-size: var(--text-4xl, 2.25rem); }
  .text-5xl { font-size: var(--text-5xl, 3rem); }
  
  /* Heading utilities */
  h1, .h1 { font-size: var(--text-h1, 2.5rem); }
  h2, .h2 { font-size: var(--text-h2, 2rem); }
  h3, .h3 { font-size: var(--text-h3, 1.75rem); }
  h4, .h4 { font-size: var(--text-h4, 1.5rem); }
  h5, .h5 { font-size: var(--text-h5, 1.25rem); }
  h6, .h6 { font-size: var(--text-h6, 1.125rem); }
}
```
Maps Tailwind classes to CSS variables.

### 5. Apply responsive scales
```javascript
function applyResponsiveScale() {
  if ('$RESPONSIVE' !== 'yes') return;
  
  // Define breakpoint scales
  const breakpointScales = {
    'sm': { min: 640, scale: 'sm' },
    'md': { min: 768, scale: 'base' },
    'lg': { min: 1024, scale: 'base' },
    'xl': { min: 1280, scale: 'lg' },
    '2xl': { min: 1536, scale: 'lg' }
  };
  
  // Apply scale based on viewport
  function updateScale() {
    const width = window.innerWidth;
    let targetScale = 'sm'; // Mobile default
    
    for (const [breakpoint, config] of Object.entries(breakpointScales)) {
      if (width >= config.min) {
        targetScale = config.scale;
      }
    }
    
    applyTypographyScale(targetScale, 1.0, '$APPLY_TO');
  }
  
  // Initial update
  updateScale();
  
  // Update on resize
  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(updateScale, 250);
  });
}
```
Implements responsive typography scaling.

### 6. Add line height adjustments
```javascript
function adjustLineHeights(scale) {
  const root = document.documentElement;
  
  // Line height ratios based on scale
  const lineHeightRatios = {
    xs: 1.6,
    sm: 1.5,
    base: 1.5,
    lg: 1.4,
    xl: 1.3
  };
  
  const ratio = lineHeightRatios[scale] || 1.5;
  
  // Apply line heights
  root.style.setProperty('--leading-none', '1');
  root.style.setProperty('--leading-tight', `${ratio * 0.85}`);
  root.style.setProperty('--leading-snug', `${ratio * 0.925}`);
  root.style.setProperty('--leading-normal', `${ratio}`);
  root.style.setProperty('--leading-relaxed', `${ratio * 1.1}`);
  root.style.setProperty('--leading-loose', `${ratio * 1.2}`);
}
```
Adjusts line heights to match scale.

### 7. Apply the scale
```javascript
// Apply the typography scale
applyTypographyScale('$SCALE', parseFloat('$SCALE_FACTOR'), '$APPLY_TO');

// Adjust line heights
adjustLineHeights('$SCALE');

// Apply responsive scaling if enabled
applyResponsiveScale();

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const saved = JSON.parse(localStorage.getItem('typography-scale') || '{}');
  if (saved.scale) {
    applyTypographyScale(saved.scale, saved.factor || 1.0, saved.applyTo || 'all');
  }
});
```
Executes the typography scale changes.

## Validation
- Text sizes update across the application
- Proportional relationships maintained
- Heading hierarchy preserved
- Line heights adjust appropriately
- Responsive scales apply at breakpoints

## Error Handling
- **"Invalid scale"** - Use xs, sm, base, lg, xl, or custom
- **"Scale not applying"** - Check CSS variable support
- **"Responsive not working"** - Verify breakpoint values
- **"Text too small/large"** - Adjust scale factor

## Safety Notes
- Test readability at all scales
- Ensure WCAG compliance for font sizes
- Check text doesn't overflow containers
- Test on various devices and screens
- Provide user control for accessibility

## Examples
- **Apply large scale globally**
  ```
  tailwind-typography-scale lg 1.0 all no
  ```
  Increases all text sizes to large scale

- **Apply custom scale to headings only**
  ```
  tailwind-typography-scale custom 1.2 headings no
  ```
  Increases heading sizes by 20%

- **Apply responsive scaling**
  ```
  tailwind-typography-scale base 1.0 all yes
  ```
  Uses different scales for different screen sizes

- **Apply small scale to UI elements**
  ```
  tailwind-typography-scale sm 1.0 ui no
  ```
  Reduces UI text sizes for compact interfaces