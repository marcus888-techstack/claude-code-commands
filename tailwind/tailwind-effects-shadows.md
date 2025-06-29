# Tailwind Effects Shadows

## Purpose
Apply and customize shadow effects to elements using Tailwind CSS shadow utilities

## Context
Use to add depth and visual hierarchy to UI elements with box shadows. Supports standard shadows, colored shadows, inset shadows, and custom shadow effects. Essential for creating modern, layered interfaces with proper elevation.

## Parameters
- `$SHADOW_TYPE` - Type of shadow to apply
  - Required
  - Options: `standard`, `colored`, `inset`, `glow`, `layered`, `custom`
- `$SHADOW_SIZE` - Size of the shadow effect
  - Optional
  - Default: `md`
  - Options: `sm`, `md`, `lg`, `xl`, `2xl`, `none`
- `$SHADOW_COLOR` - Color for colored shadows
  - Optional
  - Default: `gray`
  - Example: `blue`, `primary`, `#3B82F6`
- `$SELECTOR` - CSS selector for target elements
  - Optional
  - Default: `.shadow`
  - Example: `.card`, `button`, `.elevated`

## Steps

### 1. Define shadow presets
```javascript
const shadowPresets = {
  standard: {
    sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
    lg: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
    xl: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
    '2xl': '0 25px 50px -12px rgb(0 0 0 / 0.25)'
  },
  
  colored: {
    sm: '0 1px 2px 0 var(--shadow-color)',
    md: '0 4px 6px -1px var(--shadow-color), 0 2px 4px -2px var(--shadow-color)',
    lg: '0 10px 15px -3px var(--shadow-color), 0 4px 6px -4px var(--shadow-color)',
    xl: '0 20px 25px -5px var(--shadow-color), 0 8px 10px -6px var(--shadow-color)',
    '2xl': '0 25px 50px -12px var(--shadow-color)'
  },
  
  inset: {
    sm: 'inset 0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md: 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
    lg: 'inset 0 4px 6px -1px rgb(0 0 0 / 0.1)',
    xl: 'inset 0 8px 10px -2px rgb(0 0 0 / 0.1)',
    '2xl': 'inset 0 12px 16px -4px rgb(0 0 0 / 0.1)'
  },
  
  glow: {
    sm: '0 0 3px var(--glow-color)',
    md: '0 0 6px var(--glow-color)',
    lg: '0 0 12px var(--glow-color)',
    xl: '0 0 24px var(--glow-color)',
    '2xl': '0 0 48px var(--glow-color)'
  },
  
  layered: {
    sm: '0 1px 1px rgb(0 0 0 / 0.1), 0 2px 2px rgb(0 0 0 / 0.06)',
    md: '0 2px 2px rgb(0 0 0 / 0.1), 0 4px 4px rgb(0 0 0 / 0.06), 0 8px 8px rgb(0 0 0 / 0.04)',
    lg: '0 4px 4px rgb(0 0 0 / 0.1), 0 8px 8px rgb(0 0 0 / 0.06), 0 16px 16px rgb(0 0 0 / 0.04)',
    xl: '0 8px 8px rgb(0 0 0 / 0.1), 0 16px 16px rgb(0 0 0 / 0.06), 0 32px 32px rgb(0 0 0 / 0.04)',
    '2xl': '0 16px 16px rgb(0 0 0 / 0.1), 0 32px 32px rgb(0 0 0 / 0.06), 0 64px 64px rgb(0 0 0 / 0.04)'
  }
};
```
Defines various shadow preset styles.

### 2. Apply shadow effects
```javascript
function applyShadowEffect(selector, type, size, color) {
  const elements = document.querySelectorAll(selector);
  const shadowConfig = shadowPresets[type];
  
  if (!shadowConfig) {
    console.error(`Shadow type '${type}' not found`);
    return;
  }
  
  const shadowValue = shadowConfig[size] || shadowConfig.md;
  
  elements.forEach(element => {
    // Remove existing shadow classes
    element.className = element.className
      .split(' ')
      .filter(cls => !cls.includes('shadow'))
      .join(' ');
    
    // Apply shadow based on type
    if (type === 'colored' || type === 'glow') {
      // Set color variable
      const colorValue = getColorValue(color);
      element.style.setProperty('--shadow-color', colorValue);
      element.style.setProperty('--glow-color', colorValue);
    }
    
    // Apply shadow
    element.style.boxShadow = shadowValue;
    
    // Add utility class for identification
    element.classList.add(`shadow-${type}-${size}`);
  });
  
  // Save preferences
  saveShadowPreferences(selector, type, size, color);
}
```
Applies shadow effects to elements.

