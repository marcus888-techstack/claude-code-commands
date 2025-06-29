# Tailwind Theme Custom Vars

## Purpose
Define and manage custom CSS variables for advanced theming in Tailwind CSS applications

## Context
Use to create a flexible theming system with CSS custom properties that can be dynamically updated. Enables runtime theme switching, user customization, and complex design tokens beyond Tailwind's default configuration.

## Parameters
- `$VAR_NAME` - Name of the custom variable to define
  - Required
  - Example: `spacing-unit`, `border-radius-base`, `font-scale`
- `$VAR_VALUE` - Value for the custom variable
  - Required
  - Example: `8px`, `1.25`, `#3B82F6`
- `$SCOPE` - Scope where variable should be defined
  - Optional
  - Default: `:root`
  - Options: `:root`, `.theme-light`, `.theme-dark`, `[data-theme]`
- `$CATEGORY` - Variable category for organization
  - Optional
  - Example: `spacing`, `typography`, `effects`, `colors`

## Steps

### 1. Set up CSS variable structure
```css
/* Organize variables by category */
@layer base {
  :root {
    /* Spacing variables */
    --spacing-unit: 0.25rem;
    --spacing-xs: calc(var(--spacing-unit) * 2);
    --spacing-sm: calc(var(--spacing-unit) * 3);
    --spacing-md: calc(var(--spacing-unit) * 4);
    --spacing-lg: calc(var(--spacing-unit) * 6);
    --spacing-xl: calc(var(--spacing-unit) * 8);
    
    /* Typography variables */
    --font-scale: 1.25;
    --font-size-xs: calc(0.75rem * var(--font-scale));
    --font-size-sm: calc(0.875rem * var(--font-scale));
    --font-size-base: calc(1rem * var(--font-scale));
    --font-size-lg: calc(1.125rem * var(--font-scale));
    --font-size-xl: calc(1.25rem * var(--font-scale));
    
    /* Effects variables */
    --border-radius-base: 0.375rem;
    --shadow-color: 0 0 0;
    --shadow-opacity: 0.1;
    --transition-base: 150ms ease-in-out;
    
    /* Animation variables */
    --animation-duration: 300ms;
    --animation-timing: cubic-bezier(0.4, 0, 0.2, 1);
  }
}
```
Creates organized CSS variable structure.

### 2. Define variable setter function
```javascript
function setCustomVariable(name, value, scope = ':root', category = null) {
  // Get target element based on scope
  let targetElement;
  if (scope === ':root') {
    targetElement = document.documentElement;
  } else {
    targetElement = document.querySelector(scope);
    if (!targetElement) {
      console.error(`Scope '${scope}' not found`);
      return false;
    }
  }
  
  // Format variable name
  const varName = name.startsWith('--') ? name : `--${name}`;
  
  // Set the variable
  targetElement.style.setProperty(varName, value);
  
  // Store in organized structure
  const customVars = JSON.parse(localStorage.getItem('custom-theme-vars') || '{}');
  
  if (!customVars[scope]) {
    customVars[scope] = {};
  }
  
  if (category) {
    if (!customVars[scope][category]) {
      customVars[scope][category] = {};
    }
    customVars[scope][category][varName] = value;
  } else {
    customVars[scope][varName] = value;
  }
  
  localStorage.setItem('custom-theme-vars', JSON.stringify(customVars));
  
  return true;
}

// Apply the custom variable
setCustomVariable('$VAR_NAME', '$VAR_VALUE', '$SCOPE', '$CATEGORY');
```
Sets custom variables with proper scoping.

