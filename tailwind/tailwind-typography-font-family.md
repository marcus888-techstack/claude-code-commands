# Tailwind Typography Font Family

## Purpose
Change font families throughout your Tailwind CSS application for different text categories

## Context
Use to apply custom fonts, web fonts, or system font stacks to your application. Supports different fonts for headings, body text, and UI elements. Essential for brand consistency and improving readability.

## Parameters
- `$FONT_CATEGORY` - Category of text to change fonts for
  - Required
  - Options: `all`, `sans`, `serif`, `mono`, `headings`, `body`, `ui`
- `$FONT_FAMILY` - Font family to apply
  - Required
  - Example: `Inter`, `Roboto`, `system-ui`, `Georgia`, `custom`
- `$FONT_STACK` - Complete font stack including fallbacks
  - Optional
  - Example: `"Inter", -apple-system, sans-serif`
- `$LOAD_METHOD` - How to load custom fonts
  - Optional
  - Default: `google`
  - Options: `google`, `local`, `none`

## Steps

### 1. Define font family presets
```javascript
const fontPresets = {
  // Sans-serif fonts
  inter: {
    name: 'Inter',
    stack: '"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif',
    weights: [300, 400, 500, 600, 700, 800],
    google: 'Inter:wght@300;400;500;600;700;800'
  },
  roboto: {
    name: 'Roboto',
    stack: '"Roboto", "Helvetica Neue", Arial, sans-serif',
    weights: [300, 400, 500, 700, 900],
    google: 'Roboto:wght@300;400;500;700;900'
  },
  system: {
    name: 'System UI',
    stack: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
    weights: 'all',
    google: null
  },
  
  // Serif fonts
  georgia: {
    name: 'Georgia',
    stack: 'Georgia, Cambria, "Times New Roman", Times, serif',
    weights: 'all',
    google: null
  },
  merriweather: {
    name: 'Merriweather',
    stack: '"Merriweather", Georgia, serif',
    weights: [300, 400, 700, 900],
    google: 'Merriweather:wght@300;400;700;900'
  },
  
  // Monospace fonts
  'fira-code': {
    name: 'Fira Code',
    stack: '"Fira Code", "Courier New", monospace',
    weights: [300, 400, 500, 600, 700],
    google: 'Fira+Code:wght@300;400;500;600;700'
  },
  'jetbrains-mono': {
    name: 'JetBrains Mono',
    stack: '"JetBrains Mono", Monaco, Consolas, monospace',
    weights: [400, 700],
    google: 'JetBrains+Mono:wght@400;700'
  }
};
```
Defines available font presets.

### 2. Load fonts from Google Fonts
```javascript
function loadGoogleFont(fontConfig) {
  if (fontConfig.google && '$LOAD_METHOD' === 'google') {
    // Check if already loaded
    const existingLink = document.querySelector(`link[href*="${fontConfig.name}"]`);
    if (existingLink) return;
    
    // Create link element
    const link = document.createElement('link');
    link.rel = 'preconnect';
    link.href = 'https://fonts.googleapis.com';
    document.head.appendChild(link);
    
    const link2 = document.createElement('link');
    link2.rel = 'preconnect';
    link2.href = 'https://fonts.gstatic.com';
    link2.crossOrigin = 'anonymous';
    document.head.appendChild(link2);
    
    const fontLink = document.createElement('link');
    fontLink.rel = 'stylesheet';
    fontLink.href = `https://fonts.googleapis.com/css2?family=${fontConfig.google}&display=swap`;
    document.head.appendChild(fontLink);
    
    console.log(`Loaded Google Font: ${fontConfig.name}`);
  }
}
```
Loads fonts from Google Fonts API.

### 3. Apply font family to CSS variables
```javascript
function applyFontFamily(category, fontName, customStack) {
  const root = document.documentElement;
  const fontConfig = fontPresets[fontName] || {
    name: fontName,
    stack: customStack || `"${fontName}", sans-serif`
  };
  
  // Load font if needed
  loadGoogleFont(fontConfig);
  
  // Apply based on category
  switch (category) {
    case 'all':
      root.style.setProperty('--font-sans', fontConfig.stack);
      root.style.setProperty('--font-serif', fontConfig.stack);
      root.style.setProperty('--font-mono', fontConfig.stack);
      break;
      
    case 'sans':
      root.style.setProperty('--font-sans', fontConfig.stack);
      break;
      
    case 'serif':
      root.style.setProperty('--font-serif', fontConfig.stack);
      break;
      
    case 'mono':
      root.style.setProperty('--font-mono', fontConfig.stack);
      break;
      
    case 'headings':
      root.style.setProperty('--font-headings', fontConfig.stack);
      break;
      
    case 'body':
      root.style.setProperty('--font-body', fontConfig.stack);
      break;
      
    case 'ui':
      root.style.setProperty('--font-ui', fontConfig.stack);
      break;
  }
  
  // Save preference
  const fontPreferences = JSON.parse(localStorage.getItem('font-preferences') || '{}');
  fontPreferences[category] = {
    name: fontName,
    stack: fontConfig.stack,
    timestamp: new Date().toISOString()
  };
  localStorage.setItem('font-preferences', JSON.stringify(fontPreferences));
}
```
Sets font families via CSS variables.

### 4. Update Tailwind configuration
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    fontFamily: {
      'sans': 'var(--font-sans)',
      'serif': 'var(--font-serif)',
      'mono': 'var(--font-mono)',
      'headings': 'var(--font-headings, var(--font-sans))',
      'body': 'var(--font-body, var(--font-sans))',
      'ui': 'var(--font-ui, var(--font-sans))'
    }
  }
}
```
Configures Tailwind to use CSS variables.

