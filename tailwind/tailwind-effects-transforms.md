# Tailwind Effects Transforms

## Purpose
Apply CSS transform effects like scale, rotate, translate, and skew to elements using Tailwind utilities

## Context
Use to manipulate element positioning and appearance without affecting document flow. Perfect for hover effects, animations, and creating dynamic layouts. Supports 2D and 3D transforms with smooth transitions.

## Parameters
- `$TRANSFORM_TYPE` - Type of transform to apply
  - Required
  - Options: `scale`, `rotate`, `translate`, `skew`, `3d`, `combined`, `reset`
- `$VALUE` - Transform value
  - Required
  - Example: `1.1`, `45deg`, `10px`, `-15deg`
- `$AXIS` - Axis for the transform
  - Optional
  - Default: `both`
  - Options: `x`, `y`, `z`, `both`
- `$SELECTOR` - Target elements
  - Optional
  - Default: `.transform`
  - Example: `.card`, `img`, `button`

## Steps

### 1. Define transform configurations
```javascript
const transformConfigs = {
  scale: {
    x: (value) => `scaleX(${value})`,
    y: (value) => `scaleY(${value})`,
    both: (value) => `scale(${value})`,
    z: (value) => `scale3d(1, 1, ${value})`
  },
  
  rotate: {
    x: (value) => `rotateX(${value})`,
    y: (value) => `rotateY(${value})`,
    both: (value) => `rotate(${value})`,
    z: (value) => `rotateZ(${value})`
  },
  
  translate: {
    x: (value) => `translateX(${value})`,
    y: (value) => `translateY(${value})`,
    both: (value) => `translate(${value}, ${value})`,
    z: (value) => `translateZ(${value})`
  },
  
  skew: {
    x: (value) => `skewX(${value})`,
    y: (value) => `skewY(${value})`,
    both: (value) => `skew(${value}, ${value})`
  }
};
```
Defines transform function configurations.

### 2. Apply transforms to elements
```javascript
function applyTransform(selector, type, value, axis = 'both') {
  const elements = document.querySelectorAll(selector);
  
  if (type === 'reset') {
    elements.forEach(element => {
      element.style.transform = 'none';
      element.classList.remove('transform-applied');
    });
    return;
  }
  
  const transformConfig = transformConfigs[type];
  if (!transformConfig) {
    console.error(`Transform type '${type}' not found`);
    return;
  }
  
  const transformFunction = transformConfig[axis] || transformConfig.both;
  const transformValue = transformFunction(value);
  
  elements.forEach(element => {
    // Get existing transforms
    const currentTransform = element.style.transform || '';
    
    // Apply new transform
    if (type === '3d') {
      enable3DTransform(element);
      element.style.transform = transformValue;
    } else if (type === 'combined') {
      element.style.transform = value; // Allow custom combined transforms
    } else {
      element.style.transform = transformValue;
    }
    
    // Add utility classes
    element.classList.add('transform-applied', `transform-${type}-${axis}`);
    
    // Store transform data
    element.dataset.transformType = type;
    element.dataset.transformValue = value;
    element.dataset.transformAxis = axis;
  });
  
  // Save preferences
  saveTransformPreferences(selector, type, value, axis);
}
```
Applies transform effects to elements.

### 3. Enable 3D transforms
```javascript
function enable3DTransform(element) {
  // Set perspective on parent
  const parent = element.parentElement;
  if (parent) {
    parent.style.perspective = '1000px';
    parent.style.perspectiveOrigin = 'center';
  }
  
  // Enable 3D for element
  element.style.transformStyle = 'preserve-3d';
  element.style.backfaceVisibility = 'hidden';
}
```
Enables 3D transform capabilities.

