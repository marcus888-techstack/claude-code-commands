# Tailwind Layout Grid

## Purpose
Apply CSS Grid layouts to create responsive grid systems for your application

## Context
Use to create complex layouts with CSS Grid, including responsive grids, auto-fit layouts, and custom grid templates. Perfect for dashboards, galleries, card layouts, and any multi-column designs. More powerful than flexbox for 2D layouts.

## Parameters
- `$GRID_TYPE` - Type of grid layout to apply
  - Required
  - Options: `columns`, `auto-fit`, `auto-fill`, `areas`, `masonry`, `custom`
- `$COLUMNS` - Number of columns or column definition
  - Required
  - Example: `3`, `4`, `1fr 2fr 1fr`, `repeat(auto-fit, minmax(250px, 1fr))`
- `$GAP` - Gap between grid items
  - Optional
  - Default: `1rem`
  - Example: `0`, `0.5rem`, `2rem`, `1rem 2rem`
- `$SELECTOR` - Target container selector
  - Optional
  - Default: `.grid`
  - Example: `.gallery`, `.dashboard`, `.card-grid`

## Steps

### 1. Define grid layout presets
```javascript
const gridPresets = {
  columns: {
    '1': { gridTemplateColumns: 'repeat(1, minmax(0, 1fr))' },
    '2': { gridTemplateColumns: 'repeat(2, minmax(0, 1fr))' },
    '3': { gridTemplateColumns: 'repeat(3, minmax(0, 1fr))' },
    '4': { gridTemplateColumns: 'repeat(4, minmax(0, 1fr))' },
    '5': { gridTemplateColumns: 'repeat(5, minmax(0, 1fr))' },
    '6': { gridTemplateColumns: 'repeat(6, minmax(0, 1fr))' },
    '12': { gridTemplateColumns: 'repeat(12, minmax(0, 1fr))' }
  },
  
  'auto-fit': {
    sm: { gridTemplateColumns: 'repeat(auto-fit, minmax(150px, 1fr))' },
    md: { gridTemplateColumns: 'repeat(auto-fit, minmax(250px, 1fr))' },
    lg: { gridTemplateColumns: 'repeat(auto-fit, minmax(350px, 1fr))' }
  },
  
  'auto-fill': {
    sm: { gridTemplateColumns: 'repeat(auto-fill, minmax(150px, 1fr))' },
    md: { gridTemplateColumns: 'repeat(auto-fill, minmax(250px, 1fr))' },
    lg: { gridTemplateColumns: 'repeat(auto-fill, minmax(350px, 1fr))' }
  },
  
  areas: {
    'sidebar-content': {
      gridTemplateAreas: `"sidebar content"`,
      gridTemplateColumns: '250px 1fr'
    },
    'header-sidebar-content': {
      gridTemplateAreas: `
        "header header"
        "sidebar content"
      `,
      gridTemplateColumns: '250px 1fr',
      gridTemplateRows: 'auto 1fr'
    },
    'holy-grail': {
      gridTemplateAreas: `
        "header header header"
        "sidebar content aside"
        "footer footer footer"
      `,
      gridTemplateColumns: '200px 1fr 200px',
      gridTemplateRows: 'auto 1fr auto'
    }
  }
};
```
Defines grid layout presets.

### 2. Apply grid layout
```javascript
function applyGridLayout(selector, gridType, columns, gap) {
  const containers = document.querySelectorAll(selector);
  
  containers.forEach(container => {
    // Enable grid
    container.style.display = 'grid';
    
    // Apply gap
    if (gap.includes(' ')) {
      const [rowGap, colGap] = gap.split(' ');
      container.style.rowGap = rowGap;
      container.style.columnGap = colGap;
    } else {
      container.style.gap = gap;
    }
    
    // Apply grid type
    switch (gridType) {
      case 'columns':
        applyColumnGrid(container, columns);
        break;
      case 'auto-fit':
      case 'auto-fill':
        applyAutoGrid(container, gridType, columns);
        break;
      case 'areas':
        applyAreaGrid(container, columns);
        break;
      case 'masonry':
        applyMasonryGrid(container, columns);
        break;
      case 'custom':
        container.style.gridTemplateColumns = columns;
        break;
    }
    
    // Add utility classes
    container.classList.add('grid', `grid-${gridType}`);
    container.dataset.gridColumns = columns;
    container.dataset.gridGap = gap;
  });
  
  // Save preferences
  saveGridPreferences(selector, gridType, columns, gap);
}
```
Applies grid layout to containers.

