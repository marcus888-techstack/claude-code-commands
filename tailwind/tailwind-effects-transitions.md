# Tailwind Effects Transitions

## Purpose
Configure smooth transitions for CSS property changes using Tailwind transition utilities

## Context
Use to add smooth animations when CSS properties change due to hover, focus, or state changes. Controls which properties transition, duration, timing functions, and delays. Essential for creating polished, professional interfaces.

## Parameters
- `$PROPERTY` - Which properties to transition
  - Required
  - Options: `all`, `colors`, `opacity`, `shadow`, `transform`, `none`, `custom`
- `$DURATION` - Transition duration in milliseconds
  - Optional
  - Default: `300`
  - Options: `75`, `100`, `150`, `200`, `300`, `500`, `700`, `1000`
- `$TIMING` - Timing function for the transition
  - Optional
  - Default: `ease-in-out`
  - Options: `linear`, `ease-in`, `ease-out`, `ease-in-out`, `bounce`
- `$DELAY` - Delay before transition starts
  - Optional
  - Default: `0`
  - Example: `0`, `75`, `100`, `150`, `200`

## Steps

### 1. Define transition property groups
```javascript
const transitionProperties = {
  all: 'all',
  none: 'none',
  colors: 'color, background-color, border-color, text-decoration-color, fill, stroke',
  opacity: 'opacity',
  shadow: 'box-shadow',
  transform: 'transform',
  spacing: 'margin, padding',
  dimensions: 'width, height',
  position: 'top, right, bottom, left',
  custom: '' // Will be defined by user
};

const timingFunctions = {
  linear: 'linear',
  'ease-in': 'cubic-bezier(0.4, 0, 1, 1)',
  'ease-out': 'cubic-bezier(0, 0, 0.2, 1)',
  'ease-in-out': 'cubic-bezier(0.4, 0, 0.2, 1)',
  bounce: 'cubic-bezier(0.68, -0.55, 0.265, 1.55)'
};
```
Defines transition property groups and timing functions.

### 2. Apply transition configuration
```javascript
function applyTransitions(property, duration, timing, delay, selector = '*') {
  const elements = document.querySelectorAll(selector);
  const properties = transitionProperties[property] || property;
  const timingFunction = timingFunctions[timing] || timing;
  
  elements.forEach(element => {
    // Build transition value
    const transitionValue = `${properties} ${duration}ms ${timingFunction} ${delay}ms`;
    
    // Apply transition
    element.style.transition = transitionValue;
    
    // Add utility classes for identification
    element.classList.add('transition-configured');
    element.dataset.transitionProperty = property;
    element.dataset.transitionDuration = duration;
    element.dataset.transitionTiming = timing;
    element.dataset.transitionDelay = delay;
  });
  
  // Save configuration
  saveTransitionConfig(selector, property, duration, timing, delay);
}
```
Applies transition configuration to elements.

### 3. Create global transition utilities
```css
@layer utilities {
  /* Property-specific transitions */
  .transition-colors { 
    transition-property: color, background-color, border-color, text-decoration-color, fill, stroke; 
  }
  .transition-opacity { transition-property: opacity; }
  .transition-shadow { transition-property: box-shadow; }
  .transition-transform { transition-property: transform; }
  .transition-spacing { transition-property: margin, padding; }
  .transition-dimensions { transition-property: width, height; }
  
  /* Duration utilities */
  .duration-75 { transition-duration: 75ms; }
  .duration-100 { transition-duration: 100ms; }
  .duration-150 { transition-duration: 150ms; }
  .duration-200 { transition-duration: 200ms; }
  .duration-300 { transition-duration: 300ms; }
  .duration-500 { transition-duration: 500ms; }
  .duration-700 { transition-duration: 700ms; }
  .duration-1000 { transition-duration: 1000ms; }
  
  /* Timing function utilities */
  .ease-linear { transition-timing-function: linear; }
  .ease-in { transition-timing-function: cubic-bezier(0.4, 0, 1, 1); }
  .ease-out { transition-timing-function: cubic-bezier(0, 0, 0.2, 1); }
  .ease-in-out { transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); }
  .ease-bounce { transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); }
  
  /* Delay utilities */
  .delay-75 { transition-delay: 75ms; }
  .delay-100 { transition-delay: 100ms; }
  .delay-150 { transition-delay: 150ms; }
  .delay-200 { transition-delay: 200ms; }
  .delay-300 { transition-delay: 300ms; }
  .delay-500 { transition-delay: 500ms; }
}
```
Creates transition utility classes.

