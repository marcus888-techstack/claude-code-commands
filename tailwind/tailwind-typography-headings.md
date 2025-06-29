# Tailwind Typography Headings

## Purpose
Style all heading elements (h1-h6) consistently across your Tailwind CSS application

## Context
Use to establish a clear typographic hierarchy with consistent heading styles. Applies uniform styling to all heading levels including size, weight, spacing, and optional decorative elements. Essential for maintaining design consistency and improving content readability.

## Parameters
- `$STYLE_PRESET` - Predefined heading style to apply
  - Required
  - Options: `default`, `bold`, `elegant`, `modern`, `minimal`, `display`
- `$SIZE_SCALE` - Size scaling between heading levels  
  - Optional
  - Default: `1.25`
  - Example: `1.2`, `1.33`, `1.5`
- `$APPLY_TO` - Which headings to style
  - Optional
  - Default: `all`
  - Options: `all`, `h1-h3`, `h1-h4`, `semantic`
- `$INCLUDE_MARGINS` - Whether to include margin styles
  - Optional
  - Default: `yes`
  - Options: `yes`, `no`

## Steps

### 1. Define heading style presets
```javascript
const headingPresets = {
  default: {
    fontFamily: 'inherit',
    fontWeight: {
      h1: '800', h2: '700', h3: '600',
      h4: '600', h5: '500', h6: '500'
    },
    letterSpacing: {
      h1: '-0.025em', h2: '-0.02em', h3: '-0.01em',
      h4: '0', h5: '0', h6: '0'
    },
    textTransform: 'none',
    lineHeight: '1.2'
  },
  
  bold: {
    fontFamily: 'inherit',
    fontWeight: {
      h1: '900', h2: '800', h3: '700',
      h4: '700', h5: '600', h6: '600'
    },
    letterSpacing: {
      h1: '-0.03em', h2: '-0.025em', h3: '-0.02em',
      h4: '-0.01em', h5: '0', h6: '0'
    },
    textTransform: 'none',
    lineHeight: '1.1'
  },
  
  elegant: {
    fontFamily: 'var(--font-serif, Georgia, serif)',
    fontWeight: {
      h1: '300', h2: '300', h3: '400',
      h4: '400', h5: '500', h6: '500'
    },
    letterSpacing: {
      h1: '0.02em', h2: '0.01em', h3: '0',
      h4: '0', h5: '0', h6: '0'
    },
    textTransform: 'none',
    lineHeight: '1.4'
  },
  
  modern: {
    fontFamily: 'inherit',
    fontWeight: {
      h1: '700', h2: '600', h3: '500',
      h4: '500', h5: '500', h6: '500'
    },
    letterSpacing: {
      h1: '0', h2: '0', h3: '0',
      h4: '0', h5: '0.025em', h6: '0.025em'
    },
    textTransform: {
      h1: 'none', h2: 'none', h3: 'none',
      h4: 'none', h5: 'uppercase', h6: 'uppercase'
    },
    lineHeight: '1.3'
  },
  
  minimal: {
    fontFamily: 'inherit',
    fontWeight: {
      h1: '400', h2: '400', h3: '400',
      h4: '500', h5: '500', h6: '500'
    },
    letterSpacing: '0',
    textTransform: 'none',
    lineHeight: '1.5'
  },
  
  display: {
    fontFamily: 'var(--font-display, inherit)',
    fontWeight: {
      h1: '900', h2: '800', h3: '700',
      h4: '600', h5: '600', h6: '600'
    },
    letterSpacing: {
      h1: '-0.04em', h2: '-0.03em', h3: '-0.02em',
      h4: '-0.01em', h5: '0', h6: '0'
    },
    textTransform: 'none',
    lineHeight: '0.9'
  }
};
```
Defines available heading style presets.

### 2. Calculate heading sizes
```javascript
function calculateHeadingSizes(baseSize, scale) {
  const basePx = 16; // 1rem = 16px
  const sizes = {};
  
  // Calculate sizes using scale
  sizes.h6 = baseSize;
  sizes.h5 = baseSize * scale;
  sizes.h4 = sizes.h5 * scale;
  sizes.h3 = sizes.h4 * scale;
  sizes.h2 = sizes.h3 * scale;
  sizes.h1 = sizes.h2 * scale;
  
  // Convert to rem
  Object.keys(sizes).forEach(heading => {
    sizes[heading] = `${sizes[heading]}rem`;
  });
  
  return sizes;
}

// Default size configurations
const sizeConfigs = {
  mobile: calculateHeadingSizes(0.875, parseFloat('$SIZE_SCALE') || 1.25),
  desktop: calculateHeadingSizes(1, parseFloat('$SIZE_SCALE') || 1.25)
};
```
Calculates heading sizes based on scale.