### 3. Apply specific grid types
```javascript
function applyColumnGrid(container, columns) {
  const preset = gridPresets.columns[columns];
  if (preset) {
    container.style.gridTemplateColumns = preset.gridTemplateColumns;
  } else {
    container.style.gridTemplateColumns = `repeat(${columns}, minmax(0, 1fr))`;
  }
}

function applyAutoGrid(container, type, size) {
  const preset = gridPresets[type][size] || gridPresets[type].md;
  container.style.gridTemplateColumns = preset.gridTemplateColumns;
}

function applyAreaGrid(container, template) {
  const preset = gridPresets.areas[template];
  if (preset) {
    Object.entries(preset).forEach(([prop, value]) => {
      container.style[prop] = value;
    });
    
    // Add area classes to children
    const areas = preset.gridTemplateAreas.match(/\w+/g);
    const uniqueAreas = [...new Set(areas)];
    
    uniqueAreas.forEach((area, index) => {
      const child = container.children[index];
      if (child) {
        child.style.gridArea = area;
        child.classList.add(`grid-area-${area}`);
      }
    });
  }
}

function applyMasonryGrid(container, columns) {
  // Fallback for browsers without masonry support
  container.style.gridTemplateColumns = `repeat(${columns}, minmax(0, 1fr))`;
  container.style.gridAutoRows = '10px';
  
  // Apply masonry-like effect
  Array.from(container.children).forEach((item, index) => {
    const rowSpan = Math.ceil((item.offsetHeight + 10) / 10);
    item.style.gridRowEnd = `span ${rowSpan}`;
  });
}
```
Implements specific grid type layouts.

### 4. Create grid utilities
```css
@layer utilities {
  /* Grid display */
  .grid { display: grid; }
  .inline-grid { display: inline-grid; }
  
  /* Grid columns */
  .grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
  .grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
  .grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
  .grid-cols-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
  .grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
  .grid-cols-7 { grid-template-columns: repeat(7, minmax(0, 1fr)); }
  .grid-cols-8 { grid-template-columns: repeat(8, minmax(0, 1fr)); }
  .grid-cols-9 { grid-template-columns: repeat(9, minmax(0, 1fr)); }
  .grid-cols-10 { grid-template-columns: repeat(10, minmax(0, 1fr)); }
  .grid-cols-11 { grid-template-columns: repeat(11, minmax(0, 1fr)); }
  .grid-cols-12 { grid-template-columns: repeat(12, minmax(0, 1fr)); }
  .grid-cols-none { grid-template-columns: none; }
  
  /* Grid rows */
  .grid-rows-1 { grid-template-rows: repeat(1, minmax(0, 1fr)); }
  .grid-rows-2 { grid-template-rows: repeat(2, minmax(0, 1fr)); }
  .grid-rows-3 { grid-template-rows: repeat(3, minmax(0, 1fr)); }
  .grid-rows-4 { grid-template-rows: repeat(4, minmax(0, 1fr)); }
  .grid-rows-5 { grid-template-rows: repeat(5, minmax(0, 1fr)); }
  .grid-rows-6 { grid-template-rows: repeat(6, minmax(0, 1fr)); }
  .grid-rows-none { grid-template-rows: none; }
  
  /* Grid gap */
  .gap-0 { gap: 0px; }
  .gap-px { gap: 1px; }
  .gap-0\.5 { gap: 0.125rem; }
  .gap-1 { gap: 0.25rem; }
  .gap-1\.5 { gap: 0.375rem; }
  .gap-2 { gap: 0.5rem; }
  .gap-2\.5 { gap: 0.625rem; }
  .gap-3 { gap: 0.75rem; }
  .gap-3\.5 { gap: 0.875rem; }
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
  .gap-24 { gap: 6rem; }
  .gap-28 { gap: 7rem; }
  .gap-32 { gap: 8rem; }
  .gap-36 { gap: 9rem; }
  .gap-40 { gap: 10rem; }
  .gap-44 { gap: 11rem; }
  .gap-48 { gap: 12rem; }
  .gap-52 { gap: 13rem; }
  .gap-56 { gap: 14rem; }
  .gap-60 { gap: 15rem; }
  .gap-64 { gap: 16rem; }
  .gap-72 { gap: 18rem; }
  .gap-80 { gap: 20rem; }
  .gap-96 { gap: 24rem; }
  
  /* Column and row gap */
  .gap-x-0 { column-gap: 0px; }
  .gap-x-1 { column-gap: 0.25rem; }
  .gap-x-2 { column-gap: 0.5rem; }
  .gap-x-3 { column-gap: 0.75rem; }
  .gap-x-4 { column-gap: 1rem; }
  .gap-x-5 { column-gap: 1.25rem; }
  .gap-x-6 { column-gap: 1.5rem; }
  .gap-x-8 { column-gap: 2rem; }
  
  .gap-y-0 { row-gap: 0px; }
  .gap-y-1 { row-gap: 0.25rem; }
  .gap-y-2 { row-gap: 0.5rem; }
  .gap-y-3 { row-gap: 0.75rem; }
  .gap-y-4 { row-gap: 1rem; }
  .gap-y-5 { row-gap: 1.25rem; }
  .gap-y-6 { row-gap: 1.5rem; }
  .gap-y-8 { row-gap: 2rem; }
  
  /* Grid auto flow */
  .grid-flow-row { grid-auto-flow: row; }
  .grid-flow-col { grid-auto-flow: column; }
  .grid-flow-row-dense { grid-auto-flow: row dense; }
  .grid-flow-col-dense { grid-auto-flow: column dense; }
  
  /* Grid column span */
  .col-auto { grid-column: auto; }
  .col-span-1 { grid-column: span 1 / span 1; }
  .col-span-2 { grid-column: span 2 / span 2; }
  .col-span-3 { grid-column: span 3 / span 3; }
  .col-span-4 { grid-column: span 4 / span 4; }
  .col-span-5 { grid-column: span 5 / span 5; }
  .col-span-6 { grid-column: span 6 / span 6; }
  .col-span-7 { grid-column: span 7 / span 7; }
  .col-span-8 { grid-column: span 8 / span 8; }
  .col-span-9 { grid-column: span 9 / span 9; }
  .col-span-10 { grid-column: span 10 / span 10; }
  .col-span-11 { grid-column: span 11 / span 11; }
  .col-span-12 { grid-column: span 12 / span 12; }
  .col-span-full { grid-column: 1 / -1; }
  
  /* Grid row span */
  .row-auto { grid-row: auto; }
  .row-span-1 { grid-row: span 1 / span 1; }
  .row-span-2 { grid-row: span 2 / span 2; }
  .row-span-3 { grid-row: span 3 / span 3; }
  .row-span-4 { grid-row: span 4 / span 4; }
  .row-span-5 { grid-row: span 5 / span 5; }
  .row-span-6 { grid-row: span 6 / span 6; }
  .row-span-full { grid-row: 1 / -1; }
}
```
Creates comprehensive grid utilities.