### 4. Add advanced transition effects
```javascript
function addAdvancedTransitions() {
  const advancedStyles = `
    /* Staggered transitions for lists */
    .transition-stagger > * {
      transition: all 300ms ease-in-out;
    }
    .transition-stagger > *:nth-child(1) { transition-delay: 0ms; }
    .transition-stagger > *:nth-child(2) { transition-delay: 50ms; }
    .transition-stagger > *:nth-child(3) { transition-delay: 100ms; }
    .transition-stagger > *:nth-child(4) { transition-delay: 150ms; }
    .transition-stagger > *:nth-child(5) { transition-delay: 200ms; }
    .transition-stagger > *:nth-child(6) { transition-delay: 250ms; }
    .transition-stagger > *:nth-child(7) { transition-delay: 300ms; }
    .transition-stagger > *:nth-child(8) { transition-delay: 350ms; }
    
    /* Smooth height transitions */
    .transition-height {
      transition: height 300ms ease-in-out;
      overflow: hidden;
    }
    .transition-max-height {
      transition: max-height 300ms ease-in-out;
      overflow: hidden;
    }
    
    /* Multi-property transitions */
    .transition-all-smooth {
      transition: 
        transform 300ms cubic-bezier(0.4, 0, 0.2, 1),
        opacity 300ms ease-in-out,
        box-shadow 300ms ease-in-out,
        background-color 300ms ease-in-out;
    }
    
    /* Spring transitions */
    .transition-spring {
      transition: all 500ms cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }
    
    /* Elastic transitions */
    .transition-elastic {
      transition: all 800ms cubic-bezier(0.68, -0.55, 0.265, 1.55);
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = advancedStyles;
  document.head.appendChild(style);
}
```
Adds advanced transition effects.

### 5. Create transition presets
```javascript
function createTransitionPresets() {
  const presets = {
    'fade': {
      property: 'opacity',
      duration: 300,
      timing: 'ease-in-out',
      delay: 0
    },
    'slide': {
      property: 'transform',
      duration: 300,
      timing: 'ease-out',
      delay: 0
    },
    'color-change': {
      property: 'colors',
      duration: 200,
      timing: 'ease-in-out',
      delay: 0
    },
    'expand': {
      property: 'dimensions',
      duration: 400,
      timing: 'ease-in-out',
      delay: 0
    },
    'bounce': {
      property: 'transform',
      duration: 500,
      timing: 'bounce',
      delay: 0
    }
  };
  
  // Apply preset classes
  Object.entries(presets).forEach(([name, config]) => {
    const elements = document.querySelectorAll(`.transition-${name}`);
    elements.forEach(element => {
      applyTransitions(
        config.property,
        config.duration,
        config.timing,
        config.delay,
        element
      );
    });
  });
  
  return presets;
}
```
Creates and applies transition presets.

### 6. Handle reduced motion preferences
```javascript
function handleReducedMotion() {
  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
  
  function updateTransitions(e) {
    if (e.matches) {
      // User prefers reduced motion
      document.documentElement.classList.add('reduce-motion');
      
      // Override transitions
      const style = document.createElement('style');
      style.textContent = `
        .reduce-motion * {
          animation-duration: 0.01ms !important;
          animation-iteration-count: 1 !important;
          transition-duration: 0.01ms !important;
          transition-delay: 0ms !important;
        }
      `;
      style.id = 'reduced-motion-overrides';
      document.head.appendChild(style);
    } else {
      // Remove reduced motion overrides
      document.documentElement.classList.remove('reduce-motion');
      const overrides = document.getElementById('reduced-motion-overrides');
      if (overrides) overrides.remove();
    }
  }
  
  // Check on load
  updateTransitions(prefersReducedMotion);
  
  // Listen for changes
  prefersReducedMotion.addEventListener('change', updateTransitions);
}
```
Respects user's motion preferences.

### 7. Create transition debugging tools
```javascript
function enableTransitionDebugging() {
  // Add visual indicators for transitions
  const debugStyles = `
    .transition-debug * {
      position: relative;
    }
    .transition-debug *::after {
      content: attr(data-transition-property) " " attr(data-transition-duration) "ms";
      position: absolute;
      top: 0;
      right: 0;
      background: rgba(255, 0, 0, 0.8);
      color: white;
      font-size: 10px;
      padding: 2px 4px;
      pointer-events: none;
      z-index: 9999;
    }
    
    /* Highlight transitioning elements */
    .transition-debug *:hover {
      outline: 2px dashed red;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = debugStyles;
  document.head.appendChild(style);
  
  // Toggle debug mode
  document.addEventListener('keydown', (e) => {
    if (e.ctrlKey && e.shiftKey && e.key === 'T') {
      document.body.classList.toggle('transition-debug');
    }
  });
}
```
Adds debugging tools for transitions.

### 8. Apply and initialize
```javascript
// Apply transition configuration
applyTransitions('$PROPERTY', parseInt('$DURATION'), '$TIMING', parseInt('$DELAY'));

// Add advanced transitions
addAdvancedTransitions();

// Create presets
createTransitionPresets();

// Handle reduced motion
handleReducedMotion();

// Enable debugging in development
if (window.location.hostname === 'localhost') {
  enableTransitionDebugging();
}

// Save configuration
function saveTransitionConfig(selector, property, duration, timing, delay) {
  const config = JSON.parse(localStorage.getItem('transition-config') || '{}');
  config[selector] = { property, duration, timing, delay };
  localStorage.setItem('transition-config', JSON.stringify(config));
}

// Load saved configuration
document.addEventListener('DOMContentLoaded', () => {
  const config = JSON.parse(localStorage.getItem('transition-config') || '{}');
  Object.entries(config).forEach(([selector, settings]) => {
    applyTransitions(
      settings.property,
      settings.duration,
      settings.timing,
      settings.delay,
      selector
    );
  });
});
```
Completes transition setup and initialization.

## Validation
- Transitions apply to property changes
- Duration and timing are correct
- Delays work as expected
- Reduced motion preference is respected
- Debug mode shows transition info

## Error Handling
- **"Transition not smooth"** - Check conflicting transitions
- **"Property not animating"** - Verify property is animatable
- **"Performance issues"** - Reduce number of transitioning elements
- **"Timing looks wrong"** - Try different timing functions

## Safety Notes
- Test performance on lower-end devices
- Respect prefers-reduced-motion
- Avoid transitioning expensive properties
- Don't transition layout properties that cause reflow
- Test with various content lengths

## Examples
- **Apply color transitions globally**
  ```
  tailwind-effects-transitions colors 300 ease-in-out 0
  ```
  Smooth color changes for all elements

- **Apply transform transitions to cards**
  ```
  tailwind-effects-transitions transform 500 bounce 0 .card
  ```
  Bouncy transform transitions for cards

- **Apply opacity fade with delay**
  ```
  tailwind-effects-transitions opacity 200 ease-out 100
  ```
  Fade transitions with 100ms delay

- **Remove all transitions**
  ```
  tailwind-effects-transitions none 0 linear 0
  ```
  Disables all transitions