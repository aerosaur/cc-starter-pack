# Micro-Interactions and CSS Animations

## What Are Micro-Interactions?

Micro-interactions are small, contained moments of UI interaction that accomplish a single task and provide feedback to users. They enhance user experience by:

- **Providing feedback** - Confirming actions (button clicks, form submissions)
- **Guiding users** - Showing what's possible (hover states, interactive cues)
- **Creating delight** - Adding personality and polish to interactions
- **Communicating status** - Loading states, success/error feedback
- **Preventing errors** - Visual validation and guidance

### Types of Micro-Interactions

1. **Triggers** - What initiates the interaction (hover, click, focus, scroll)
2. **Rules** - What happens during the interaction
3. **Feedback** - How users perceive the interaction
4. **Loops & Modes** - What happens next (repeat, change state)

## Best Practices

### 1. Subtlety is Essential
Animations should enhance without distracting. Keep them:
- **Brief** - 200-300ms for most interactions
- **Purposeful** - Every animation should serve a function
- **Subtle** - Natural, not flashy

### 2. Performance First

**Use GPU-Accelerated Properties:**
```css
/* ✅ GOOD - GPU accelerated */
.element {
  transform: translateX(100px);
  opacity: 0.5;
  filter: blur(4px);
}

/* ❌ AVOID - Triggers layout/paint */
.element {
  left: 100px;
  width: 200px;
  background: red;
}
```

**Properties to Prefer:**
- `transform` (translate, scale, rotate)
- `opacity`
- `filter`

**Properties to Avoid:**
- `top`, `left`, `right`, `bottom` (use `transform: translate` instead)
- `width`, `height` (use `transform: scale` instead)
- `margin`, `padding`

### 3. Timing and Easing

**Duration Guidelines:**
```css
/* Micro-interactions: 200-300ms */
.button:hover {
  transition: transform 250ms ease-out;
}

/* Larger movements: 300-500ms */
.modal {
  animation: slideIn 400ms ease-out;
}

/* Background changes: 150-200ms */
.card:hover {
  transition: background-color 200ms ease;
}
```

**Easing Functions:**
```css
/* Entering (starting slow) */
ease-in
cubic-bezier(0.4, 0, 1, 1)

/* Exiting (ending slow) */
ease-out
cubic-bezier(0, 0, 0.2, 1)

/* Both (natural feeling) */
ease-in-out
cubic-bezier(0.4, 0, 0.2, 1)

/* Custom easing */
cubic-bezier(0.68, -0.55, 0.265, 1.55) /* Bounce effect */
```

### 4. Accessibility

**Respect Reduced Motion:**
```css
/* Default animation */
.element {
  transition: transform 300ms ease-out;
}

/* Disable for users who prefer reduced motion */
@media (prefers-reduced-motion: reduce) {
  .element {
    transition: none;
  }
}

/* Or provide instant alternative */
@media (prefers-reduced-motion: reduce) {
  .element {
    transition: opacity 0ms;
  }
}
```

## Common Micro-Interactions

### Button Hover Effects

**1. Lift Effect**
```css
.button {
  position: relative;
  transition: transform 200ms ease-out,
              box-shadow 200ms ease-out;
}

.button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
}
```

**2. Scale Effect**
```css
.button {
  transition: transform 200ms cubic-bezier(0.34, 1.56, 0.64, 1);
}

.button:hover {
  transform: scale(1.05);
}

.button:active {
  transform: scale(0.98);
}
```

**3. Shine/Sweep Effect**
```css
.button {
  position: relative;
  overflow: hidden;
}

.button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  transition: left 600ms ease;
}

.button:hover::before {
  left: 100%;
}
```

**4. Fill Effect**
```css
.button {
  position: relative;
  z-index: 1;
  background: transparent;
  border: 2px solid #0066cc;
  color: #0066cc;
  overflow: hidden;
  transition: color 300ms ease;
}

.button::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #0066cc;
  z-index: -1;
  transform: scaleY(0);
  transform-origin: bottom;
  transition: transform 300ms ease;
}

.button:hover {
  color: white;
}

.button:hover::before {
  transform: scaleY(1);
}
```

### Form Field Interactions

**1. Focus Ring Animation**
```css
.input {
  border: 2px solid #ddd;
  transition: border-color 200ms ease;
  position: relative;
}

.input::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background: #0066cc;
  transition: width 300ms ease;
}

.input:focus {
  outline: none;
  border-color: #0066cc;
}

.input:focus::after {
  width: 100%;
}
```

