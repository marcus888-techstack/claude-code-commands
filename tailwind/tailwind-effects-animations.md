# Tailwind Effects Animations

## Purpose
Apply CSS animations to elements using Tailwind's animation utilities and custom keyframe animations

## Context
Use to add eye-catching animations like pulse, bounce, spin, or custom keyframe animations. Perfect for loading states, attention-grabbing elements, or enhancing user interactions. Supports both Tailwind's built-in animations and custom creations.

## Parameters
- `$ANIMATION` - Animation to apply
  - Required
  - Options: `pulse`, `bounce`, `spin`, `ping`, `fade`, `slide`, `shake`, `custom`
- `$DURATION` - Animation duration
  - Optional
  - Default: `1000`
  - Example: `500`, `1000`, `2000` (milliseconds)
- `$ITERATION` - Number of iterations
  - Optional
  - Default: `infinite`
  - Options: `1`, `2`, `3`, `infinite`
- `$SELECTOR` - Target elements
  - Optional
  - Default: `.animate`
  - Example: `.loading`, `button`, `#logo`

## Steps

### 1. Define animation keyframes
```javascript
const animationKeyframes = {
  pulse: `
    @keyframes pulse {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.5; }
    }
  `,
  
  bounce: `
    @keyframes bounce {
      0%, 100% {
        transform: translateY(-25%);
        animation-timing-function: cubic-bezier(0.8, 0, 1, 1);
      }
      50% {
        transform: translateY(0);
        animation-timing-function: cubic-bezier(0, 0, 0.2, 1);
      }
    }
  `,
  
  spin: `
    @keyframes spin {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }
  `,
  
  ping: `
    @keyframes ping {
      75%, 100% {
        transform: scale(2);
        opacity: 0;
      }
    }
  `,
  
  fade: `
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
    @keyframes fadeOut {
      from { opacity: 1; }
      to { opacity: 0; }
    }
  `,
  
  slide: `
    @keyframes slideInLeft {
      from {
        transform: translateX(-100%);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
    }
    @keyframes slideInRight {
      from {
        transform: translateX(100%);
        opacity: 0;
      }
      to {
        transform: translateX(0);
        opacity: 1;
      }
    }
  `,
  
  shake: `
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      10%, 30%, 50%, 70%, 90% { transform: translateX(-10px); }
      20%, 40%, 60%, 80% { transform: translateX(10px); }
    }
  `,
  
  wiggle: `
    @keyframes wiggle {
      0%, 100% { transform: rotate(-3deg); }
      50% { transform: rotate(3deg); }
    }
  `,
  
  heartbeat: `
    @keyframes heartbeat {
      0% { transform: scale(1); }
      14% { transform: scale(1.3); }
      28% { transform: scale(1); }
      42% { transform: scale(1.3); }
      70% { transform: scale(1); }
    }
  `,
  
  float: `
    @keyframes float {
      0%, 100% { transform: translateY(0); }
      50% { transform: translateY(-20px); }
    }
  `
};
```
Defines various animation keyframes.

### 2. Apply animations to elements
```javascript
function applyAnimation(selector, animation, duration, iteration) {
  const elements = document.querySelectorAll(selector);
  
  // Inject keyframes if not already present
  if (!document.querySelector(`style[data-animation="${animation}"]`)) {
    const style = document.createElement('style');
    style.setAttribute('data-animation', animation);
    style.textContent = animationKeyframes[animation] || '';
    document.head.appendChild(style);
  }
  
  // Apply animation to elements
  elements.forEach(element => {
    // Remove existing animation classes
    element.className = element.className
      .split(' ')
      .filter(cls => !cls.startsWith('animate-'))
      .join(' ');
    
    // Build animation value
    const animationName = getAnimationName(animation);
    const iterationCount = iteration === 'infinite' ? 'infinite' : iteration;
    const animationValue = `${animationName} ${duration}ms ease-in-out ${iterationCount}`;
    
    // Apply animation
    element.style.animation = animationValue;
    
    // Add utility class
    element.classList.add(`animate-${animation}`);
    
    // Handle animation end
    if (iteration !== 'infinite') {
      element.addEventListener('animationend', function handler() {
        element.style.animation = '';
        element.removeEventListener('animationend', handler);
      });
    }
  });
  
  // Save preferences
  saveAnimationPreferences(selector, animation, duration, iteration);
}
```
Applies animations to selected elements.