### 3. Apply heading styles
```javascript
function applyHeadingStyles(preset, sizeScale, applyTo, includeMargins) {
  const styles = headingPresets[preset] || headingPresets.default;
  const root = document.documentElement;
  
  // Determine which headings to style
  const headings = applyTo === 'all' ? ['h1', 'h2', 'h3', 'h4', 'h5', 'h6'] :
                   applyTo === 'h1-h3' ? ['h1', 'h2', 'h3'] :
                   applyTo === 'h1-h4' ? ['h1', 'h2', 'h3', 'h4'] :
                   ['h1', 'h2', 'h3', 'h4', 'h5', 'h6'];
  
  // Apply CSS variables for each heading
  headings.forEach(heading => {
    // Font family
    root.style.setProperty(`--heading-${heading}-family`, styles.fontFamily);
    
    // Font weight
    const weight = typeof styles.fontWeight === 'object' 
      ? styles.fontWeight[heading] 
      : styles.fontWeight;
    root.style.setProperty(`--heading-${heading}-weight`, weight);
    
    // Letter spacing
    const letterSpacing = typeof styles.letterSpacing === 'object'
      ? styles.letterSpacing[heading]
      : styles.letterSpacing;
    root.style.setProperty(`--heading-${heading}-letter-spacing`, letterSpacing);
    
    // Text transform
    const transform = typeof styles.textTransform === 'object'
      ? styles.textTransform[heading]
      : styles.textTransform;
    root.style.setProperty(`--heading-${heading}-transform`, transform);
    
    // Line height
    root.style.setProperty(`--heading-${heading}-line-height`, styles.lineHeight);
    
    // Sizes
    root.style.setProperty(`--heading-${heading}-size`, sizeConfigs.desktop[heading]);
    root.style.setProperty(`--heading-${heading}-size-mobile`, sizeConfigs.mobile[heading]);
  });
  
  // Save preferences
  localStorage.setItem('heading-preferences', JSON.stringify({
    preset: preset,
    scale: sizeScale,
    applyTo: applyTo,
    includeMargins: includeMargins
  }));
}
```
Applies heading styles via CSS variables.

### 4. Create heading CSS rules
```javascript
function createHeadingCSS(includeMargins) {
  const marginStyles = includeMargins === 'yes' ? `
    /* Heading margins */
    h1 { margin-top: 0; margin-bottom: 0.5em; }
    h2 { margin-top: 1.5em; margin-bottom: 0.5em; }
    h3 { margin-top: 1.25em; margin-bottom: 0.5em; }
    h4 { margin-top: 1em; margin-bottom: 0.5em; }
    h5 { margin-top: 1em; margin-bottom: 0.5em; }
    h6 { margin-top: 1em; margin-bottom: 0.5em; }
    
    /* First child margins */
    h1:first-child, h2:first-child, h3:first-child,
    h4:first-child, h5:first-child, h6:first-child {
      margin-top: 0;
    }
  ` : '';
  
  const css = `
    /* Heading styles */
    h1, .h1 {
      font-family: var(--heading-h1-family);
      font-weight: var(--heading-h1-weight);
      letter-spacing: var(--heading-h1-letter-spacing);
      text-transform: var(--heading-h1-transform);
      line-height: var(--heading-h1-line-height);
      font-size: var(--heading-h1-size-mobile);
    }
    
    h2, .h2 {
      font-family: var(--heading-h2-family);
      font-weight: var(--heading-h2-weight);
      letter-spacing: var(--heading-h2-letter-spacing);
      text-transform: var(--heading-h2-transform);
      line-height: var(--heading-h2-line-height);
      font-size: var(--heading-h2-size-mobile);
    }
    
    h3, .h3 {
      font-family: var(--heading-h3-family);
      font-weight: var(--heading-h3-weight);
      letter-spacing: var(--heading-h3-letter-spacing);
      text-transform: var(--heading-h3-transform);
      line-height: var(--heading-h3-line-height);
      font-size: var(--heading-h3-size-mobile);
    }
    
    h4, .h4 {
      font-family: var(--heading-h4-family);
      font-weight: var(--heading-h4-weight);
      letter-spacing: var(--heading-h4-letter-spacing);
      text-transform: var(--heading-h4-transform);
      line-height: var(--heading-h4-line-height);
      font-size: var(--heading-h4-size-mobile);
    }
    
    h5, .h5 {
      font-family: var(--heading-h5-family);
      font-weight: var(--heading-h5-weight);
      letter-spacing: var(--heading-h5-letter-spacing);
      text-transform: var(--heading-h5-transform);
      line-height: var(--heading-h5-line-height);
      font-size: var(--heading-h5-size-mobile);
    }
    
    h6, .h6 {
      font-family: var(--heading-h6-family);
      font-weight: var(--heading-h6-weight);
      letter-spacing: var(--heading-h6-letter-spacing);
      text-transform: var(--heading-h6-transform);
      line-height: var(--heading-h6-line-height);
      font-size: var(--heading-h6-size-mobile);
    }
    
    /* Desktop sizes */
    @media (min-width: 768px) {
      h1, .h1 { font-size: var(--heading-h1-size); }
      h2, .h2 { font-size: var(--heading-h2-size); }
      h3, .h3 { font-size: var(--heading-h3-size); }
      h4, .h4 { font-size: var(--heading-h4-size); }
      h5, .h5 { font-size: var(--heading-h5-size); }
      h6, .h6 { font-size: var(--heading-h6-size); }
    }
    
    ${marginStyles}
  `;
  
  const style = document.createElement('style');
  style.textContent = css;
  style.id = 'heading-styles';
  
  // Remove existing styles
  const existing = document.getElementById('heading-styles');
  if (existing) existing.remove();
  
  document.head.appendChild(style);
}
```
Creates CSS rules for headings.

