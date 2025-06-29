# Tailwind Effects Hover

## Purpose
Add interactive hover effects to elements using Tailwind CSS hover utilities and custom animations

## Context
Use to enhance user experience with visual feedback on interactive elements. Supports various hover effects including color changes, transforms, shadows, and custom animations. Essential for creating responsive and engaging interfaces.

## Parameters
- `$EFFECT_TYPE` - Type of hover effect to apply
  - Required
  - Options: `lift`, `grow`, `shrink`, `glow`, `fade`, `slide`, `rotate`, `custom`
- `$INTENSITY` - Intensity of the effect
  - Optional
  - Default: `medium`
  - Options: `subtle`, `medium`, `strong`
- `$DURATION` - Transition duration
  - Optional
  - Default: `300`
  - Example: `150`, `300`, `500` (milliseconds)
- `$SELECTOR` - CSS selector for target elements
  - Optional
  - Default: `.hover-effect`
  - Example: `button`, `.card`, `a`

## Steps

### 1. Define hover effect presets
```javascript
const hoverEffects = {
  lift: {
    subtle: {
      transform: 'translateY(-2px)',
      shadow: '0 4px 6px -1px rgb(0 0 0 / 0.1)'
    },
    medium: {
      transform: 'translateY(-4px)',
      shadow: '0 10px 15px -3px rgb(0 0 0 / 0.1)'
    },
    strong: {
      transform: 'translateY(-8px)',
      shadow: '0 20px 25px -5px rgb(0 0 0 / 0.1)'
    }
  },
  
  grow: {
    subtle: { transform: 'scale(1.02)' },
    medium: { transform: 'scale(1.05)' },
    strong: { transform: 'scale(1.1)' }
  },
  
  shrink: {
    subtle: { transform: 'scale(0.98)' },
    medium: { transform: 'scale(0.95)' },
    strong: { transform: 'scale(0.9)' }
  },
  
  glow: {
    subtle: { boxShadow: '0 0 10px rgba(var(--color-primary-500), 0.3)' },
    medium: { boxShadow: '0 0 20px rgba(var(--color-primary-500), 0.5)' },
    strong: { boxShadow: '0 0 30px rgba(var(--color-primary-500), 0.7)' }
  },
  
  fade: {
    subtle: { opacity: '0.9' },
    medium: { opacity: '0.7' },
    strong: { opacity: '0.5' }
  },
  
  slide: {
    subtle: { transform: 'translateX(2px)' },
    medium: { transform: 'translateX(4px)' },
    strong: { transform: 'translateX(8px)' }
  },
  
  rotate: {
    subtle: { transform: 'rotate(2deg)' },
    medium: { transform: 'rotate(5deg)' },
    strong: { transform: 'rotate(10deg)' }
  }
};
```
Defines various hover effect presets.

### 2. Apply hover effects
```javascript
function applyHoverEffect(selector, effectType, intensity, duration) {
  const elements = document.querySelectorAll(selector);
  const effect = hoverEffects[effectType];
  
  if (!effect) {
    console.error(`Hover effect '${effectType}' not found`);
    return;
  }
  
  const effectStyles = effect[intensity] || effect.medium;
  const transitionDuration = `${duration}ms`;
  
  elements.forEach(element => {
    // Set base transition
    element.style.transition = `all ${transitionDuration} cubic-bezier(0.4, 0, 0.2, 1)`;
    
    // Store original styles
    const originalStyles = {
      transform: element.style.transform || 'none',
      boxShadow: element.style.boxShadow || 'none',
      opacity: element.style.opacity || '1'
    };
    
    // Apply hover listeners
    element.addEventListener('mouseenter', () => {
      Object.entries(effectStyles).forEach(([prop, value]) => {
        element.style[prop] = value;
      });
    });
    
    element.addEventListener('mouseleave', () => {
      Object.entries(originalStyles).forEach(([prop, value]) => {
        element.style[prop] = value;
      });
    });
    
    // Add hover class for CSS fallback
    element.classList.add(`hover-${effectType}-${intensity}`);
  });
  
  // Save preferences
  saveHoverPreferences(selector, effectType, intensity, duration);
}
```
Applies hover effects to elements.

