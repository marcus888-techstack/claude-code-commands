# Tailwind Effects Filters

## Purpose
Apply CSS filter effects like blur, brightness, contrast, and grayscale to elements using Tailwind utilities

## Context
Use to modify the visual appearance of elements with image-like effects. Perfect for hover states, image galleries, overlays, and creating atmospheric UI effects. Supports all modern CSS filter functions.

## Parameters
- `$FILTER_TYPE` - Type of filter to apply
  - Required
  - Options: `blur`, `brightness`, `contrast`, `grayscale`, `hue-rotate`, `invert`, `saturate`, `sepia`, `combined`
- `$VALUE` - Filter intensity value
  - Required
  - Example: `5px`, `150%`, `0.5`, `180deg`
- `$APPLY_TO` - When to apply the filter
  - Optional
  - Default: `always`
  - Options: `always`, `hover`, `focus`, `active`
- `$SELECTOR` - Target elements
  - Optional
  - Default: `.filter`
  - Example: `img`, `.hero`, `.card-image`

## Steps

### 1. Define filter configurations
```javascript
const filterConfigs = {
  blur: {
    unit: 'px',
    min: 0,
    max: 20,
    default: 4,
    cssFunction: 'blur'
  },
  brightness: {
    unit: '%',
    min: 0,
    max: 200,
    default: 100,
    cssFunction: 'brightness'
  },
  contrast: {
    unit: '%',
    min: 0,
    max: 200,
    default: 100,
    cssFunction: 'contrast'
  },
  grayscale: {
    unit: '%',
    min: 0,
    max: 100,
    default: 100,
    cssFunction: 'grayscale'
  },
  'hue-rotate': {
    unit: 'deg',
    min: 0,
    max: 360,
    default: 180,
    cssFunction: 'hue-rotate'
  },
  invert: {
    unit: '%',
    min: 0,
    max: 100,
    default: 100,
    cssFunction: 'invert'
  },
  saturate: {
    unit: '%',
    min: 0,
    max: 200,
    default: 150,
    cssFunction: 'saturate'
  },
  sepia: {
    unit: '%',
    min: 0,
    max: 100,
    default: 100,
    cssFunction: 'sepia'
  },
  'drop-shadow': {
    unit: '',
    default: '0 4px 6px rgba(0, 0, 0, 0.1)',
    cssFunction: 'drop-shadow'
  }
};
```
Defines filter configurations and defaults.

### 2. Apply filters to elements
```javascript
function applyFilter(selector, filterType, value, applyTo = 'always') {
  const elements = document.querySelectorAll(selector);
  
  if (filterType === 'none') {
    elements.forEach(element => {
      element.style.filter = 'none';
      element.classList.remove('filter-applied');
    });
    return;
  }
  
  let filterValue;
  if (filterType === 'combined') {
    // Allow custom combined filters
    filterValue = value;
  } else {
    const config = filterConfigs[filterType];
    if (!config) {
      console.error(`Filter type '${filterType}' not found`);
      return;
    }
    
    // Normalize value with unit
    const normalizedValue = value.toString().replace(/[^0-9.-]/g, '');
    filterValue = `${config.cssFunction}(${normalizedValue}${config.unit})`;
  }
  
  elements.forEach(element => {
    if (applyTo === 'always') {
      element.style.filter = filterValue;
    } else {
      // Remove inline filter for state-based application
      element.style.filter = '';
      
      // Create unique class for this filter
      const className = `filter-${filterType}-${applyTo}`;
      element.classList.add(className);
      
      // Add CSS rule for state
      addFilterStateRule(className, filterValue, applyTo);
    }
    
    // Add utility classes
    element.classList.add('filter-applied', `filter-${filterType}`);
    
    // Store filter data
    element.dataset.filterType = filterType;
    element.dataset.filterValue = value;
    element.dataset.filterApplyTo = applyTo;
  });
  
  // Save preferences
  saveFilterPreferences(selector, filterType, value, applyTo);
}
```
Applies filter effects to elements.

