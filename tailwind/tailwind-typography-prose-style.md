# Tailwind Typography Prose Style

## Purpose
Apply and customize Tailwind Typography plugin prose styles for rich content formatting

## Context
Use to style long-form content like articles, blog posts, or documentation with beautiful typography. The prose classes provide opinionated styles for HTML elements within content areas, ensuring consistent and readable text formatting.

## Parameters
- `$PROSE_SIZE` - Size variant of prose styles
  - Required
  - Options: `sm`, `base`, `lg`, `xl`, `2xl`
- `$PROSE_THEME` - Color theme for prose
  - Optional
  - Default: `gray`
  - Options: `gray`, `slate`, `zinc`, `neutral`, `stone`
- `$PROSE_MODIFIERS` - Additional prose modifiers
  - Optional
  - Example: `invert`, `no-quotes`, `compact`
- `$SELECTOR` - Target selector for prose styles
  - Optional
  - Default: `.prose`
  - Example: `article`, `.content`, `#post`

## Steps

### 1. Configure Typography plugin
```javascript
// tailwind.config.js
module.exports = {
  plugins: [
    require('@tailwindcss/typography')({
      className: 'prose', // Base class name
    }),
  ],
  theme: {
    extend: {
      typography: (theme) => ({
        DEFAULT: {
          css: {
            // Global prose customizations
            maxWidth: '65ch',
            color: theme('colors.gray.700'),
            '[class~="lead"]': {
              color: theme('colors.gray.600'),
            },
          },
        },
        // Size variants
        sm: {
          css: {
            fontSize: '0.875rem',
            lineHeight: '1.75',
          },
        },
        lg: {
          css: {
            fontSize: '1.125rem',
            lineHeight: '1.75',
          },
        },
      }),
    },
  },
}
```
Sets up Typography plugin configuration.

### 2. Apply prose classes dynamically
```javascript
function applyProseStyle(selector, size, theme, modifiers) {
  const elements = document.querySelectorAll(selector);
  
  // Build prose classes
  const proseClasses = [
    'prose',
    `prose-${size}`,
    theme !== 'gray' ? `prose-${theme}` : '',
    ...modifiers.split(' ').filter(Boolean).map(mod => {
      if (mod === 'invert') return 'prose-invert';
      if (mod === 'no-quotes') return 'prose-quoteless';
      if (mod === 'compact') return 'prose-compact';
      return `prose-${mod}`;
    })
  ].filter(Boolean);
  
  elements.forEach(element => {
    // Remove existing prose classes
    element.className = element.className
      .split(' ')
      .filter(cls => !cls.startsWith('prose'))
      .join(' ');
    
    // Add new prose classes
    element.classList.add(...proseClasses);
  });
  
  // Save preferences
  localStorage.setItem('prose-preferences', JSON.stringify({
    selector: selector,
    size: size,
    theme: theme,
    modifiers: modifiers
  }));
}
```
Applies prose styles to selected elements.

### 3. Define custom prose themes
```css
/* Custom prose color themes */
@layer components {
  /* Blue theme */
  .prose-blue {
    --tw-prose-body: theme('colors.gray.700');
    --tw-prose-headings: theme('colors.blue.900');
    --tw-prose-links: theme('colors.blue.600');
    --tw-prose-links-hover: theme('colors.blue.700');
    --tw-prose-bold: theme('colors.blue.900');
    --tw-prose-counters: theme('colors.blue.600');
    --tw-prose-bullets: theme('colors.blue.400');
    --tw-prose-hr: theme('colors.blue.200');
    --tw-prose-quotes: theme('colors.blue.900');
    --tw-prose-quote-borders: theme('colors.blue.300');
    --tw-prose-captions: theme('colors.blue.700');
    --tw-prose-code: theme('colors.blue.900');
    --tw-prose-pre-code: theme('colors.blue.100');
    --tw-prose-pre-bg: theme('colors.blue.900');
  }
  
  /* Dark mode adjustments */
  .dark .prose-invert {
    --tw-prose-body: theme('colors.gray.300');
    --tw-prose-headings: theme('colors.white');
    --tw-prose-links: theme('colors.blue.400');
    --tw-prose-links-hover: theme('colors.blue.300');
  }
}
```
Creates custom prose color themes.

### 4. Add prose style variants
```javascript
function addProseVariants() {
  const style = document.createElement('style');
  style.textContent = `
    /* Compact prose variant */
    .prose-compact {
      font-size: 0.95em;
    }
    .prose-compact p {
      margin-top: 0.75em;
      margin-bottom: 0.75em;
    }
    .prose-compact h2 {
      margin-top: 1.5em;
      margin-bottom: 0.75em;
    }
    
    /* No quotes variant */
    .prose-quoteless blockquote {
      font-style: normal;
      font-weight: 400;
      quotes: none;
    }
    .prose-quoteless blockquote::before,
    .prose-quoteless blockquote::after {
      content: none;
    }
    
    /* Lead paragraph style */
    .prose .lead {
      font-size: 1.25em;
      line-height: 1.6;
      margin-top: 1.2em;
      margin-bottom: 1.2em;
      font-weight: 400;
    }
  `;
  document.head.appendChild(style);
}
```
Adds custom prose style variants.