### 3. Get animation name mapping
```javascript
function getAnimationName(animation) {
  const animationMap = {
    pulse: 'pulse',
    bounce: 'bounce',
    spin: 'spin',
    ping: 'ping',
    fade: 'fadeIn',
    'fade-out': 'fadeOut',
    slide: 'slideInLeft',
    'slide-right': 'slideInRight',
    shake: 'shake',
    wiggle: 'wiggle',
    heartbeat: 'heartbeat',
    float: 'float'
  };
  
  return animationMap[animation] || animation;
}
```
Maps animation types to keyframe names.

### 4. Create complex animations
```javascript
function createComplexAnimations() {
  const complexAnimations = `
    /* Morphing animation */
    @keyframes morph {
      0% { border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%; }
      50% { border-radius: 30% 60% 70% 40% / 50% 60% 30% 60%; }
      100% { border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%; }
    }
    
    /* Gradient animation */
    @keyframes gradient {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
    .animate-gradient {
      background-size: 200% 200%;
      animation: gradient 3s ease infinite;
    }
    
    /* Typewriter effect */
    @keyframes typewriter {
      from { width: 0; }
      to { width: 100%; }
    }
    @keyframes blink {
      50% { border-color: transparent; }
    }
    .animate-typewriter {
      overflow: hidden;
      border-right: 3px solid;
      white-space: nowrap;
      animation: 
        typewriter 3s steps(40, end),
        blink 0.75s step-end infinite;
    }
    
    /* 3D flip */
    @keyframes flip3d {
      0% {
        transform: perspective(400px) rotateY(0);
      }
      100% {
        transform: perspective(400px) rotateY(360deg);
      }
    }
    
    /* Rubber band */
    @keyframes rubberBand {
      0% { transform: scale3d(1, 1, 1); }
      30% { transform: scale3d(1.25, 0.75, 1); }
      40% { transform: scale3d(0.75, 1.25, 1); }
      50% { transform: scale3d(1.15, 0.85, 1); }
      65% { transform: scale3d(0.95, 1.05, 1); }
      75% { transform: scale3d(1.05, 0.95, 1); }
      100% { transform: scale3d(1, 1, 1); }
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = complexAnimations;
  document.head.appendChild(style);
}
```
Creates complex animation effects.

### 5. Add animation utilities
```javascript
function addAnimationUtilities() {
  const utilities = `
    /* Animation delays */
    .animation-delay-100 { animation-delay: 100ms; }
    .animation-delay-200 { animation-delay: 200ms; }
    .animation-delay-300 { animation-delay: 300ms; }
    .animation-delay-500 { animation-delay: 500ms; }
    .animation-delay-1000 { animation-delay: 1000ms; }
    
    /* Animation fill modes */
    .animation-fill-both { animation-fill-mode: both; }
    .animation-fill-forwards { animation-fill-mode: forwards; }
    .animation-fill-backwards { animation-fill-mode: backwards; }
    
    /* Animation play states */
    .animation-paused { animation-play-state: paused; }
    .animation-running { animation-play-state: running; }
    
    /* Hover to animate */
    .hover-animate:hover {
      animation-play-state: running;
    }
    .hover-animate {
      animation-play-state: paused;
    }
    
    /* Animation directions */
    .animation-reverse { animation-direction: reverse; }
    .animation-alternate { animation-direction: alternate; }
    .animation-alternate-reverse { animation-direction: alternate-reverse; }
  `;
  
  const style = document.createElement('style');
  style.textContent = utilities;
  document.head.appendChild(style);
}
```
Adds animation utility classes.

### 6. Create loading animations
```javascript
function createLoadingAnimations() {
  const loadingAnimations = `
    /* Dots loading */
    @keyframes dot1 {
      0%, 80%, 100% { transform: scale(0); opacity: 0; }
      40% { transform: scale(1); opacity: 1; }
    }
    .loading-dots span {
      display: inline-block;
      width: 10px;
      height: 10px;
      background: currentColor;
      border-radius: 50%;
      animation: dot1 1.4s infinite ease-in-out both;
    }
    .loading-dots span:nth-child(1) { animation-delay: -0.32s; }
    .loading-dots span:nth-child(2) { animation-delay: -0.16s; }
    
    /* Spinner */
    .loading-spinner {
      border: 3px solid rgb(var(--color-gray-200));
      border-top-color: rgb(var(--color-primary-500));
      border-radius: 50%;
      animation: spin 1s linear infinite;
    }
    
    /* Progress bar */
    @keyframes progress {
      0% { width: 0%; }
      100% { width: 100%; }
    }
    .loading-progress {
      position: relative;
      overflow: hidden;
    }
    .loading-progress::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      height: 100%;
      background: rgb(var(--color-primary-500));
      animation: progress 2s ease-in-out infinite;
    }
    
    /* Skeleton loading */
    @keyframes skeleton {
      0% { background-position: -200% 0; }
      100% { background-position: 200% 0; }
    }
    .loading-skeleton {
      background: linear-gradient(
        90deg,
        rgb(var(--color-gray-200)) 25%,
        rgb(var(--color-gray-300)) 50%,
        rgb(var(--color-gray-200)) 75%
      );
      background-size: 200% 100%;
      animation: skeleton 1.5s ease-in-out infinite;
    }
  `;
  
  const style = document.createElement('style');
  style.textContent = loadingAnimations;
  document.head.appendChild(style);
}
```
Creates loading-specific animations.

### 7. Handle animation performance
```javascript
function optimizeAnimationPerformance() {
  // Use will-change for performance
  const animatedElements = document.querySelectorAll('[class*="animate-"]');
  
  animatedElements.forEach(element => {
    // Add will-change on animation start
    element.addEventListener('animationstart', () => {
      element.style.willChange = 'transform, opacity';
    });
    
    // Remove will-change on animation end
    element.addEventListener('animationend', () => {
      element.style.willChange = 'auto';
    });
  });
  
  // Pause animations when not visible
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.style.animationPlayState = 'running';
      } else {
        entry.target.style.animationPlayState = 'paused';
      }
    });
  });
  
  animatedElements.forEach(element => {
    observer.observe(element);
  });
}
```
Optimizes animation performance.

### 8. Initialize animations
```javascript
// Apply main animation
applyAnimation('$SELECTOR', '$ANIMATION', parseInt('$DURATION'), '$ITERATION');