### 3. Add filter state rules
```javascript
function addFilterStateRule(className, filterValue, state) {
  const styleId = `filter-style-${className}`;
  
  // Remove existing style if present
  const existing = document.getElementById(styleId);
  if (existing) existing.remove();
  
  const pseudoClass = state === 'hover' ? ':hover' :
                     state === 'focus' ? ':focus' :
                     state === 'active' ? ':active' : '';
  
  const css = `.${className}${pseudoClass} { filter: ${filterValue}; }`;
  
  const style = document.createElement('style');
  style.id = styleId;
  style.textContent = css;
  document.head.appendChild(style);
}
```
Creates CSS rules for state-based filters.

### 4. Create filter utilities
```css
@layer utilities {
  /* Blur utilities */
  .blur-none { filter: blur(0); }
  .blur-sm { filter: blur(4px); }
  .blur { filter: blur(8px); }
  .blur-md { filter: blur(12px); }
  .blur-lg { filter: blur(16px); }
  .blur-xl { filter: blur(24px); }
  .blur-2xl { filter: blur(40px); }
  .blur-3xl { filter: blur(64px); }
  
  /* Brightness utilities */
  .brightness-0 { filter: brightness(0); }
  .brightness-50 { filter: brightness(0.5); }
  .brightness-75 { filter: brightness(0.75); }
  .brightness-90 { filter: brightness(0.9); }
  .brightness-95 { filter: brightness(0.95); }
  .brightness-100 { filter: brightness(1); }
  .brightness-105 { filter: brightness(1.05); }
  .brightness-110 { filter: brightness(1.1); }
  .brightness-125 { filter: brightness(1.25); }
  .brightness-150 { filter: brightness(1.5); }
  .brightness-200 { filter: brightness(2); }
  
  /* Contrast utilities */
  .contrast-0 { filter: contrast(0); }
  .contrast-50 { filter: contrast(0.5); }
  .contrast-75 { filter: contrast(0.75); }
  .contrast-100 { filter: contrast(1); }
  .contrast-125 { filter: contrast(1.25); }
  .contrast-150 { filter: contrast(1.5); }
  .contrast-200 { filter: contrast(2); }
  
  /* Grayscale utilities */
  .grayscale-0 { filter: grayscale(0); }
  .grayscale { filter: grayscale(1); }
  
  /* Hue rotate utilities */
  .hue-rotate-0 { filter: hue-rotate(0deg); }
  .hue-rotate-15 { filter: hue-rotate(15deg); }
  .hue-rotate-30 { filter: hue-rotate(30deg); }
  .hue-rotate-60 { filter: hue-rotate(60deg); }
  .hue-rotate-90 { filter: hue-rotate(90deg); }
  .hue-rotate-180 { filter: hue-rotate(180deg); }
  .-hue-rotate-180 { filter: hue-rotate(-180deg); }
  .-hue-rotate-90 { filter: hue-rotate(-90deg); }
  .-hue-rotate-60 { filter: hue-rotate(-60deg); }
  .-hue-rotate-30 { filter: hue-rotate(-30deg); }
  .-hue-rotate-15 { filter: hue-rotate(-15deg); }
  
  /* Invert utilities */
  .invert-0 { filter: invert(0); }
  .invert { filter: invert(1); }
  
  /* Saturate utilities */
  .saturate-0 { filter: saturate(0); }
  .saturate-50 { filter: saturate(0.5); }
  .saturate-100 { filter: saturate(1); }
  .saturate-150 { filter: saturate(1.5); }
  .saturate-200 { filter: saturate(2); }
  
  /* Sepia utilities */
  .sepia-0 { filter: sepia(0); }
  .sepia { filter: sepia(1); }
  
  /* Drop shadow utilities */
  .drop-shadow-sm { filter: drop-shadow(0 1px 1px rgb(0 0 0 / 0.05)); }
  .drop-shadow { filter: drop-shadow(0 1px 2px rgb(0 0 0 / 0.1)) drop-shadow(0 1px 1px rgb(0 0 0 / 0.06)); }
  .drop-shadow-md { filter: drop-shadow(0 4px 3px rgb(0 0 0 / 0.07)) drop-shadow(0 2px 2px rgb(0 0 0 / 0.06)); }
  .drop-shadow-lg { filter: drop-shadow(0 10px 8px rgb(0 0 0 / 0.04)) drop-shadow(0 4px 3px rgb(0 0 0 / 0.1)); }
  .drop-shadow-xl { filter: drop-shadow(0 20px 13px rgb(0 0 0 / 0.03)) drop-shadow(0 8px 5px rgb(0 0 0 / 0.08)); }
  .drop-shadow-2xl { filter: drop-shadow(0 25px 25px rgb(0 0 0 / 0.15)); }
  .drop-shadow-none { filter: drop-shadow(0 0 #0000); }
}
```
Creates filter utility classes.