### 5. Add responsive grid layouts
```javascript
function addResponsiveGrids() {
  const responsiveStyles = `
    /* Responsive grid columns */
    @media (min-width: 640px) {
      .sm\\:grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
      .sm\\:grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .sm\\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      .sm\\:grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
      .sm\\:grid-cols-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
      .sm\\:grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
    }
    
    @media (min-width: 768px) {
      .md\\:grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
      .md\\:grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .md\\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      .md\\:grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
      .md\\:grid-cols-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
      .md\\:grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
    }
    
    @media (min-width: 1024px) {
      .lg\\:grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
      .lg\\:grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .lg\\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      .lg\\:grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
      .lg\\:grid-cols-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
      .lg\\:grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
      .lg\\:grid-cols-7 { grid-template-columns: repeat(7, minmax(0, 1fr)); }
      .lg\\:grid-cols-8 { grid-template-columns: repeat(8, minmax(0, 1fr)); }
    }
    
    @media (min-width: 1280px) {
      .xl\\:grid-cols-1 { grid-template-columns: repeat(1, minmax(0, 1fr)); }
      .xl\\:grid-cols-2 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .xl\\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
      .xl\\:grid-cols-4 { grid-template-columns: repeat(4, minmax(0, 1fr)); }
      .xl\\:grid-cols-5 { grid-template-columns: repeat(5, minmax(0, 1fr)); }
      .xl\\:grid-cols-6 { grid-template-columns: repeat(6, minmax(0, 1fr)); }
      .xl\\:grid-cols-7 { grid-template-columns: repeat(7, minmax(0, 1fr)); }
      .xl\\:grid-cols-8 { grid-template-columns: repeat(8, minmax(0, 1fr)); }
    }
    
    /* Auto-responsive grid */
    .grid-auto-responsive {
      grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = responsiveStyles;
  document.head.appendChild(style);
}
```
Adds responsive grid layouts.

### 6. Create grid patterns
```javascript
function createGridPatterns() {
  const patterns = `
    /* Card grid pattern */
    .grid-cards {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 1.5rem;
    }
    
    /* Gallery grid pattern */
    .grid-gallery {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 0.5rem;
    }
    
    /* Dashboard grid pattern */
    .grid-dashboard {
      display: grid;
      grid-template-columns: repeat(12, 1fr);
      gap: 1rem;
    }
    
    /* Bento grid pattern */
    .grid-bento {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      grid-auto-rows: minmax(100px, auto);
      gap: 1rem;
    }
    .grid-bento > :nth-child(1) { grid-column: span 2; grid-row: span 2; }
    .grid-bento > :nth-child(2) { grid-column: span 2; }
    .grid-bento > :nth-child(3) { grid-row: span 2; }
    
    /* Magazine layout */
    .grid-magazine {
      display: grid;
      grid-template-columns: repeat(6, 1fr);
      gap: 2rem;
    }
    .grid-magazine > :first-child {
      grid-column: span 4;
      grid-row: span 2;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = patterns;
  document.head.appendChild(style);
}
```
Creates common grid patterns.

### 7. Add grid debugging
```javascript
function addGridDebugging() {
  const debugStyles = `
    /* Grid debugging */
    .grid-debug {
      position: relative;
    }
    
    .grid-debug > * {
      outline: 1px dashed rgba(255, 0, 0, 0.5);
      position: relative;
    }
    
    .grid-debug > *::before {
      content: attr(data-grid-item);
      position: absolute;
      top: 2px;
      left: 2px;
      background: rgba(255, 0, 0, 0.8);
      color: white;
      font-size: 10px;
      padding: 2px 4px;
      pointer-events: none;
      z-index: 1;
    }
    
    /* Show grid lines */
    .grid-lines {
      background-image: 
        repeating-linear-gradient(0deg, 
          transparent, 
          transparent calc(var(--grid-size, 100px) - 1px), 
          rgba(0, 0, 255, 0.1) calc(var(--grid-size, 100px) - 1px), 
          rgba(0, 0, 255, 0.1) var(--grid-size, 100px)
        ),
        repeating-linear-gradient(90deg, 
          transparent, 
          transparent calc(var(--grid-size, 100px) - 1px), 
          rgba(0, 0, 255, 0.1) calc(var(--grid-size, 100px) - 1px), 
          rgba(0, 0, 255, 0.1) var(--grid-size, 100px)
        );
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = debugStyles;
  document.head.appendChild(style);
  
  // Add item numbers to grid children
  document.querySelectorAll('.grid-debug').forEach(grid => {
    Array.from(grid.children).forEach((child, index) => {
      child.dataset.gridItem = index + 1;
    });
  });
}
```
Adds grid debugging features.

### 8. Initialize grid system
```javascript
// Apply grid layout
applyGridLayout('$SELECTOR', '$GRID_TYPE', '$COLUMNS', '$GAP');

// Add responsive grids
addResponsiveGrids();

// Create patterns
createGridPatterns();

// Add debugging (development only)
if (window.location.hostname === 'localhost') {
  addGridDebugging();
}

// Save preferences
function saveGridPreferences(selector, gridType, columns, gap) {
  const preferences = JSON.parse(localStorage.getItem('grid-preferences') || '{}');
  preferences[selector] = { gridType, columns, gap };
  localStorage.setItem('grid-preferences', JSON.stringify(preferences));
}

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('grid-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyGridLayout(selector, config.gridType, config.columns, config.gap);
  });
  
  // Handle masonry grids on load
  document.querySelectorAll('.grid-masonry').forEach(container => {
    applyMasonryGrid(container, container.dataset.gridColumns || '3');
  });
});

