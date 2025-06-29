# Tailwind Layout Container

## Purpose
Configure container settings for consistent page layout and content width across your application

## Context
Use to establish maximum content widths, responsive breakpoints, and centered layouts. The container utility helps maintain readable line lengths and consistent spacing on different screen sizes. Essential for creating professional, responsive layouts.

## Parameters
- `$CONTAINER_TYPE` - Type of container configuration
  - Required
  - Options: `default`, `fluid`, `narrow`, `wide`, `full`, `custom`
- `$CENTER` - Whether to center the container
  - Optional
  - Default: `yes`
  - Options: `yes`, `no`
- `$PADDING` - Horizontal padding for the container
  - Optional
  - Default: `1rem`
  - Example: `1rem`, `2rem`, `20px`
- `$BREAKPOINTS` - Apply responsive width changes
  - Optional
  - Default: `yes`
  - Options: `yes`, `no`

## Steps

### 1. Define container presets
```javascript
const containerPresets = {
  default: {
    maxWidth: {
      'DEFAULT': '100%',
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px'
    }
  },
  
  fluid: {
    maxWidth: {
      'DEFAULT': '100%',
      'sm': '100%',
      'md': '100%',
      'lg': '100%',
      'xl': '100%',
      '2xl': '100%'
    }
  },
  
  narrow: {
    maxWidth: {
      'DEFAULT': '100%',
      'sm': '640px',
      'md': '768px',
      'lg': '800px',
      'xl': '900px',
      '2xl': '1000px'
    }
  },
  
  wide: {
    maxWidth: {
      'DEFAULT': '100%',
      'sm': '640px',
      'md': '768px',
      'lg': '1200px',
      'xl': '1400px',
      '2xl': '1600px'
    }
  },
  
  full: {
    maxWidth: {
      'DEFAULT': '100%',
      'sm': '100%',
      'md': '100%',
      'lg': '100%',
      'xl': '100%',
      '2xl': '100%'
    }
  }
};
```
Defines container width presets.

### 2. Apply container configuration
```javascript
function applyContainerConfig(type, center, padding, breakpoints) {
  const preset = containerPresets[type] || containerPresets.default;
  const root = document.documentElement;
  
  // Apply CSS variables for container settings
  Object.entries(preset.maxWidth).forEach(([breakpoint, width]) => {
    const varName = breakpoint === 'DEFAULT' 
      ? '--container-max-width' 
      : `--container-max-width-${breakpoint}`;
    root.style.setProperty(varName, width);
  });
  
  // Apply padding
  root.style.setProperty('--container-padding-x', padding);
  
  // Apply centering
  root.style.setProperty('--container-margin', center === 'yes' ? '0 auto' : '0');
  
  // Create container styles
  createContainerStyles(preset, center, padding, breakpoints);
  
  // Save preferences
  saveContainerPreferences(type, center, padding, breakpoints);
}
```
Applies container configuration.

### 3. Create container styles
```javascript
function createContainerStyles(preset, center, padding, breakpoints) {
  let css = `
    /* Base container styles */
    .container {
      width: 100%;
      max-width: var(--container-max-width, 100%);
      margin: var(--container-margin, 0 auto);
      padding-left: var(--container-padding-x, 1rem);
      padding-right: var(--container-padding-x, 1rem);
    }
    
    /* Container variants */
    .container-fluid {
      max-width: 100%;
    }
    
    .container-narrow {
      max-width: min(var(--container-max-width), 65ch);
    }
    
    .container-wide {
      max-width: calc(var(--container-max-width) * 1.2);
    }
    
    .container-full {
      max-width: 100%;
      padding-left: 0;
      padding-right: 0;
    }
    
    /* No padding variant */
    .container-no-padding {
      padding-left: 0;
      padding-right: 0;
    }
  `;
  
  // Add responsive breakpoints
  if (breakpoints === 'yes') {
    const breakpointStyles = `
      /* Responsive container widths */
      @media (min-width: 640px) {
        .container {
          max-width: var(--container-max-width-sm, 640px);
        }
      }
      
      @media (min-width: 768px) {
        .container {
          max-width: var(--container-max-width-md, 768px);
        }
      }
      
      @media (min-width: 1024px) {
        .container {
          max-width: var(--container-max-width-lg, 1024px);
        }
      }
      
      @media (min-width: 1280px) {
        .container {
          max-width: var(--container-max-width-xl, 1280px);
        }
      }
      
      @media (min-width: 1536px) {
        .container {
          max-width: var(--container-max-width-2xl, 1536px);
        }
      }
    `;
    css += breakpointStyles;
  }
  
  // Apply styles
  const styleId = 'container-styles';
  let style = document.getElementById(styleId);
  if (!style) {
    style = document.createElement('style');
    style.id = styleId;
    document.head.appendChild(style);
  }
  style.textContent = css;
}
```
Creates container CSS styles.