### 5. Add decorative elements
```javascript
function addHeadingDecorations(preset) {
  const decorations = {
    elegant: `
      h1::after {
        content: '';
        display: block;
        width: 3rem;
        height: 0.125rem;
        background: currentColor;
        margin-top: 0.5rem;
        opacity: 0.25;
      }
    `,
    modern: `
      h5, h6 {
        position: relative;
        padding-left: 1rem;
      }
      h5::before, h6::before {
        content: '';
        position: absolute;
        left: 0;
        top: 50%;
        transform: translateY(-50%);
        width: 0.25rem;
        height: 1em;
        background: var(--color-primary-500, currentColor);
      }
    `,
    display: `
      h1 {
        position: relative;
        display: inline-block;
      }
      h1::before {
        content: '';
        position: absolute;
        bottom: -0.1em;
        left: 0;
        right: 0;
        height: 0.1em;
        background: linear-gradient(90deg, 
          var(--color-primary-500, #3B82F6) 0%, 
          transparent 100%);
      }
    `
  };
  
  if (decorations[preset]) {
    const style = document.createElement('style');
    style.textContent = decorations[preset];
    style.id = 'heading-decorations';
    
    const existing = document.getElementById('heading-decorations');
    if (existing) existing.remove();
    
    document.head.appendChild(style);
  }
}
```
Adds decorative elements based on preset.

### 6. Add utility classes
```javascript
function addHeadingUtilities() {
  const utilities = `
    /* Balance text */
    .heading-balance {
      text-wrap: balance;
    }
    
    /* Gradient headings */
    .heading-gradient {
      background: linear-gradient(135deg, 
        var(--color-primary-600) 0%, 
        var(--color-secondary-600) 100%);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    
    /* Outlined headings */
    .heading-outline {
      -webkit-text-stroke: 2px currentColor;
      -webkit-text-fill-color: transparent;
    }
    
    /* Heading with subtitle */
    .heading-group {
      margin-bottom: 1rem;
    }
    .heading-subtitle {
      font-size: 0.5em;
      font-weight: normal;
      opacity: 0.7;
      display: block;
      margin-top: 0.25em;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = utilities;
  document.head.appendChild(style);
}
```
Adds utility classes for headings.

### 7. Apply all heading styles
```javascript
// Apply heading styles
applyHeadingStyles('$STYLE_PRESET', '$SIZE_SCALE', '$APPLY_TO', '$INCLUDE_MARGINS');

// Create CSS rules
createHeadingCSS('$INCLUDE_MARGINS');

// Add decorations
addHeadingDecorations('$STYLE_PRESET');

// Add utilities
addHeadingUtilities();

// Load preferences
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('heading-preferences') || '{}');
  if (preferences.preset) {
    applyHeadingStyles(
      preferences.preset,
      preferences.scale || 1.25,
      preferences.applyTo || 'all',
      preferences.includeMargins || 'yes'
    );
  }
});
```
Executes all heading style functions.

## Validation
- All specified headings have consistent styles
- Size hierarchy is maintained
- Responsive sizing works correctly
- Decorative elements render properly
- Margins create proper spacing

## Error Handling
- **"Styles not applying"** - Check CSS specificity
- **"Invalid preset"** - Use available preset names
- **"Sizes too large/small"** - Adjust size scale
- **"Decorations not showing"** - Check CSS support

## Safety Notes
- Test heading hierarchy for accessibility
- Ensure sufficient color contrast
- Verify readability at all sizes
- Check responsive behavior
- Consider SEO impact of heading structure

## Examples
- **Apply bold heading style**
  ```
  tailwind-typography-headings bold 1.25 all yes
  ```
  Applies bold preset to all headings with margins

- **Apply elegant style to main headings**
  ```
  tailwind-typography-headings elegant 1.33 h1-h3 yes
  ```
  Applies elegant serif style to h1-h3 only

- **Apply minimal style without margins**
  ```
  tailwind-typography-headings minimal 1.2 all no
  ```
  Applies minimal style without default margins

- **Apply display style with large scale**
  ```
  tailwind-typography-headings display 1.5 h1-h4 yes
  ```
  Applies dramatic display style with 1.5x scale