// Create additional animations
createComplexAnimations();
addAnimationUtilities();
createLoadingAnimations();

// Optimize performance
optimizeAnimationPerformance();

// Handle reduced motion
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
if (prefersReducedMotion.matches) {
  document.documentElement.classList.add('reduce-motion');
}

// Save preferences
function saveAnimationPreferences(selector, animation, duration, iteration) {
  const preferences = JSON.parse(localStorage.getItem('animation-preferences') || '{}');
  preferences[selector] = { animation, duration, iteration };
  localStorage.setItem('animation-preferences', JSON.stringify(preferences));
}

// Load saved preferences
document.addEventListener('DOMContentLoaded', () => {
  const preferences = JSON.parse(localStorage.getItem('animation-preferences') || '{}');
  Object.entries(preferences).forEach(([selector, config]) => {
    applyAnimation(selector, config.animation, config.duration, config.iteration);
  });
});
```
Completes animation initialization.

## Validation
- Animations play correctly
- Duration and iteration count work
- Performance is optimized
- Reduced motion is respected
- Animations pause when not visible

## Error Handling
- **"Animation not playing"** - Check keyframes are loaded
- **"Performance issues"** - Reduce number of animated elements
- **"Animation jumpy"** - Check for conflicting CSS
- **"Not smooth on mobile"** - Use transform instead of position

## Safety Notes
- Test performance on various devices
- Respect prefers-reduced-motion setting
- Avoid animating too many elements
- Use transform and opacity for best performance
- Consider battery usage on mobile

## Examples
- **Apply pulse to buttons**
  ```
  tailwind-effects-animations pulse 2000 infinite button
  ```
  Makes all buttons pulse continuously

- **Apply bounce to icons**
  ```
  tailwind-effects-animations bounce 1000 3 .icon
  ```
  Icons bounce 3 times

- **Apply spin to loading**
  ```
  tailwind-effects-animations spin 1000 infinite .loading
  ```
  Creates spinning loading indicator

- **Apply shake once**
  ```
  tailwind-effects-animations shake 500 1 .error
  ```
  Shakes error elements once