### 4. Add container utilities
```javascript
function addContainerUtilities() {
  const utilities = `
    /* Container position utilities */
    .container-left {
      margin-left: 0;
      margin-right: auto;
    }
    
    .container-right {
      margin-left: auto;
      margin-right: 0;
    }
    
    .container-center {
      margin-left: auto;
      margin-right: auto;
    }
    
    /* Container padding utilities */
    .container-px-0 { padding-left: 0; padding-right: 0; }
    .container-px-4 { padding-left: 1rem; padding-right: 1rem; }
    .container-px-6 { padding-left: 1.5rem; padding-right: 1.5rem; }
    .container-px-8 { padding-left: 2rem; padding-right: 2rem; }
    .container-px-12 { padding-left: 3rem; padding-right: 3rem; }
    .container-px-16 { padding-left: 4rem; padding-right: 4rem; }
    
    /* Responsive padding */
    @media (min-width: 640px) {
      .sm\\:container-px-0 { padding-left: 0; padding-right: 0; }
      .sm\\:container-px-4 { padding-left: 1rem; padding-right: 1rem; }
      .sm\\:container-px-6 { padding-left: 1.5rem; padding-right: 1.5rem; }
      .sm\\:container-px-8 { padding-left: 2rem; padding-right: 2rem; }
    }
    
    @media (min-width: 768px) {
      .md\\:container-px-0 { padding-left: 0; padding-right: 0; }
      .md\\:container-px-4 { padding-left: 1rem; padding-right: 1rem; }
      .md\\:container-px-6 { padding-left: 1.5rem; padding-right: 1.5rem; }
      .md\\:container-px-8 { padding-left: 2rem; padding-right: 2rem; }
    }
    
    @media (min-width: 1024px) {
      .lg\\:container-px-0 { padding-left: 0; padding-right: 0; }
      .lg\\:container-px-4 { padding-left: 1rem; padding-right: 1rem; }
      .lg\\:container-px-6 { padding-left: 1.5rem; padding-right: 1.5rem; }
      .lg\\:container-px-8 { padding-left: 2rem; padding-right: 2rem; }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = utilities;
  document.head.appendChild(style);
}
```
Adds container utility classes.

### 5. Create nested container styles
```javascript
function createNestedContainerStyles() {
  const nestedStyles = `
    /* Nested containers */
    .container .container {
      padding-left: 0;
      padding-right: 0;
    }
    
    /* Container with sidebar layout */
    .container-with-sidebar {
      display: grid;
      grid-template-columns: 1fr;
      gap: 2rem;
    }
    
    @media (min-width: 1024px) {
      .container-with-sidebar {
        grid-template-columns: 300px 1fr;
      }
      
      .container-with-sidebar-right {
        grid-template-columns: 1fr 300px;
      }
    }
    
    /* Container sections */
    .container-section {
      padding-top: 4rem;
      padding-bottom: 4rem;
    }
    
    .container-section-sm {
      padding-top: 2rem;
      padding-bottom: 2rem;
    }
    
    .container-section-lg {
      padding-top: 6rem;
      padding-bottom: 6rem;
    }
    
    /* Container with max width constraint */
    .container-prose {
      max-width: 65ch;
      margin-left: auto;
      margin-right: auto;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = nestedStyles;
  document.head.appendChild(style);
}
```
Creates nested container styles.

### 6. Add container debugging
```javascript
function addContainerDebugging() {
  const debugStyles = `
    /* Container debugging */
    .container-debug {
      position: relative;
    }
    
    .container-debug::before {
      content: '';
      position: absolute;
      inset: 0;
      border: 2px dashed rgba(255, 0, 0, 0.5);
      pointer-events: none;
    }
    
    .container-debug::after {
      content: attr(data-container-info);
      position: absolute;
      top: 0;
      right: 0;
      background: rgba(255, 0, 0, 0.8);
      color: white;
      font-size: 12px;
      padding: 2px 8px;
      pointer-events: none;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = debugStyles;
  document.head.appendChild(style);
  
  // Update container info on resize
  function updateContainerInfo() {
    document.querySelectorAll('.container-debug').forEach(container => {
      const width = container.offsetWidth;
      const maxWidth = window.getComputedStyle(container).maxWidth;
      container.dataset.containerInfo = `${width}px / ${maxWidth}`;
    });
  }
  
  window.addEventListener('resize', updateContainerInfo);
  updateContainerInfo();
}
```
Adds container debugging features.

### 7. Create responsive container examples
```javascript
function createContainerExamples() {
  const examples = `
    <!-- Basic container -->
    <div class="container">
      <h1>Centered content with responsive max-width</h1>
    </div>
    
    <!-- Fluid container -->
    <div class="container container-fluid">
      <h1>Full-width fluid container</h1>
    </div>
    
    <!-- Narrow container for prose -->
    <article class="container container-narrow">
      <h1>Narrow container for better readability</h1>
      <p>Long-form content goes here...</p>
    </article>
    
    <!-- Container with custom padding -->
    <div class="container container-px-8 lg:container-px-16">
      <h1>Container with responsive padding</h1>
    </div>
    
    <!-- Nested containers -->
    <div class="container">
      <header>Header in container</header>
      <div class="container-full">
        <img src="full-width-image.jpg" alt="Full width within container">
      </div>
      <article>Content continues in container</article>
    </div>
  `;
  
  // Store examples for reference
  window.containerExamples = examples;
}
```
Creates container usage examples.

### 8. Initialize container system
```javascript
// Apply container configuration
applyContainerConfig('$CONTAINER_TYPE', '$CENTER', '$PADDING', '$BREAKPOINTS');

// Add utilities
addContainerUtilities();

// Create nested styles
createNestedContainerStyles();

// Add debugging (development only)
if (window.location.hostname === 'localhost') {
  addContainerDebugging();
}

// Create examples
createContainerExamples();

// Save preferences
function saveContainerPreferences(type, center, padding, breakpoints) {
  const preferences = {
    type: type,
    center: center,
    padding: padding,
    breakpoints: breakpoints
  };
  localStorage.setItem('container-preferences', JSON.stringify(preferences));
}

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const saved = JSON.parse(localStorage.getItem('container-preferences') || '{}');
  if (saved.type) {
    applyContainerConfig(
      saved.type,
      saved.center || 'yes',
      saved.padding || '1rem',
      saved.breakpoints || 'yes'
    );
  }
  
  // Apply container classes to existing elements
  document.querySelectorAll('[data-container]').forEach(element => {
    const containerType = element.dataset.container;
    element.classList.add('container');
    if (containerType !== 'default') {
      element.classList.add(`container-${containerType}`);
    }
  });
});
```
Completes container initialization.

## Validation
- Container widths apply correctly
- Responsive breakpoints work
- Centering functions properly
- Padding is consistent
- Nested containers behave correctly

## Error Handling
- **"Container too narrow"** - Check max-width settings
- **"Not centered"** - Verify margin auto is applied
- **"Padding missing"** - Check padding values
- **"Breakpoints not working"** - Verify media queries

## Safety Notes
- Test on various screen sizes
- Ensure readable line lengths
- Check horizontal scrolling
- Verify padding on mobile
- Test with RTL languages

## Examples
- **Apply default container**
  ```
  tailwind-layout-container default yes 1rem yes
  ```
  Standard responsive container with padding

- **Create fluid container**
  ```
  tailwind-layout-container fluid yes 2rem no
  ```
  Full-width container with 2rem padding

- **Narrow prose container**
  ```
  tailwind-layout-container narrow yes 1.5rem yes
  ```
  Narrow container for readable text

- **Wide container without center**
  ```
  tailwind-layout-container wide no 1rem yes
  ```
  Wide container aligned to left