### 5. Add filter combinations
```javascript
function addFilterCombinations() {
  const combinations = `
    /* Common filter combinations */
    .filter-vintage {
      filter: sepia(0.5) contrast(1.2) brightness(0.9);
    }
    
    .filter-dramatic {
      filter: contrast(1.5) brightness(0.9) saturate(1.2);
    }
    
    .filter-muted {
      filter: saturate(0.5) contrast(0.8) brightness(1.1);
    }
    
    .filter-cool {
      filter: hue-rotate(-10deg) saturate(0.8) brightness(1.05);
    }
    
    .filter-warm {
      filter: hue-rotate(10deg) saturate(1.2) brightness(1.05);
    }
    
    .filter-noir {
      filter: grayscale(1) contrast(1.2) brightness(0.9);
    }
    
    /* Hover filter effects */
    .hover-brighten:hover {
      filter: brightness(1.1);
    }
    
    .hover-darken:hover {
      filter: brightness(0.9);
    }
    
    .hover-desaturate:hover {
      filter: saturate(0.5);
    }
    
    .hover-blur:hover {
      filter: blur(4px);
    }
    
    /* Image overlay filters */
    .filter-overlay {
      position: relative;
    }
    .filter-overlay::before {
      content: '';
      position: absolute;
      inset: 0;
      background: rgba(0, 0, 0, 0.5);
      mix-blend-mode: multiply;
    }
    
    /* Backdrop filters */
    .backdrop-blur-sm { backdrop-filter: blur(4px); }
    .backdrop-blur { backdrop-filter: blur(8px); }
    .backdrop-blur-md { backdrop-filter: blur(12px); }
    .backdrop-blur-lg { backdrop-filter: blur(16px); }
    .backdrop-blur-xl { backdrop-filter: blur(24px); }
    
    .backdrop-brightness-50 { backdrop-filter: brightness(0.5); }
    .backdrop-brightness-75 { backdrop-filter: brightness(0.75); }
    .backdrop-brightness-100 { backdrop-filter: brightness(1); }
    .backdrop-brightness-125 { backdrop-filter: brightness(1.25); }
    .backdrop-brightness-150 { backdrop-filter: brightness(1.5); }
  `;
  
  const style = document.createElement('style');
  style.textContent = combinations;
  document.head.appendChild(style);
}
```
Adds filter combination effects.

### 6. Create interactive filter controls
```javascript
function createFilterControls() {
  const controlsHTML = `
    <div class="filter-controls" style="position: fixed; bottom: 20px; right: 20px; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); display: none;">
      <h3 style="margin: 0 0 10px;">Filter Controls</h3>
      <div class="filter-control">
        <label>Blur: <span id="blur-value">0</span>px</label>
        <input type="range" id="blur-range" min="0" max="20" value="0">
      </div>
      <div class="filter-control">
        <label>Brightness: <span id="brightness-value">100</span>%</label>
        <input type="range" id="brightness-range" min="0" max="200" value="100">
      </div>
      <div class="filter-control">
        <label>Contrast: <span id="contrast-value">100</span>%</label>
        <input type="range" id="contrast-range" min="0" max="200" value="100">
      </div>
      <div class="filter-control">
        <label>Saturate: <span id="saturate-value">100</span>%</label>
        <input type="range" id="saturate-range" min="0" max="200" value="100">
      </div>
      <button id="reset-filters">Reset</button>
    </div>
  `;
  
  // Add controls to page
  const controlsDiv = document.createElement('div');
  controlsDiv.innerHTML = controlsHTML;
  document.body.appendChild(controlsDiv);
  
  // Toggle controls with keyboard shortcut
  document.addEventListener('keydown', (e) => {
    if (e.ctrlKey && e.shiftKey && e.key === 'F') {
      const controls = document.querySelector('.filter-controls');
      controls.style.display = controls.style.display === 'none' ? 'block' : 'none';
    }
  });
  
  // Wire up controls
  const filters = ['blur', 'brightness', 'contrast', 'saturate'];
  filters.forEach(filter => {
    const range = document.getElementById(`${filter}-range`);
    const value = document.getElementById(`${filter}-value`);
    
    range?.addEventListener('input', (e) => {
      value.textContent = e.target.value;
      updateLiveFilters();
    });
  });
  
  // Reset button
  document.getElementById('reset-filters')?.addEventListener('click', () => {
    filters.forEach(filter => {
      const range = document.getElementById(`${filter}-range`);
      const config = filterConfigs[filter];
      if (range && config) {
        range.value = config.default;
        document.getElementById(`${filter}-value`).textContent = config.default;
      }
    });
    updateLiveFilters();
  });
}

function updateLiveFilters() {
  const target = document.querySelector('.filter-preview');
  if (!target) return;
  
  const blur = document.getElementById('blur-range')?.value || 0;
  const brightness = document.getElementById('brightness-range')?.value || 100;
  const contrast = document.getElementById('contrast-range')?.value || 100;
  const saturate = document.getElementById('saturate-range')?.value || 100;
  
  target.style.filter = `blur(${blur}px) brightness(${brightness}%) contrast(${contrast}%) saturate(${saturate}%)`;
}
```
Creates interactive filter controls.