### 4. Create transform utilities
```css
@layer utilities {
  /* Transform origin */
  .origin-center { transform-origin: center; }
  .origin-top { transform-origin: top; }
  .origin-top-right { transform-origin: top right; }
  .origin-right { transform-origin: right; }
  .origin-bottom-right { transform-origin: bottom right; }
  .origin-bottom { transform-origin: bottom; }
  .origin-bottom-left { transform-origin: bottom left; }
  .origin-left { transform-origin: left; }
  .origin-top-left { transform-origin: top left; }
  
  /* Scale utilities */
  .scale-0 { transform: scale(0); }
  .scale-50 { transform: scale(0.5); }
  .scale-75 { transform: scale(0.75); }
  .scale-90 { transform: scale(0.9); }
  .scale-95 { transform: scale(0.95); }
  .scale-100 { transform: scale(1); }
  .scale-105 { transform: scale(1.05); }
  .scale-110 { transform: scale(1.1); }
  .scale-125 { transform: scale(1.25); }
  .scale-150 { transform: scale(1.5); }
  
  /* Rotate utilities */
  .rotate-0 { transform: rotate(0deg); }
  .rotate-1 { transform: rotate(1deg); }
  .rotate-2 { transform: rotate(2deg); }
  .rotate-3 { transform: rotate(3deg); }
  .rotate-6 { transform: rotate(6deg); }
  .rotate-12 { transform: rotate(12deg); }
  .rotate-45 { transform: rotate(45deg); }
  .rotate-90 { transform: rotate(90deg); }
  .rotate-180 { transform: rotate(180deg); }
  .-rotate-180 { transform: rotate(-180deg); }
  .-rotate-90 { transform: rotate(-90deg); }
  .-rotate-45 { transform: rotate(-45deg); }
  .-rotate-12 { transform: rotate(-12deg); }
  .-rotate-6 { transform: rotate(-6deg); }
  .-rotate-3 { transform: rotate(-3deg); }
  .-rotate-2 { transform: rotate(-2deg); }
  .-rotate-1 { transform: rotate(-1deg); }
  
  /* Translate utilities */
  .translate-x-0 { transform: translateX(0); }
  .translate-x-1 { transform: translateX(0.25rem); }
  .translate-x-2 { transform: translateX(0.5rem); }
  .translate-x-4 { transform: translateX(1rem); }
  .translate-x-8 { transform: translateX(2rem); }
  .-translate-x-1 { transform: translateX(-0.25rem); }
  .-translate-x-2 { transform: translateX(-0.5rem); }
  .-translate-x-4 { transform: translateX(-1rem); }
  .-translate-x-8 { transform: translateX(-2rem); }
  
  .translate-y-0 { transform: translateY(0); }
  .translate-y-1 { transform: translateY(0.25rem); }
  .translate-y-2 { transform: translateY(0.5rem); }
  .translate-y-4 { transform: translateY(1rem); }
  .translate-y-8 { transform: translateY(2rem); }
  .-translate-y-1 { transform: translateY(-0.25rem); }
  .-translate-y-2 { transform: translateY(-0.5rem); }
  .-translate-y-4 { transform: translateY(-1rem); }
  .-translate-y-8 { transform: translateY(-2rem); }
  
  /* Skew utilities */
  .skew-x-0 { transform: skewX(0deg); }
  .skew-x-1 { transform: skewX(1deg); }
  .skew-x-2 { transform: skewX(2deg); }
  .skew-x-3 { transform: skewX(3deg); }
  .skew-x-6 { transform: skewX(6deg); }
  .skew-x-12 { transform: skewX(12deg); }
  .-skew-x-12 { transform: skewX(-12deg); }
  .-skew-x-6 { transform: skewX(-6deg); }
  .-skew-x-3 { transform: skewX(-3deg); }
  .-skew-x-2 { transform: skewX(-2deg); }
  .-skew-x-1 { transform: skewX(-1deg); }
  
  .skew-y-0 { transform: skewY(0deg); }
  .skew-y-1 { transform: skewY(1deg); }
  .skew-y-2 { transform: skewY(2deg); }
  .skew-y-3 { transform: skewY(3deg); }
  .skew-y-6 { transform: skewY(6deg); }
  .skew-y-12 { transform: skewY(12deg); }
  .-skew-y-12 { transform: skewY(-12deg); }
  .-skew-y-6 { transform: skewY(-6deg); }
  .-skew-y-3 { transform: skewY(-3deg); }
  .-skew-y-2 { transform: skewY(-2deg); }
  .-skew-y-1 { transform: skewY(-1deg); }
}
```
Creates transform utility classes.

### 5. Add 3D transform effects
```javascript
function add3DTransforms() {
  const styles3D = `
    /* 3D card flip */
    .transform-3d-flip {
      transform-style: preserve-3d;
      transition: transform 0.6s;
    }
    .transform-3d-flip:hover {
      transform: rotateY(180deg);
    }
    .transform-3d-flip-face {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden;
    }
    .transform-3d-flip-back {
      transform: rotateY(180deg);
    }
    
    /* 3D tilt */
    .transform-3d-tilt {
      transform-style: preserve-3d;
      transform: perspective(1000px) rotateX(0) rotateY(0);
      transition: transform 0.3s;
    }
    
    /* 3D cube */
    .transform-3d-cube {
      transform-style: preserve-3d;
      transform: rotateX(-20deg) rotateY(30deg);
    }
    .transform-3d-cube-face {
      position: absolute;
      width: 200px;
      height: 200px;
    }
    .transform-3d-cube-front { transform: translateZ(100px); }
    .transform-3d-cube-back { transform: rotateY(180deg) translateZ(100px); }
    .transform-3d-cube-right { transform: rotateY(90deg) translateZ(100px); }
    .transform-3d-cube-left { transform: rotateY(-90deg) translateZ(100px); }
    .transform-3d-cube-top { transform: rotateX(90deg) translateZ(100px); }
    .transform-3d-cube-bottom { transform: rotateX(-90deg) translateZ(100px); }
  `;
  
  const style = document.createElement('style');
  style.textContent = styles3D;
  document.head.appendChild(style);
}
```
Adds 3D transform effects.