### 5. Customize specific elements
```javascript
function customizeProseElements() {
  const customizations = {
    // Headings
    h1: {
      fontFamily: 'var(--font-headings, inherit)',
      fontWeight: '800',
      letterSpacing: '-0.02em'
    },
    h2: {
      fontFamily: 'var(--font-headings, inherit)',
      fontWeight: '700',
      letterSpacing: '-0.01em'
    },
    
    // Links
    a: {
      textDecoration: 'underline',
      textDecorationColor: 'var(--tw-prose-links)',
      textUnderlineOffset: '2px',
      transition: 'color 0.2s ease-in-out'
    },
    
    // Code blocks
    'pre code': {
      fontFamily: 'var(--font-mono, monospace)',
      fontSize: '0.875em'
    },
    
    // Blockquotes
    blockquote: {
      fontStyle: 'italic',
      borderLeftWidth: '4px',
      borderLeftColor: 'var(--tw-prose-quote-borders)',
      paddingLeft: '1em'
    }
  };
  
  // Apply customizations via CSS
  const css = Object.entries(customizations)
    .map(([selector, styles]) => {
      const props = Object.entries(styles)
        .map(([prop, value]) => `${prop.replace(/([A-Z])/g, '-$1').toLowerCase()}: ${value};`)
        .join('\n    ');
      return `'$SELECTOR' ${selector} {\n    ${props}\n  }`;
    })
    .join('\n\n  ');
  
  const style = document.createElement('style');
  style.textContent = css;
  document.head.appendChild(style);
}
```
Customizes individual prose elements.

### 6. Add responsive prose sizing
```javascript
function applyResponsiveProse() {
  const breakpoints = {
    sm: 640,
    md: 768,
    lg: 1024,
    xl: 1280
  };
  
  const sizes = {
    sm: 'prose-sm',
    md: 'prose-base',
    lg: 'prose-lg',
    xl: 'prose-xl'
  };
  
  function updateProseSize() {
    const width = window.innerWidth;
    let targetSize = 'prose-base';
    
    Object.entries(breakpoints).forEach(([breakpoint, minWidth]) => {
      if (width >= minWidth) {
        targetSize = sizes[breakpoint] || targetSize;
      }
    });
    
    const elements = document.querySelectorAll('$SELECTOR');
    elements.forEach(element => {
      // Remove size classes
      Object.values(sizes).forEach(size => element.classList.remove(size));
      // Add new size
      element.classList.add(targetSize);
    });
  }
  
  // Initial update
  updateProseSize();
  
  // Update on resize
  let resizeTimer;
  window.addEventListener('resize', () => {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(updateProseSize, 250);
  });
}
```
Implements responsive prose sizing.

### 7. Add prose utilities
```javascript
function addProseUtilities() {
  // First letter styling
  const firstLetterStyle = `
    .prose-dropcap > p:first-of-type::first-letter {
      float: left;
      font-size: 4em;
      line-height: 0.8;
      margin: 0.1em 0.1em 0 0;
      font-weight: 700;
      color: var(--tw-prose-headings);
    }
  `;
  
  // Columns for prose
  const columnsStyle = `
    @media (min-width: 768px) {
      .prose-columns-2 {
        column-count: 2;
        column-gap: 2rem;
      }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = firstLetterStyle + columnsStyle;
  document.head.appendChild(style);
}
```
Adds additional prose utility classes.

### 8. Apply the prose style
```javascript
// Apply prose styles
applyProseStyle('$SELECTOR', '$PROSE_SIZE', '$PROSE_THEME', '$PROSE_MODIFIERS');

// Add custom variants
addProseVariants();

// Customize elements
customizeProseElements();

// Add utilities
addProseUtilities();

// Load preferences on page load
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('prose-preferences') || '{}');
  if (preferences.selector) {
    applyProseStyle(
      preferences.selector,
      preferences.size || 'base',
      preferences.theme || 'gray',
      preferences.modifiers || ''
    );
  }
});
```
Executes prose style application.

## Validation
- Prose classes applied to correct elements
- Typography styles render properly
- Color theme applied correctly
- Modifiers work as expected
- Responsive sizing functions

## Error Handling
- **"Plugin not installed"** - Install @tailwindcss/typography
- **"Styles not applying"** - Check element selector
- **"Colors incorrect"** - Verify theme name
- **"Size not changing"** - Check class conflicts

## Safety Notes
- Test with various content lengths
- Ensure readability on all devices
- Check color contrast ratios
- Validate with screen readers
- Consider print styles

## Examples
- **Apply large prose to articles**
  ```
  tailwind-typography-prose-style lg gray "" article
  ```
  Applies large prose styles to article elements

- **Apply inverted prose for dark mode**
  ```
  tailwind-typography-prose-style base slate "invert" .content
  ```
  Applies inverted prose for dark backgrounds

- **Apply compact prose**
  ```
  tailwind-typography-prose-style sm neutral "compact no-quotes" .documentation
  ```
  Applies small, compact prose without blockquote styling

- **Apply blue theme prose**
  ```
  tailwind-typography-prose-style xl blue "" #blog-post
  ```
  Applies extra-large prose with blue color theme