**2. Label Float Animation**
```css
.form-group {
  position: relative;
}

.input {
  padding: 1.5rem 0.75rem 0.5rem;
}

.label {
  position: absolute;
  top: 1rem;
  left: 0.75rem;
  transition: all 200ms ease;
  pointer-events: none;
  color: #666;
}

.input:focus ~ .label,
.input:not(:placeholder-shown) ~ .label {
  top: 0.25rem;
  font-size: 0.75rem;
  color: #0066cc;
}
```

**3. Error Shake**
```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-10px); }
  20%, 40%, 60%, 80% { transform: translateX(10px); }
}

.input.error {
  border-color: #dc3545;
  animation: shake 500ms ease-in-out;
}
```

**4. Success Checkmark**
```css
@keyframes checkmark {
  0% {
    stroke-dashoffset: 50;
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    stroke-dashoffset: 0;
  }
}

.success-icon {
  stroke-dasharray: 50;
  stroke-dashoffset: 50;
  animation: checkmark 600ms ease-out forwards;
}
```

### Loading States

**1. Spinner**
```css
@keyframes spin {
  to { transform: rotate(360deg); }
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #0066cc;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

**2. Pulse**
```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.loading {
  animation: pulse 2s ease-in-out infinite;
}
```

**3. Skeleton Screen**
```css
@keyframes shimmer {
  0% {
    background-position: -1000px 0;
  }
  100% {
    background-position: 1000px 0;
  }
}

.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 0px,
    #e0e0e0 40px,
    #f0f0f0 80px
  );
  background-size: 1000px 100%;
  animation: shimmer 2s infinite;
}
```

**4. Progress Bar**
```css
@keyframes progress {
  from { width: 0; }
  to { width: 100%; }
}

.progress-bar {
  height: 4px;
  background: #0066cc;
  animation: progress 2s ease-out forwards;
}

/* Indeterminate */
@keyframes indeterminate {
  0% {
    left: -35%;
    right: 100%;
  }
  60% {
    left: 100%;
    right: -90%;
  }
  100% {
    left: 100%;
    right: -90%;
  }
}

.progress-indeterminate::after {
  content: '';
  position: absolute;
  background: #0066cc;
  top: 0;
  bottom: 0;
  animation: indeterminate 2s infinite;
}
```

### Card Interactions

**1. Hover Lift**
```css
.card {
  transition: transform 300ms ease,
              box-shadow 300ms ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}
```

**2. Image Zoom on Hover**
```css
.card {
  overflow: hidden;
}

.card__image {
  transition: transform 400ms ease;
}

.card:hover .card__image {
  transform: scale(1.1);
}
```

**3. Reveal Effect**
```css
.card__overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.8);
  padding: 1rem;
  transform: translateY(100%);
  transition: transform 300ms ease;
}

.card:hover .card__overlay {
  transform: translateY(0);
}
```

### Icon Animations

**1. Hamburger to X**
```css
.hamburger {
  width: 30px;
  height: 3px;
  background: #333;
  position: relative;
  transition: background 200ms ease;
}

.hamburger::before,
.hamburger::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: #333;
  transition: transform 300ms ease;
}

.hamburger::before { top: -8px; }
.hamburger::after { top: 8px; }

.hamburger.active {
  background: transparent;
}

.hamburger.active::before {
  transform: rotate(45deg) translateY(8px);
}

.hamburger.active::after {
  transform: rotate(-45deg) translateY(-8px);
}
```

**2. Heart Beat**
```css
@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  10%, 30% { transform: scale(0.9); }
  20%, 40% { transform: scale(1.1); }
}

.icon-heart.liked {
  animation: heartbeat 800ms ease-in-out;
  fill: #e74c3c;
}
```

**3. Bounce**
```css
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-15px);
  }
  60% {
    transform: translateY(-7px);
  }
}

.icon {
  animation: bounce 1s ease infinite;
}
```

### Notification/Toast Animations

**1. Slide In**
```css
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

