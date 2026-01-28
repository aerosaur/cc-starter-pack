# HTML Semantics and Best Practices

## HTML5 Semantic Elements

### Document Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Page description for SEO">
  <title>Page Title</title>
</head>
<body>
  <header>
    <nav>
      <!-- Primary navigation -->
    </nav>
  </header>

  <main>
    <article>
      <header>
        <h1>Article Title</h1>
        <p>Published on <time datetime="2025-01-15">January 15, 2025</time></p>
      </header>

      <section>
        <!-- Article content -->
      </section>

      <footer>
        <!-- Article metadata -->
      </footer>
    </article>

    <aside>
      <!-- Sidebar content -->
    </aside>
  </main>

  <footer>
    <!-- Site footer -->
  </footer>
</body>
</html>
```

### Semantic Elements Guide

| Element | Purpose | Example Use Case |
|---------|---------|------------------|
| `<header>` | Introductory content | Site header, article header |
| `<nav>` | Navigation links | Main menu, breadcrumbs |
| `<main>` | Main content | Primary page content (only one per page) |
| `<article>` | Self-contained content | Blog post, news article |
| `<section>` | Thematic grouping | Chapter, tab panel |
| `<aside>` | Tangentially related | Sidebar, callout box |
| `<footer>` | Footer content | Site footer, article footer |
| `<figure>` | Self-contained media | Image with caption |
| `<figcaption>` | Caption for figure | Image description |
| `<time>` | Date/time | Publication date |
| `<mark>` | Highlighted text | Search results highlight |
| `<details>` | Disclosure widget | FAQ accordion |
| `<summary>` | Summary for details | Accordion trigger |

## Accessibility (ARIA)

### ARIA Roles

```html
<!-- Landmark roles -->
<div role="banner">Site header</div>
<div role="navigation">Nav menu</div>
<div role="main">Main content</div>
<div role="complementary">Sidebar</div>
<div role="contentinfo">Footer</div>

<!-- Widget roles -->
<div role="button" tabindex="0">Custom button</div>
<div role="tab" aria-selected="true">Tab 1</div>
<div role="tabpanel">Tab content</div>
<div role="dialog" aria-modal="true">Modal</div>
```

### ARIA States and Properties

```html
<!-- Expanded/Collapsed -->
<button aria-expanded="false" aria-controls="menu">
  Menu
</button>
<div id="menu" aria-hidden="true">
  Menu content
</div>

<!-- Labels and Descriptions -->
<button aria-label="Close dialog">×</button>
<input aria-labelledby="label-id" aria-describedby="desc-id">
<label id="label-id">Email</label>
<p id="desc-id">We'll never share your email</p>

<!-- Live Regions -->
<div aria-live="polite" role="status">
  Status message will be announced
</div>
<div aria-live="assertive" role="alert">
  Error message will interrupt
</div>

<!-- Required/Invalid -->
<input type="email" required aria-required="true" aria-invalid="false">
<span role="alert" aria-live="assertive">
  Invalid email address
</span>
```

### Focus Management

```html
<!-- Tab order -->
<button tabindex="0">Natural order</button>
<div tabindex="0">Focusable div</div>
<div tabindex="-1">Programmatically focusable</div>

<!-- Skip links -->
<a href="#main-content" class="skip-link">
  Skip to main content
</a>
<main id="main-content">
  <!-- Content -->
</main>
```

## Forms Best Practices

### Accessible Forms

```html
<form>
  <!-- Text Input -->
  <div class="form-group">
    <label for="name">
      Name
      <span aria-label="required">*</span>
    </label>
    <input
      type="text"
      id="name"
      name="name"
      required
      aria-required="true"
      aria-describedby="name-hint"
    >
    <small id="name-hint">Enter your full name</small>
  </div>

  <!-- Email with Validation -->
  <div class="form-group">
    <label for="email">Email</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      aria-invalid="false"
      aria-describedby="email-error"
    >
    <span id="email-error" role="alert" aria-live="assertive">
      <!-- Error message inserted here -->
    </span>
  </div>

  <!-- Select -->
  <div class="form-group">
    <label for="country">Country</label>
    <select id="country" name="country">
      <option value="">Select a country</option>
      <option value="us">United States</option>
      <option value="uk">United Kingdom</option>
    </select>
  </div>

  <!-- Radio Buttons -->
  <fieldset>
    <legend>Choose a plan</legend>
    <div class="form-check">
      <input type="radio" id="basic" name="plan" value="basic">
      <label for="basic">Basic</label>
    </div>
    <div class="form-check">
      <input type="radio" id="pro" name="plan" value="pro">
      <label for="pro">Pro</label>
    </div>
  </fieldset>

  <!-- Checkboxes -->
  <div class="form-check">
    <input type="checkbox" id="terms" name="terms" required>
    <label for="terms">
      I agree to the <a href="/terms">terms and conditions</a>
    </label>
  </div>

  <button type="submit">Submit</button>
</form>
```

### Form Validation Pattern

```javascript
class FormValidator {
  constructor(form) {
    this.form = form;
    this.init();
  }

  init() {
    this.form.noValidate = true; // Disable browser validation
    this.form.addEventListener('submit', (e) => this.handleSubmit(e));

    // Real-time validation
    this.form.querySelectorAll('input, select, textarea').forEach(field => {
      field.addEventListener('blur', () => this.validateField(field));
      field.addEventListener('input', () => {
        if (field.classList.contains('invalid')) {
          this.validateField(field);
        }
      });
    });
  }