### 7. Add performance optimizations
```javascript
function optimizeFilterPerformance() {
  // Use will-change for filtered elements
  const filteredElements = document.querySelectorAll('[class*="filter-"]');
  
  filteredElements.forEach(element => {
    // Add will-change on hover for interactive filters
    if (element.classList.contains('hover-')) {
      element.addEventListener('mouseenter', () => {
        element.style.willChange = 'filter';
      });
      
      element.addEventListener('mouseleave', () => {
        setTimeout(() => {
          element.style.willChange = 'auto';
        }, 300);
      });
    }
  });
  
  // Reduce filter complexity on low-end devices
  if (navigator.hardwareConcurrency <= 2) {
    document.documentElement.classList.add('reduce-filters');
    
    const style = document.createElement('style');
    style.textContent = `
      .reduce-filters [class*="filter-"] {
        filter: none !important;
      }
      .reduce-filters .filter-grayscale {
        filter: grayscale(1) !important;
      }
    `;
    document.head.appendChild(style);
  }
}
```
Optimizes filter performance.

### 8. Initialize filters
```javascript
// Apply main filter
applyFilter('$SELECTOR', '$FILTER_TYPE', '$VALUE', '$APPLY_TO');

// Add filter combinations
addFilterCombinations();

// Create interactive controls (development only)
if (window.location.hostname === 'localhost') {
  createFilterControls();
}

// Optimize performance
optimizeFilterPerformance();

// Save preferences
function saveFilterPreferences(selector, filterType, value, applyTo) {
  const preferences = JSON.parse(localStorage.getItem('filter-preferences') || '{}');
  preferences[selector] = { filterType, value, applyTo };
  localStorage.setItem('filter-preferences', JSON.stringify(preferences));
}

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('filter-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyFilter(selector, config.filterType, config.value, config.applyTo);
  });
});

// Add transition for smooth filter changes
const style = document.createElement('style');
style.textContent = `
  .filter-applied {
    transition: filter 0.3s ease;
  }
`;
document.head.appendChild(style);
```
Completes filter initialization.

## Validation
- Filters apply correctly to elements
- State-based filters work on interaction
- Filter combinations render properly
- Performance is acceptable
- Preferences persist on reload

## Error Handling
- **"Filter not applying"** - Check browser support
- **"Performance slow"** - Reduce filter complexity
- **"Values out of range"** - Check min/max limits
- **"Backdrop filter not working"** - Limited browser support

## Safety Notes
- Test performance impact on mobile
- Avoid complex filters on many elements
- Check browser compatibility
- Consider accessibility (contrast/readability)
- Provide option to disable filters

## Examples
- **Apply blur to images**
  ```
  tailwind-effects-filters blur 5px always img
  ```
  Applies 5px blur to all images

- **Grayscale on hover**
  ```
  tailwind-effects-filters grayscale 100% hover ".gallery img"
  ```
  Makes gallery images grayscale on hover

- **Brighten cards**
  ```
  tailwind-effects-filters brightness 110% always .card
  ```
  Makes all cards 10% brighter

- **Combined vintage effect**
  ```
  tailwind-effects-filters combined "sepia(0.5) contrast(1.2) brightness(0.9)" always .vintage
  ```
  Applies vintage filter combination