### 5. Apply font classes to elements
```css
/* Apply fonts to specific elements */
@layer base {
  body {
    font-family: var(--font-body, var(--font-sans));
  }
  
  h1, h2, h3, h4, h5, h6 {
    font-family: var(--font-headings, var(--font-sans));
  }
  
  button, input, select, textarea {
    font-family: var(--font-ui, var(--font-sans));
  }
  
  code, kbd, samp, pre {
    font-family: var(--font-mono);
  }
  
  /* Utility classes */
  .font-headings { font-family: var(--font-headings, var(--font-sans)); }
  .font-body { font-family: var(--font-body, var(--font-sans)); }
  .font-ui { font-family: var(--font-ui, var(--font-sans)); }
}
```
Applies fonts to HTML elements.

### 6. Handle font loading states
```javascript
function handleFontLoading() {
  if ('fonts' in document) {
    // Use Font Loading API
    document.fonts.ready.then(() => {
      document.documentElement.classList.add('fonts-loaded');
    });
    
    // Check specific fonts
    const fontsToCheck = ['Inter', 'Roboto', '$FONT_FAMILY'];
    
    fontsToCheck.forEach(fontName => {
      document.fonts.check(`16px "${fontName}"`).then(loaded => {
        if (loaded) {
          document.documentElement.classList.add(`font-${fontName.toLowerCase()}-loaded`);
        }
      });
    });
  }
}
```
Manages font loading states for smooth rendering.

### 7. Add font weight utilities
```javascript
function setupFontWeights(fontConfig) {
  if (!fontConfig.weights || fontConfig.weights === 'all') return;
  
  const root = document.documentElement;
  const weightMap = {
    thin: 100,
    extralight: 200,
    light: 300,
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700,
    extrabold: 800,
    black: 900
  };
  
  // Set available weights
  fontConfig.weights.forEach(weight => {
    const weightName = Object.keys(weightMap).find(key => weightMap[key] === weight);
    if (weightName) {
      root.style.setProperty(`--font-weight-${weightName}`, weight);
    }
  });
}
```
Configures available font weights.

### 8. Apply and validate
```javascript
// Apply the font family
applyFontFamily('$FONT_CATEGORY', '$FONT_FAMILY', '$FONT_STACK');

// Handle font loading
handleFontLoading();

// Load saved preferences on page load
document.addEventListener('DOMContentLoaded', () => {
  const fontPreferences = JSON.parse(localStorage.getItem('font-preferences') || '{}');
  
  Object.entries(fontPreferences).forEach(([category, config]) => {
    applyFontFamily(category, config.name, config.stack);
  });
});

// Add CSS for smooth font transitions
const style = document.createElement('style');
style.textContent = `
  .fonts-loaded body {
    opacity: 1;
    transition: opacity 0.3s ease-in-out;
  }
  body {
    opacity: 0.9; /* Slightly visible during font load */
  }
`;
document.head.appendChild(style);
```
Completes font family application.

## Validation
- Fonts load successfully
- Correct fonts applied to elements
- Fallback fonts work properly
- No FOUT (Flash of Unstyled Text)
- Preferences persist on reload

## Error Handling
- **"Font not loading"** - Check internet connection or CORS
- **"Invalid font name"** - Verify font exists in preset or custom
- **"Fallback showing"** - Primary font may not be available
- **"Mixed fonts"** - Check CSS specificity conflicts

## Safety Notes
- Always include fallback fonts
- Test on slow connections
- Consider font file sizes
- Ensure fonts support required characters
- Check font licensing for web use

## Examples
- **Apply Inter to all text**
  ```
  tailwind-typography-font-family all inter "" google
  ```
  Sets Inter as the font for entire application

- **Use Georgia for headings**
  ```
  tailwind-typography-font-family headings georgia
  ```
  Applies Georgia serif font to all headings

- **Set custom font stack**
  ```
  tailwind-typography-font-family body custom "Helvetica, Arial, sans-serif" none
  ```
  Uses custom font stack without loading web fonts

- **Apply JetBrains Mono to code**
  ```
  tailwind-typography-font-family mono jetbrains-mono "" google
  ```
  Sets JetBrains Mono for code elements