### 3. Get color values
```javascript
function getColorValue(color) {
  // Check if it's a hex color
  if (color.startsWith('#')) {
    return color;
  }
  
  // Check if it's an RGB/RGBA value
  if (color.includes('rgb')) {
    return color;
  }
  
  // Check Tailwind color palette
  const tailwindColors = {
    gray: 'rgb(107 114 128 / 0.1)',
    red: 'rgb(239 68 68 / 0.1)',
    blue: 'rgb(59 130 246 / 0.1)',
    green: 'rgb(34 197 94 / 0.1)',
    yellow: 'rgb(250 204 21 / 0.1)',
    purple: 'rgb(147 51 234 / 0.1)',
    pink: 'rgb(236 72 153 / 0.1)',
    primary: 'rgb(var(--color-primary-500) / 0.1)',
    secondary: 'rgb(var(--color-secondary-500) / 0.1)'
  };
  
  return tailwindColors[color] || tailwindColors.gray;
}
```
Converts color names to values.

### 4. Create shadow utilities
```css
/* Shadow utility classes */
@layer utilities {
  /* Elevation classes */
  .elevation-1 { box-shadow: var(--shadow-sm); }
  .elevation-2 { box-shadow: var(--shadow-md); }
  .elevation-3 { box-shadow: var(--shadow-lg); }
  .elevation-4 { box-shadow: var(--shadow-xl); }
  .elevation-5 { box-shadow: var(--shadow-2xl); }
  
  /* Smooth shadow transitions */
  .shadow-transition {
    transition: box-shadow 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }
  
  /* Hover shadow effects */
  .shadow-hover-sm:hover { box-shadow: var(--shadow-sm); }
  .shadow-hover-md:hover { box-shadow: var(--shadow-md); }
  .shadow-hover-lg:hover { box-shadow: var(--shadow-lg); }
  .shadow-hover-xl:hover { box-shadow: var(--shadow-xl); }
  
  /* Focus shadow effects */
  .shadow-focus:focus {
    box-shadow: 0 0 0 3px var(--color-primary-500 / 0.1);
  }
}
```
Defines shadow utility classes.

