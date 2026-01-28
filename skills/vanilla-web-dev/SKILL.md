---
name: vanilla-web-dev
description: Expert guidance for vanilla HTML, CSS, and JavaScript development without frameworks. Use when building semantic HTML structures, creating scalable CSS architecture, implementing vanilla JavaScript functionality, or working on projects that don't use React/Vue/Angular. Covers responsive design, accessibility, performance optimization, and modern CSS features.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Vanilla Web Development

Comprehensive toolkit for building modern web applications using semantic HTML, scalable CSS architecture, and vanilla JavaScript - no frameworks required.

## Core Capabilities

### HTML Best Practices
- Semantic HTML5 structure and elements
- Accessibility (ARIA, WCAG compliance)
- SEO-friendly markup
- Progressive enhancement patterns
- Form validation and best practices

### CSS Architecture
- Scalable CSS organization (BEM, OOCSS, SMACSS)
- Modern CSS features (Grid, Flexbox, Custom Properties)
- Responsive design patterns
- Micro-interactions and animations for user engagement
- Performance optimization
- CSS naming conventions and methodology

### Vanilla JavaScript
- Modern ES6+ patterns and features
- DOM manipulation best practices
- Event handling and delegation
- Async patterns (Promises, async/await)
- Module patterns and code organization
- Browser API usage (Fetch, Storage, etc.)

## When to Use This Skill

Use this skill when:
- Building websites without React/Vue/Angular frameworks
- Creating lightweight, performant web pages
- Working with HTML email templates
- Building Chrome extensions or browser-based tools
- Implementing landing pages or marketing sites
- Working on projects where framework overhead isn't justified
- Learning web fundamentals
- Optimizing page performance and bundle size

## Quick Start Guide

### Project Structure

```
project/
├── index.html          # Entry point
├── css/
│   ├── reset.css      # CSS reset/normalize
│   ├── variables.css  # CSS custom properties
│   ├── base.css       # Base styles
│   ├── layout.css     # Layout patterns
│   ├── components/    # Component styles
│   └── utilities.css  # Utility classes
├── js/
│   ├── main.js        # Entry point
│   ├── utils/         # Utility functions
│   ├── components/    # Component modules
│   └── services/      # API/data services
└── assets/
    ├── images/
    ├── fonts/
    └── icons/
```

### CSS Methodology

This skill recommends BEM (Block Element Modifier) for CSS naming:

```css
/* Block */
.card { }

/* Element */
.card__header { }
.card__body { }
.card__footer { }

/* Modifier */
.card--featured { }
.card--small { }
```

### JavaScript Patterns

Modern vanilla JavaScript patterns for component-based development:

```javascript
// Component pattern
class Component {
  constructor(element) {
    this.element = element;
    this.init();
  }

  init() {
    this.attachEvents();
    this.render();
  }

  attachEvents() {
    // Event binding
  }

  render() {
    // DOM updates
  }
}

// Module pattern
const myModule = (() => {
  // Private
  const privateVar = 'secret';

  // Public API
  return {
    publicMethod() {
      // ...
    }
  };
})();
```

## Reference Documentation

### CSS Architecture Guide

Comprehensive CSS organization strategies available in `references/css-architecture.md`:

- Folder structure and file organization
- Naming conventions (BEM, OOCSS, SMACSS)
- CSS methodologies comparison
- Component-based CSS patterns
- Performance optimization strategies
- Team collaboration guidelines

### HTML Semantics Guide

Best practices for semantic HTML in `references/html-semantics.md`:

- HTML5 semantic elements guide
- Document structure patterns
- Accessibility best practices (ARIA)
- Form design and validation
- SEO optimization
- Progressive enhancement strategies

### Vanilla JavaScript Patterns

Modern JavaScript patterns in `references/vanilla-js-patterns.md`:

- ES6+ features and syntax
- DOM manipulation techniques
- Event handling patterns
- Async programming (Promises, async/await)
- Module patterns
- State management without frameworks
- Browser API usage guide

### Responsive Design Guide

Mobile-first responsive patterns in `references/responsive-design.md`:

- Mobile-first methodology
- Breakpoint strategies
- Responsive layouts (Grid, Flexbox)
- Touch and gesture handling
- Performance on mobile devices
- Progressive enhancement for devices

### Micro-Interactions & Animations

Comprehensive guide to engaging animations in `references/micro-interactions-animations.md`:

- Button hover effects (lift, scale, shine, fill)
- Form field interactions (focus rings, label floats, error shake)
- Loading states (spinners, pulse, skeleton screens)
- Card interactions (hover lift, image zoom, reveal)
- Icon animations (hamburger menu, heart beat, bounce)
- Modal/dialog animations
- Performance optimization techniques
- Accessibility with prefers-reduced-motion
- CSS and JavaScript integration patterns

## Common Workflows

### 1. Building a Component

```javascript
// 1. Create HTML structure
<div class="accordion" data-component="accordion">
  <div class="accordion__item">
    <button class="accordion__trigger">Title</button>
    <div class="accordion__content">Content</div>
  </div>
</div>

// 2. Style with BEM
.accordion { /* block */ }
.accordion__item { /* element */ }
.accordion__trigger { /* element */ }
.accordion__content { /* element */ }
.accordion__item--active { /* modifier */ }

// 3. Add JavaScript behavior
class Accordion {
  constructor(element) {
    this.element = element;
    this.items = this.element.querySelectorAll('.accordion__item');
    this.init();
  }

  init() {
    this.items.forEach(item => {
      const trigger = item.querySelector('.accordion__trigger');
      trigger.addEventListener('click', () => this.toggle(item));
    });
  }

  toggle(item) {
    item.classList.toggle('accordion__item--active');
  }
}

// Initialize
document.querySelectorAll('[data-component="accordion"]').forEach(el => {
  new Accordion(el);
});
```

### 2. Responsive Layout

```css
/* Mobile-first approach */
.container {
  padding: 1rem;
}

.grid {
  display: grid;
  gap: 1rem;
  /* Single column on mobile */
  grid-template-columns: 1fr;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
  }

  .grid {
    /* Two columns on tablet */
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }

  .grid {
    /* Three columns on desktop */
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### 3. Fetch API Pattern

```javascript
// Service module for API calls
const api = {
  baseURL: 'https://api.example.com',

  async get(endpoint) {
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`);
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      return await response.json();
    } catch (error) {
      console.error('API Error:', error);
      throw error;
    }
  },

  async post(endpoint, data) {
    try {
      const response = await fetch(`${this.baseURL}${endpoint}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      return await response.json();
    } catch (error) {
      console.error('API Error:', error);
      throw error;
    }
  }
};

// Usage
async function loadData() {
  const data = await api.get('/users');
  renderUsers(data);
}
```

## Best Practices

### Performance
- Minimize DOM manipulation (batch updates)
- Use event delegation for dynamic content
- Lazy load images and assets
- Debounce/throttle expensive operations
- Use CSS transforms for animations
- Minimize reflows and repaints

### Accessibility
- Use semantic HTML elements
- Provide ARIA labels where needed
- Ensure keyboard navigation works
- Maintain proper heading hierarchy
- Provide alt text for images
- Test with screen readers

### Code Organization
- Keep concerns separated (HTML/CSS/JS)
- Use consistent naming conventions
- Write modular, reusable code
- Comment complex logic
- Use modern ES6+ features
- Avoid global variables

### CSS Specificity
- Keep specificity low
- Avoid !important
- Use classes over IDs for styling
- Follow BEM naming to prevent conflicts
- Organize by specificity (low to high)

## Tools and Resources

### Development Tools
- Browser DevTools for debugging
- Lighthouse for performance auditing
- WAVE for accessibility testing
- Can I Use for browser support
- CSS/JS validators

### Build Tools (Optional)
- PostCSS for CSS processing
- Babel for JavaScript transpiling
- Webpack/Vite for bundling
- Prettier for code formatting
- ESLint for code quality

## Migration from Frameworks

When moving from React/Vue to vanilla:

1. **Components** → Classes or factory functions
2. **State Management** → Plain objects + events
3. **Props** → Data attributes or function parameters
4. **Lifecycle Methods** → Constructor + event listeners
5. **JSX/Templates** → Template literals or innerHTML
6. **Virtual DOM** → Direct DOM manipulation (optimized)

## Troubleshooting

### Common Issues

1. **Event listeners not working**
   - Check element exists in DOM before binding
   - Use event delegation for dynamic content
   - Ensure script runs after DOM loads

2. **CSS not applying**
   - Check specificity conflicts
   - Verify class names match BEM convention
   - Use DevTools to inspect computed styles

3. **JavaScript errors**
   - Check browser console for details
   - Verify ES6+ support or add polyfills
   - Use strict mode ('use strict')

## Resources

- Architecture Guide: `references/css-architecture.md`
- HTML Guide: `references/html-semantics.md`
- JavaScript Patterns: `references/vanilla-js-patterns.md`
- Responsive Design: `references/responsive-design.md`