  handleSubmit(e) {
    e.preventDefault();

    const isValid = this.validateForm();
    if (isValid) {
      // Submit form
      this.form.submit();
    } else {
      // Focus first invalid field
      const firstInvalid = this.form.querySelector('.invalid');
      firstInvalid?.focus();
    }
  }

  validateForm() {
    let isValid = true;
    this.form.querySelectorAll('input, select, textarea').forEach(field => {
      if (!this.validateField(field)) {
        isValid = false;
      }
    });
    return isValid;
  }

  validateField(field) {
    const validity = field.validity;
    const errorElement = document.getElementById(`${field.id}-error`);

    if (validity.valid) {
      field.classList.remove('invalid');
      field.setAttribute('aria-invalid', 'false');
      if (errorElement) {
        errorElement.textContent = '';
      }
      return true;
    } else {
      field.classList.add('invalid');
      field.setAttribute('aria-invalid', 'true');

      // Get error message
      let message = '';
      if (validity.valueMissing) {
        message = 'This field is required';
      } else if (validity.typeMismatch) {
        message = `Please enter a valid ${field.type}`;
      } else if (validity.tooShort) {
        message = `Minimum length is ${field.minLength} characters`;
      } else if (validity.patternMismatch) {
        message = field.getAttribute('data-pattern-message') || 'Invalid format';
      }

      if (errorElement) {
        errorElement.textContent = message;
      }
      return false;
    }
  }
}

// Usage
document.querySelectorAll('form').forEach(form => {
  new FormValidator(form);
});
```

## SEO Best Practices

### Meta Tags

```html
<head>
  <!-- Essential -->
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title - Site Name</title>
  <meta name="description" content="150-160 character description">

  <!-- Open Graph (Facebook, LinkedIn) -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="Description">
  <meta property="og:image" content="https://example.com/image.jpg">
  <meta property="og:url" content="https://example.com/page">
  <meta property="og:type" content="website">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Page Title">
  <meta name="twitter:description" content="Description">
  <meta name="twitter:image" content="https://example.com/image.jpg">

  <!-- Canonical URL -->
  <link rel="canonical" href="https://example.com/page">

  <!-- Structured Data -->
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Article Title",
    "author": {
      "@type": "Person",
      "name": "Author Name"
    },
    "datePublished": "2025-01-15"
  }
  </script>
</head>
```

### Heading Hierarchy

```html
<!-- Good hierarchy -->
<h1>Page Title</h1>
  <h2>Main Section</h2>
    <h3>Subsection</h3>
    <h3>Another Subsection</h3>
  <h2>Another Section</h2>
    <h3>Subsection</h3>

<!-- Bad: Skipping levels -->
<h1>Title</h1>
  <h3>Oops, skipped h2</h3>

<!-- Bad: Multiple h1s -->
<h1>First Title</h1>
<h1>Second Title</h1>
```

### Image Optimization

```html
<!-- Responsive images -->
<img
  src="image-800.jpg"
  srcset="image-400.jpg 400w,
          image-800.jpg 800w,
          image-1200.jpg 1200w"
  sizes="(max-width: 400px) 100vw,
         (max-width: 800px) 50vw,
         800px"
  alt="Descriptive alt text"
  loading="lazy"
  width="800"
  height="600"
>

<!-- Picture element for art direction -->
<picture>
  <source media="(min-width: 800px)" srcset="wide.jpg">
  <source media="(min-width: 400px)" srcset="medium.jpg">
  <img src="narrow.jpg" alt="Description">
</picture>
```

## Progressive Enhancement

### HTML First Approach

```html
<!-- Works without JavaScript -->
<details>
  <summary>Click to expand</summary>
  <p>Content is accessible even without JavaScript</p>
</details>

<!-- Form works without JS -->
<form action="/submit" method="POST">
  <input type="text" name="query" required>
  <button type="submit">Search</button>
</form>

<!-- Links work without JS -->
<a href="/page">Link</a>

<!-- Then enhance with JavaScript -->
<script>
  // Add JS enhancements
  document.querySelector('a').addEventListener('click', (e) => {
    // Enhance with AJAX, but link still works if JS fails
  });
</script>
```

## Common Patterns

### Modal Dialog

```html
<dialog id="my-dialog" aria-labelledby="dialog-title">
  <header>
    <h2 id="dialog-title">Dialog Title</h2>
    <button type="button" aria-label="Close dialog">×</button>
  </header>
  <div>
    Dialog content
  </div>
  <footer>
    <button type="button">Cancel</button>
    <button type="button">Confirm</button>
  </footer>
</dialog>

<button onclick="document.getElementById('my-dialog').showModal()">
  Open Dialog
</button>
```

### Card Component

```html
<article class="card">
  <img src="image.jpg" alt="Card image" class="card__image">
  <div class="card__body">
    <h3 class="card__title">Card Title</h3>
    <p class="card__text">Card description text</p>
    <a href="/link" class="card__link">Read more</a>
  </div>
</article>
```

### Navigation Menu

```html
<nav aria-label="Main navigation">
  <button
    class="nav-toggle"
    aria-expanded="false"
    aria-controls="nav-menu"
  >
    Menu
  </button>
  <ul id="nav-menu" class="nav-menu">
    <li><a href="/" aria-current="page">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>
```