### 3. Create compound hover effects
```javascript
function createCompoundEffects() {
  const compoundEffects = `
    /* Lift and glow */
    .hover-lift-glow {
      transition: all 300ms ease;
    }
    .hover-lift-glow:hover {
      transform: translateY(-4px);
      box-shadow: 
        0 10px 15px -3px rgb(0 0 0 / 0.1),
        0 0 20px rgba(var(--color-primary-500), 0.3);
    }
    
    /* Grow and fade border */
    .hover-grow-border {
      border: 2px solid transparent;
      transition: all 300ms ease;
    }
    .hover-grow-border:hover {
      transform: scale(1.05);
      border-color: rgb(var(--color-primary-500));
    }
    
    /* 3D tilt effect */
    .hover-tilt {
      transition: transform 300ms ease;
      transform-style: preserve-3d;
    }
    .hover-tilt:hover {
      transform: 
        perspective(1000px) 
        rotateX(10deg) 
        rotateY(-10deg) 
        scale(1.05);
    }
    
    /* Underline slide */
    .hover-underline {
      position: relative;
      overflow: hidden;
    }
    .hover-underline::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 2px;
      background: rgb(var(--color-primary-500));
      transform: translateX(-100%);
      transition: transform 300ms ease;
    }
    .hover-underline:hover::after {
      transform: translateX(0);
    }
    
    /* Icon slide */
    .hover-icon-slide {
      overflow: hidden;
    }
    .hover-icon-slide .icon {
      transition: transform 300ms ease;
    }
    .hover-icon-slide:hover .icon {
      transform: translateX(4px);
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = compoundEffects;
  document.head.appendChild(style);
}
```
Creates compound hover effects.

### 4. Add gradient hover effects
```javascript
function addGradientHoverEffects() {
  const gradientEffects = `
    /* Gradient shift */
    .hover-gradient-shift {
      background-size: 200% 100%;
      background-position: 0% 0%;
      transition: background-position 300ms ease;
    }
    .hover-gradient-shift:hover {
      background-position: 100% 0%;
    }
    
    /* Gradient appear */
    .hover-gradient-appear {
      position: relative;
      z-index: 1;
    }
    .hover-gradient-appear::before {
      content: '';
      position: absolute;
      inset: 0;
      background: linear-gradient(
        135deg,
        rgb(var(--color-primary-500)),
        rgb(var(--color-secondary-500))
      );
      opacity: 0;
      transition: opacity 300ms ease;
      z-index: -1;
      border-radius: inherit;
    }
    .hover-gradient-appear:hover::before {
      opacity: 1;
    }
    
    /* Gradient border */
    .hover-gradient-border {
      position: relative;
      background: white;
      border: 2px solid transparent;
    }
    .hover-gradient-border::before {
      content: '';
      position: absolute;
      inset: -2px;
      background: linear-gradient(45deg, 
        rgb(var(--color-primary-500)),
        rgb(var(--color-secondary-500))
      );
      border-radius: inherit;
      opacity: 0;
      transition: opacity 300ms ease;
      z-index: -1;
    }
    .hover-gradient-border:hover::before {
      opacity: 1;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = gradientEffects;
  document.head.appendChild(style);
}
```
Adds gradient-based hover effects.

### 5. Create text hover effects
```javascript
function createTextHoverEffects() {
  const textEffects = `
    /* Text color change */
    .hover-text-primary {
      transition: color 300ms ease;
    }
    .hover-text-primary:hover {
      color: rgb(var(--color-primary-500));
    }
    
    /* Text gradient on hover */
    .hover-text-gradient {
      background-image: linear-gradient(
        45deg,
        currentColor 50%,
        rgb(var(--color-primary-500)) 50%
      );
      background-size: 200% 100%;
      background-position: 0% 0%;
      -webkit-background-clip: text;
      background-clip: text;
      transition: background-position 300ms ease;
    }
    .hover-text-gradient:hover {
      background-position: -100% 0%;
      -webkit-text-fill-color: transparent;
    }
    
    /* Letter spacing */
    .hover-text-spread {
      transition: letter-spacing 300ms ease;
    }
    .hover-text-spread:hover {
      letter-spacing: 0.1em;
    }
    
    /* Text shadow */
    .hover-text-shadow:hover {
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = textEffects;
  document.head.appendChild(style);
}
```
Creates text-specific hover effects.

### 6. Add button hover effects
```javascript
function addButtonHoverEffects() {
  const buttonEffects = `
    /* Button fill */
    .hover-fill {
      position: relative;
      overflow: hidden;
      z-index: 1;
    }
    .hover-fill::before {
      content: '';
      position: absolute;
      top: 0;
      left: -100%;
      width: 100%;
      height: 100%;
      background: rgb(var(--color-primary-600));
      transition: left 300ms ease;
      z-index: -1;
    }
    .hover-fill:hover::before {
      left: 0;
    }
    .hover-fill:hover {
      color: white;
    }
    
    /* Button sweep */
    .hover-sweep {
      position: relative;
      overflow: hidden;
    }
    .hover-sweep::before {
      content: '';
      position: absolute;
      top: 0;
      left: -100%;
      width: 100%;
      height: 100%;
      background: rgba(255, 255, 255, 0.2);
      transform: skewX(-20deg);
      transition: left 500ms ease;
    }
    .hover-sweep:hover::before {
      left: 125%;
    }
    
    /* Button pulse */
    @keyframes pulse {
      0% { box-shadow: 0 0 0 0 rgba(var(--color-primary-500), 0.4); }
      70% { box-shadow: 0 0 0 10px rgba(var(--color-primary-500), 0); }
      100% { box-shadow: 0 0 0 0 rgba(var(--color-primary-500), 0); }
    }
    .hover-pulse:hover {
      animation: pulse 1s infinite;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = buttonEffects;
  document.head.appendChild(style);
}
```
Adds button-specific hover effects.

### 7. Handle touch devices
```javascript
function handleTouchDevices() {
  // Detect touch capability
  const isTouchDevice = 'ontouchstart' in window || 
                       navigator.maxTouchPoints > 0;
  
  if (isTouchDevice) {
    // Add touch-friendly hover effects
    document.documentElement.classList.add('touch-device');
    
    // Convert hover to tap for touch devices
    const hoverElements = document.querySelectorAll('[class*="hover-"]');
    
    hoverElements.forEach(element => {
      element.addEventListener('touchstart', function() {
        this.classList.add('touch-active');
      });
      
      element.addEventListener('touchend', function() {
        setTimeout(() => {
          this.classList.remove('touch-active');
        }, 300);
      });
    });
  }
  
  // Add CSS for touch devices
  const touchStyles = `
    .touch-device [class*="hover-"]:active,
    .touch-device .touch-active {
      transform: scale(0.98);
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = touchStyles;
  document.head.appendChild(style);
}
```
Handles hover effects on touch devices.

### 8. Apply and initialize effects
```javascript
// Apply main hover effect
applyHoverEffect('$SELECTOR', '$EFFECT_TYPE', '$INTENSITY', parseInt('$DURATION'));

// Add additional effect styles
createCompoundEffects();
addGradientHoverEffects();
createTextHoverEffects();
addButtonHoverEffects();

// Handle touch devices
handleTouchDevices();

// Save preferences
function saveHoverPreferences(selector, effect, intensity, duration) {
  const preferences = JSON.parse(localStorage.getItem('hover-preferences') || '{}');
  preferences[selector] = { effect, intensity, duration };
  localStorage.setItem('hover-preferences', JSON.stringify(preferences));
}

// Load preferences on page load
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('hover-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyHoverEffect(selector, config.effect, config.intensity, config.duration);
  });
});
```
Completes hover effect initialization.

## Validation
- Hover effects trigger on mouse enter
- Transitions are smooth
- Effects revert on mouse leave
- Touch devices show alternative feedback
- Preferences persist on reload

## Error Handling
- **"Effect not working"** - Check element has pointer-events
- **"Transition too fast/slow"** - Adjust duration parameter
- **"Effect too subtle"** - Increase intensity level
- **"Performance issues"** - Reduce number of animated properties

## Safety Notes
- Test on various devices and browsers
- Ensure effects don't break layout
- Consider reduced motion preferences
- Avoid effects that cause layout shift
- Test with keyboard navigation

## Examples
- **Apply lift effect to cards**
  ```
  tailwind-effects-hover lift medium 300 .card
  ```
  Cards lift up with shadow on hover

- **Apply glow effect to buttons**
  ```
  tailwind-effects-hover glow strong 500 "button"
  ```
  Buttons get strong glow effect

- **Apply subtle grow to links**
  ```
  tailwind-effects-hover grow subtle 200 "a"
  ```
  Links slightly grow on hover

- **Apply rotate effect to icons**
  ```
  tailwind-effects-hover rotate medium 400 ".icon"
  ```
  Icons rotate on hover