// Recalculate masonry on window resize
let resizeTimer;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => {
    document.querySelectorAll('.grid-masonry').forEach(container => {
      applyMasonryGrid(container, container.dataset.gridColumns || '3');
    });
  }, 250);
});
```
Completes grid system initialization.

## Validation
- Grid displays correctly
- Columns and rows align properly
- Gap spacing is consistent
- Responsive breakpoints work
- Grid areas map correctly

## Error Handling
- **"Grid not displaying"** - Check display: grid is set
- **"Columns not equal"** - Verify minmax values
- **"Items overlapping"** - Check grid-area assignments
- **"Responsive not working"** - Verify breakpoint classes

## Safety Notes
- Test on various screen sizes
- Check for content overflow
- Verify accessibility of grid layouts
- Test with dynamic content
- Consider fallbacks for older browsers

## Examples
- **Create 3-column grid**
  ```
  tailwind-layout-grid columns 3 1rem .gallery
  ```
  Creates 3 equal columns with 1rem gap

- **Auto-fit responsive grid**
  ```
  tailwind-layout-grid auto-fit md 2rem .cards
  ```
  Auto-fitting grid with medium size items

- **Holy grail layout**
  ```
  tailwind-layout-grid areas holy-grail 1rem .layout
  ```
  Classic header-sidebar-content-footer layout

- **Custom grid template**
  ```
  tailwind-layout-grid custom "200px 1fr 200px" 2rem .dashboard
  ```
  Custom grid with fixed sidebars