### 5. Add advanced shadow effects
```javascript
function addAdvancedShadows() {
  const advancedStyles = `
    /* Neumorphism shadows */
    .shadow-neumorphic {
      background: var(--color-gray-100);
      box-shadow: 
        6px 6px 12px rgb(0 0 0 / 0.1),
        -6px -6px 12px rgb(255 255 255 / 0.7);
    }
    .shadow-neumorphic-inset {
      background: var(--color-gray-100);
      box-shadow: 
        inset 6px 6px 12px rgb(0 0 0 / 0.1),
        inset -6px -6px 12px rgb(255 255 255 / 0.7);
    }
    
    /* Long shadows */
    .shadow-long {
      box-shadow: 
        1px 1px 0 rgb(0 0 0 / 0.05),
        2px 2px 0 rgb(0 0 0 / 0.05),
        3px 3px 0 rgb(0 0 0 / 0.05),
        4px 4px 0 rgb(0 0 0 / 0.05),
        5px 5px 0 rgb(0 0 0 / 0.05),
        6px 6px 12px rgb(0 0 0 / 0.1);
    }
    
    /* Material Design shadows */
    .shadow-material-1 {
      box-shadow: 
        0 1px 3px rgba(0,0,0,0.12), 
        0 1px 2px rgba(0,0,0,0.24);
    }
    .shadow-material-2 {
      box-shadow: 
        0 3px 6px rgba(0,0,0,0.16), 
        0 3px 6px rgba(0,0,0,0.23);
    }
    .shadow-material-3 {
      box-shadow: 
        0 10px 20px rgba(0,0,0,0.19), 
        0 6px 6px rgba(0,0,0,0.23);
    }
    
    /* Gradient shadows */
    .shadow-gradient::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: inherit;
      filter: blur(20px) opacity(0.5);
      transform: translateY(10px);
      z-index: -1;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = advancedStyles;
  document.head.appendChild(style);
}
```
Adds advanced shadow effect styles.

### 6. Create responsive shadows
```javascript
function applyResponsiveShadows(selector) {
  const breakpoints = {
    sm: 640,
    md: 768,
    lg: 1024,
    xl: 1280
  };
  
  const shadowSizes = {
    sm: 'sm',
    md: 'md',
    lg: 'lg',
    xl: 'xl'
  };
  
  function updateShadows() {
    const width = window.innerWidth;
    let targetSize = 'sm';
    
    Object.entries(breakpoints).forEach(([breakpoint, minWidth]) => {
      if (width >= minWidth) {
        targetSize = shadowSizes[breakpoint];
      }
    });
    
    applyShadowEffect(selector, 'standard', targetSize);
  }
  
  // Initial update
  updateShadows();
  
  // Update on resize
  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(updateShadows, 250);
  });
}
```
Implements responsive shadow sizing.

### 7. Apply interactive shadows
```javascript
function applyInteractiveShadows(selector) {
  const elements = document.querySelectorAll(selector);
  
  elements.forEach(element => {
    // Add transition
    element.classList.add('shadow-transition');
    
    // Mouse enter - increase shadow
    element.addEventListener('mouseenter', () => {
      const currentSize = element.dataset.shadowSize || 'md';
      const sizes = ['sm', 'md', 'lg', 'xl', '2xl'];
      const currentIndex = sizes.indexOf(currentSize);
      const nextSize = sizes[Math.min(currentIndex + 1, sizes.length - 1)];
      
      element.style.boxShadow = shadowPresets.standard[nextSize];
    });
    
    // Mouse leave - restore shadow
    element.addEventListener('mouseleave', () => {
      const originalSize = element.dataset.shadowSize || 'md';
      element.style.boxShadow = shadowPresets.standard[originalSize];
    });
    
    // Store original size
    element.dataset.shadowSize = 'md';
  });
}
```
Adds interactive shadow effects.

### 8. Apply and persist shadows
```javascript
// Apply shadow effect
applyShadowEffect('$SELECTOR', '$SHADOW_TYPE', '$SHADOW_SIZE', '$SHADOW_COLOR');

// Add advanced shadows
addAdvancedShadows();

// Apply interactive shadows if requested
if ('$SHADOW_TYPE' === 'standard') {
  applyInteractiveShadows('$SELECTOR');
}

// Save and load preferences
function saveShadowPreferences(selector, type, size, color) {
  const preferences = JSON.parse(localStorage.getItem('shadow-preferences') || '{}');
  preferences[selector] = { type, size, color };
  localStorage.setItem('shadow-preferences', JSON.stringify(preferences));
}

// Load preferences on page load
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('shadow-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyShadowEffect(selector, config.type, config.size, config.color);
  });
});
```
Completes shadow application and persistence.

## Validation
- Shadows appear on target elements
- Shadow size and type are correct
- Colors apply to colored shadows
- Interactive effects work smoothly
- Shadows persist on page reload

## Error Handling
- **"Shadow not visible"** - Check z-index and overflow
- **"Invalid shadow type"** - Use available shadow types
- **"Color not applying"** - Verify color format
- **"Performance issues"** - Reduce number of layered shadows

## Safety Notes
- Test shadow performance on mobile
- Ensure shadows don't obscure content
- Check accessibility with high contrast mode
- Avoid excessive shadow layers
- Consider reduced motion preferences

## Examples
- **Apply standard shadow to cards**
  ```
  tailwind-effects-shadows standard lg gray .card
  ```
  Applies large standard shadow to card elements

- **Apply colored glow effect**
  ```
  tailwind-effects-shadows glow xl blue ".glow-button"
  ```
  Applies blue glow effect to buttons

- **Apply inset shadow**
  ```
  tailwind-effects-shadows inset md gray "input, textarea"
  ```
  Applies inset shadows to form inputs

- **Apply layered shadows**
  ```
  tailwind-effects-shadows layered xl gray .elevated
  ```
  Applies multi-layer shadow for depth effect