.toast {
  animation: slideInRight 300ms ease-out;
}
```

**2. Fade In Up**
```css
@keyframes fadeInUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.notification {
  animation: fadeInUp 400ms ease-out;
}
```

### Modal/Dialog Animations

**1. Scale In**
```css
@keyframes scaleIn {
  from {
    transform: scale(0.8);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

.modal {
  animation: scaleIn 300ms ease-out;
}
```

**2. Backdrop Fade**
```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-backdrop {
  animation: fadeIn 200ms ease;
}
```

## JavaScript Integration

### Toggle Classes for Animations

```javascript
// Add animation class
element.classList.add('animate');

// Remove after animation completes
element.addEventListener('animationend', () => {
  element.classList.remove('animate');
}, { once: true });

// Or use timeout
setTimeout(() => {
  element.classList.remove('animate');
}, 500);
```

### Intersection Observer for Scroll Animations

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('animate-in');
      observer.unobserve(entry.target);
    }
  });
}, {
  threshold: 0.1,
  rootMargin: '50px'
});

document.querySelectorAll('.animate-on-scroll').forEach(el => {
  observer.observe(el);
});
```

```css
.animate-on-scroll {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 600ms ease, transform 600ms ease;
}

.animate-on-scroll.animate-in {
  opacity: 1;
  transform: translateY(0);
}
```

### Stagger Animations

```javascript
function staggerAnimation(elements, delay = 100) {
  elements.forEach((el, index) => {
    setTimeout(() => {
      el.classList.add('animate-in');
    }, index * delay);
  });
}

// Usage
const items = document.querySelectorAll('.list-item');
staggerAnimation(items, 80);
```

```css
.list-item {
  opacity: 0;
  transform: translateX(-20px);
  transition: opacity 300ms ease, transform 300ms ease;
}

.list-item.animate-in {
  opacity: 1;
  transform: translateX(0);
}
```

## Advanced Techniques

### CSS Variables for Dynamic Animations

```css
.element {
  --duration: 300ms;
  --delay: 0ms;
  transition: transform var(--duration) ease var(--delay);
}

.element:hover {
  transform: translateY(-5px);
}
```

```javascript
// Change timing dynamically
element.style.setProperty('--duration', '500ms');
element.style.setProperty('--delay', '200ms');
```

### State Machines for Complex Interactions

```javascript
class StatefulButton {
  constructor(button) {
    this.button = button;
    this.state = 'idle'; // idle, loading, success, error
  }

  setState(newState) {
    this.button.classList.remove(this.state);
    this.state = newState;
    this.button.classList.add(newState);
  }

  async handleClick() {
    this.setState('loading');

    try {
      await someAsyncOperation();
      this.setState('success');

      setTimeout(() => {
        this.setState('idle');
      }, 2000);
    } catch (error) {
      this.setState('error');

      setTimeout(() => {
        this.setState('idle');
      }, 2000);
    }
  }
}
```

```css
.button.idle { /* Default state */ }
.button.loading { /* Show spinner */ }
.button.success { /* Show checkmark */ }
.button.error { /* Show error icon */ }
```

## Performance Optimization

### Use `will-change` Sparingly

```css
/* ✅ GOOD - Only when hovering */
.button:hover {
  will-change: transform;
  transform: scale(1.05);
}

/* ❌ BAD - Always on */
.button {
  will-change: transform;
}
```

### Batch Animations

```javascript
// ❌ BAD - Multiple reflows
elements.forEach(el => {
  el.style.transform = 'translateX(100px)';
  el.style.opacity = '0';
});

// ✅ GOOD - Single reflow
requestAnimationFrame(() => {
  elements.forEach(el => {
    el.style.transform = 'translateX(100px)';
    el.style.opacity = '0';
  });
});
```

### Debounce Resize/Scroll Handlers

```javascript
let resizeTimeout;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimeout);
  resizeTimeout = setTimeout(() => {
    // Update animations based on new size
  }, 100);
});
```

## Checklist for Great Micro-Interactions

- [ ] **Duration**: Under 300ms for most interactions
- [ ] **Purpose**: Every animation serves a function
- [ ] **Performance**: Use GPU-accelerated properties
- [ ] **Easing**: Natural, appropriate timing functions
- [ ] **Accessibility**: Respects `prefers-reduced-motion`
- [ ] **Subtlety**: Enhances without distracting
- [ ] **Consistency**: Matches brand personality
- [ ] **Feedback**: Confirms user actions clearly
- [ ] **Testing**: Works across browsers and devices
- [ ] **Mobile**: Performs well on lower-powered devices