### 3. Extend Tailwind config to use custom vars
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      spacing: {
        'xs': 'var(--spacing-xs)',
        'sm': 'var(--spacing-sm)',
        'md': 'var(--spacing-md)',
        'lg': 'var(--spacing-lg)',
        'xl': 'var(--spacing-xl)',
      },
      fontSize: {
        'xs': 'var(--font-size-xs)',
        'sm': 'var(--font-size-sm)',
        'base': 'var(--font-size-base)',
        'lg': 'var(--font-size-lg)',
        'xl': 'var(--font-size-xl)',
      },
      borderRadius: {
        'base': 'var(--border-radius-base)',
        'sm': 'calc(var(--border-radius-base) * 0.5)',
        'lg': 'calc(var(--border-radius-base) * 1.5)',
        'xl': 'calc(var(--border-radius-base) * 2)',
      },
      transitionDuration: {
        'base': 'var(--animation-duration)',
      },
      transitionTimingFunction: {
        'base': 'var(--animation-timing)',
      }
    }
  }
}
```
Integrates custom variables with Tailwind utilities.

### 4. Create computed variables
```javascript
function createComputedVariables() {
  const root = document.documentElement;
  
  // Create computed spacing scale
  const spacingUnit = getComputedStyle(root).getPropertyValue('--spacing-unit');
  
  for (let i = 1; i <= 12; i++) {
    root.style.setProperty(`--spacing-${i}`, `calc(${spacingUnit} * ${i})`);
  }
  
  // Create computed color shades
  const primaryColor = getComputedStyle(root).getPropertyValue('--color-primary-500');
  if (primaryColor) {
    // Generate shades based on primary color
    const shades = generateColorScale(primaryColor);
    Object.entries(shades).forEach(([shade, value]) => {
      root.style.setProperty(`--color-primary-${shade}`, value);
    });
  }
}
```
Creates calculated variables based on base values.

### 5. Add variable validation
```javascript
function validateVariable(name, value, category) {
  const validations = {
    spacing: (val) => /^\d+(\.\d+)?(px|rem|em|%)$/.test(val),
    typography: (val) => /^\d+(\.\d+)?(px|rem|em|)$/.test(val) || !isNaN(val),
    colors: (val) => /^#[0-9A-Fa-f]{6}$|^rgb\(|^hsl\(/.test(val),
    effects: (val) => true, // Allow any value for effects
  };
  
  if (category && validations[category]) {
    return validations[category](value);
  }
  
  return true; // Default to valid
}
```
Validates variable values based on category.

### 6. Create variable groups
```javascript
function setVariableGroup(groupName, variables) {
  const groups = {
    'compact': {
      '--spacing-unit': '0.125rem',
      '--font-scale': '0.875',
      '--border-radius-base': '0.25rem',
    },
    'comfortable': {
      '--spacing-unit': '0.25rem',
      '--font-scale': '1',
      '--border-radius-base': '0.375rem',
    },
    'spacious': {
      '--spacing-unit': '0.375rem',
      '--font-scale': '1.125',
      '--border-radius-base': '0.5rem',
    }
  };
  
  const group = groups[groupName] || variables;
  
  Object.entries(group).forEach(([name, value]) => {
    setCustomVariable(name, value);
  });
}
```
Applies groups of related variables.

### 7. Load and apply saved variables
```javascript
function loadCustomVariables() {
  const customVars = JSON.parse(localStorage.getItem('custom-theme-vars') || '{}');
  
  Object.entries(customVars).forEach(([scope, vars]) => {
    const targetElement = scope === ':root' 
      ? document.documentElement 
      : document.querySelector(scope);
      
    if (targetElement) {
      // Handle nested structure (with categories)
      const applyVars = (obj) => {
        Object.entries(obj).forEach(([key, value]) => {
          if (typeof value === 'object') {
            applyVars(value);
          } else if (key.startsWith('--')) {
            targetElement.style.setProperty(key, value);
          }
        });
      };
      
      applyVars(vars);
    }
  });
  
  // Recompute calculated variables
  createComputedVariables();
}

// Load on page load
document.addEventListener('DOMContentLoaded', loadCustomVariables);
```
Restores custom variables from storage.

## Validation
- Variables are set on correct scope
- Values are valid for their category
- Computed variables update correctly
- Variables persist across reloads
- Tailwind utilities use custom values

## Error Handling
- **"Invalid variable name"** - Use valid CSS property names
- **"Scope not found"** - Ensure selector exists in DOM
- **"Invalid value format"** - Check value matches category requirements
- **"Variable not updating"** - Check for typos and specificity issues

## Safety Notes
- Validate values to prevent CSS injection
- Use meaningful variable names
- Document custom variables
- Test with different value ranges
- Provide reset functionality

## Examples
- **Set spacing unit**
  ```
  tailwind-theme-custom-vars spacing-unit 0.5rem :root spacing
  ```
  Changes base spacing unit affecting all spacing

- **Set dark theme variable**
  ```
  tailwind-theme-custom-vars shadow-opacity 0.3 .dark effects
  ```
  Sets shadow opacity for dark theme

- **Set typography scale**
  ```
  tailwind-theme-custom-vars font-scale 1.125 :root typography
  ```
  Increases font sizes by 12.5%

- **Set component-specific variable**
  ```
  tailwind-theme-custom-vars card-padding 1.5rem ".card" spacing
  ```
  Sets custom padding for card components