### 6. Create interactive transforms
```javascript
function createInteractiveTransforms() {
  // Mouse parallax effect
  document.addEventListener('mousemove', (e) => {
    const parallaxElements = document.querySelectorAll('.transform-parallax');
    
    parallaxElements.forEach(element => {
      const speed = element.dataset.parallaxSpeed || 0.5;
      const x = (window.innerWidth - e.pageX * speed) / 100;
      const y = (window.innerHeight - e.pageY * speed) / 100;
      
      element.style.transform = `translate(${x}px, ${y}px)`;
    });
  });
  
  // Tilt on hover
  const tiltElements = document.querySelectorAll('.transform-3d-tilt');
  
  tiltElements.forEach(element => {
    element.addEventListener('mousemove', (e) => {
      const rect = element.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      
      const centerX = rect.width / 2;
      const centerY = rect.height / 2;
      
      const rotateX = (y - centerY) / 10;
      const rotateY = (centerX - x) / 10;
      
      element.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg)`;
    });
    
    element.addEventListener('mouseleave', () => {
      element.style.transform = 'perspective(1000px) rotateX(0) rotateY(0)';
    });
  });
}
```
Creates interactive transform effects.

### 7. Add transform combinations
```javascript
function addTransformCombinations() {
  const combinations = `
    /* Hover combinations */
    .hover-lift-rotate:hover {
      transform: translateY(-4px) rotate(2deg);
    }
    
    .hover-grow-rotate:hover {
      transform: scale(1.05) rotate(-2deg);
    }
    
    .hover-slide-skew:hover {
      transform: translateX(4px) skewX(-2deg);
    }
    
    /* Active state transforms */
    .active-shrink:active {
      transform: scale(0.95);
    }
    
    .active-press:active {
      transform: translateY(2px);
    }
    
    /* Focus transforms */
    .focus-grow:focus {
      transform: scale(1.05);
      outline: none;
    }
    
    /* Combined utility classes */
    .transform-center {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    
    .transform-flip-x {
      transform: scaleX(-1);
    }
    
    .transform-flip-y {
      transform: scaleY(-1);
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = combinations;
  document.head.appendChild(style);
}
```
Adds transform combination effects.

### 8. Initialize transforms
```javascript
// Apply main transform
applyTransform('$SELECTOR', '$TRANSFORM_TYPE', '$VALUE', '$AXIS');

// Add 3D transforms
add3DTransforms();

// Create interactive transforms
createInteractiveTransforms();

// Add combinations
addTransformCombinations();

// Performance optimization
const transformElements = document.querySelectorAll('[class*="transform-"]');
transformElements.forEach(element => {
  // Use will-change for frequently transformed elements
  if (element.classList.contains('transform-interactive')) {
    element.style.willChange = 'transform';
  }
  
  // Add GPU acceleration
  element.style.transform += ' translateZ(0)';
});

// Save preferences
function saveTransformPreferences(selector, type, value, axis) {
  const preferences = JSON.parse(localStorage.getItem('transform-preferences') || '{}');
  preferences[selector] = { type, value, axis };
  localStorage.setItem('transform-preferences', JSON.stringify(preferences));
}

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('transform-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyTransform(selector, config.type, config.value, config.axis);
  });
});
```
Completes transform initialization.

## Validation
- Transforms apply correctly
- 3D transforms have proper perspective
- Interactive transforms respond smoothly
- Performance is optimized
- Transforms combine properly

## Error Handling
- **"Transform not applying"** - Check CSS conflicts
- **"3D not working"** - Ensure perspective is set
- **"Performance issues"** - Use translateZ(0) for GPU
- **"Jumpy animations"** - Add will-change property

## Safety Notes
- Test performance on mobile devices
- Use GPU acceleration for smooth transforms
- Avoid transforming too many elements
- Consider motion sickness with 3D effects
- Test with different viewport sizes

## Examples
- **Scale elements on hover**
  ```
  tailwind-effects-transforms scale 1.1 both ".card:hover"
  ```
  Scales cards to 110% on hover

- **Rotate images**
  ```
  tailwind-effects-transforms rotate 15deg both "img"
  ```
  Rotates all images by 15 degrees

- **3D card flip**
  ```
  tailwind-effects-transforms 3d "rotateY(180deg)" y ".flip-card"
  ```
  Creates 3D flip effect

- **Reset transforms**
  ```
  tailwind-effects-transforms reset "" both ".transform"
  ```
  Removes all transforms