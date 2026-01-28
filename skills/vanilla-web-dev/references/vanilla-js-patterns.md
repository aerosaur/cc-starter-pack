# Vanilla JavaScript Patterns and Best Practices

## Modern ES6+ Features

### Let, Const, and Block Scope

```javascript
// Use const by default
const PI = 3.14159;
const user = { name: 'John' };

// Use let when reassignment is needed
let counter = 0;
counter++;

// Avoid var (function-scoped, hoisting issues)
// var oldStyle = 'avoid this';
```

### Arrow Functions

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// With block body
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// No parameters
const getRandomNumber = () => Math.random();

// Single parameter (parentheses optional)
const double = x => x * 2;
```

### Template Literals

```javascript
const name = 'Alice';
const age = 30;

// String interpolation
const greeting = `Hello, ${name}! You are ${age} years old.`;

// Multi-line strings
const html = `
  <div class="card">
    <h2>${name}</h2>
    <p>Age: ${age}</p>
  </div>
`;

// Expression interpolation
const message = `Sum: ${10 + 20}`;
```

### Destructuring

```javascript
// Object destructuring
const user = { name: 'John', age: 30, city: 'NYC' };
const { name, age } = user;

// With renaming
const { name: userName, age: userAge } = user;

// With defaults
const { name, country = 'USA' } = user;

// Array destructuring
const colors = ['red', 'green', 'blue'];
const [first, second] = colors;

// Skip elements
const [, , third] = colors;

// Rest operator
const [head, ...rest] = colors;
```

### Spread Operator

```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];

// Object spread
const defaults = { theme: 'dark', lang: 'en' };
const userPrefs = { lang: 'es' };
const settings = { ...defaults, ...userPrefs };

// Copy arrays/objects
const arrCopy = [...arr1];
const objCopy = { ...defaults };

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers); // Same as Math.max(1, 2, 3)
```

### Default Parameters

```javascript
function greet(name = 'Guest', greeting = 'Hello') {
  return `${greeting}, ${name}!`;
}

greet(); // "Hello, Guest!"
greet('Alice'); // "Hello, Alice!"
greet('Bob', 'Hi'); // "Hi, Bob!"
```

## Component Patterns

### Class-Based Components

```javascript
class Component {
  constructor(element, options = {}) {
    this.element = element;
    this.options = {
      autoInit: true,
      ...options
    };

    if (this.options.autoInit) {
      this.init();
    }
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

  destroy() {
    // Cleanup
  }
}

// Usage
const component = new Component(document.querySelector('.my-component'), {
  autoInit: false
});
```

### Factory Pattern

```javascript
function createComponent(element, options) {
  let state = {
    isActive: false,
    ...options
  };

  function init() {
    attachEvents();
    render();
  }

  function attachEvents() {
    element.addEventListener('click', handleClick);
  }

  function handleClick() {
    state.isActive = !state.isActive;
    render();
  }

  function render() {
    element.classList.toggle('active', state.isActive);
  }

  function destroy() {
    element.removeEventListener('click', handleClick);
  }

  // Public API
  return {
    init,
    destroy,
    get state() { return { ...state }; }
  };
}

// Usage
const component = createComponent(document.querySelector('.my-component'));
component.init();
```

### Module Pattern

```javascript
const MyModule = (() => {
  // Private variables
  let privateVar = 'secret';
  let count = 0;

  // Private functions
  function privateFunction() {
    console.log(privateVar);
  }

  // Public API
  return {
    increment() {
      count++;
      return count;
    },

    getCount() {
      return count;
    },

    reset() {
      count = 0;
    }
  };
})();

// Usage
MyModule.increment(); // 1
MyModule.getCount(); // 1
// MyModule.privateVar; // undefined
```

## DOM Manipulation

### Selecting Elements

```javascript
// Single element
const element = document.querySelector('.my-class');
const byId = document.getElementById('my-id');

// Multiple elements (returns NodeList)
const elements = document.querySelectorAll('.my-class');

// Modern approach with optional chaining
const maybeElement = document.querySelector('.maybe')?.textContent;

// Converting NodeList to Array
const elementsArray = Array.from(elements);
const elementsArray2 = [...elements];
```

### Creating Elements

```javascript
// Create and configure
const div = document.createElement('div');
div.className = 'card';
div.id = 'card-1';
div.textContent = 'Card content';

// Set attributes
div.setAttribute('data-id', '123');
div.setAttribute('aria-label', 'Card');

// Add styles
div.style.backgroundColor = '#f0f0f0';
div.style.padding = '1rem';

// Append to DOM
document.body.appendChild(div);

// Template literal approach
const container = document.querySelector('.container');
container.innerHTML = `
  <div class="card" data-id="123">
    <h2>Title</h2>
    <p>Content</p>
  </div>
`;
```

### Modifying Elements

```javascript
const element = document.querySelector('.my-element');

// Text content
element.textContent = 'New text';
element.innerHTML = '<strong>Bold text</strong>';

// Classes
element.classList.add('active');
element.classList.remove('inactive');
element.classList.toggle('visible');
element.classList.contains('active'); // boolean

// Attributes
element.getAttribute('data-id');
element.setAttribute('data-id', '123');
element.removeAttribute('data-id');
element.hasAttribute('data-id'); // boolean

// Dataset (data-* attributes)
element.dataset.id = '123'; // Sets data-id
const id = element.dataset.id; // Gets data-id

// Styles
element.style.color = 'red';
element.style.backgroundColor = '#fff';
```

## Event Handling

### Adding Event Listeners

```javascript
const button = document.querySelector('button');

// Basic event listener
button.addEventListener('click', function(e) {
  console.log('Clicked!', e);
});

// Arrow function
button.addEventListener('click', (e) => {
  console.log('Clicked!', e);
});

// Named handler (for removal)
function handleClick(e) {
  console.log('Clicked!', e);
}
button.addEventListener('click', handleClick);

// Remove event listener
button.removeEventListener('click', handleClick);

// Event options
button.addEventListener('click', handleClick, {
  once: true,      // Run once then remove
  passive: true,   // Won't call preventDefault
  capture: false   // Bubble phase (default)
});
```

### Event Delegation

```javascript
// Instead of attaching to each item
const list = document.querySelector('.list');

list.addEventListener('click', (e) => {
  // Check if clicked element matches selector
  if (e.target.matches('.list-item')) {
    console.log('Item clicked:', e.target);
  }

  // Or use closest for nested elements
  const item = e.target.closest('.list-item');
  if (item) {
    console.log('Item clicked:', item);
  }
});
```

### Common Event Patterns

```javascript
// Prevent default
form.addEventListener('submit', (e) => {
  e.preventDefault();
  // Handle form submission
});

// Stop propagation
element.addEventListener('click', (e) => {
  e.stopPropagation();
  // Prevents event from bubbling up
});

// Debounce
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// Usage
const searchInput = document.querySelector('input');
const debouncedSearch = debounce((e) => {
  console.log('Searching for:', e.target.value);
}, 300);

searchInput.addEventListener('input', debouncedSearch);

// Throttle
function throttle(func, wait) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, wait);
    }
  };
}

// Usage
window.addEventListener('scroll', throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100));
```

## Async Patterns

### Promises

```javascript
// Creating a promise
function fetchData(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve({ data: 'Success!' });
      } else {
        reject(new Error('URL required'));
      }
    }, 1000);
  });
}

// Using promises
fetchData('/api/data')
  .then(response => {
    console.log(response.data);
    return response.data;
  })
  .then(data => {
    console.log('Processed:', data);
  })
  .catch(error => {
    console.error('Error:', error);
  })
  .finally(() => {
    console.log('Done!');
  });

// Promise.all (parallel execution)
Promise.all([
  fetchData('/api/users'),
  fetchData('/api/posts'),
  fetchData('/api/comments')
])
  .then(([users, posts, comments]) => {
    console.log({ users, posts, comments });
  })
  .catch(error => {
    console.error('One failed:', error);
  });

// Promise.race (first to complete)
Promise.race([
  fetchData('/api/fast'),
  fetchData('/api/slow')
])
  .then(result => {
    console.log('First result:', result);
  });
```

### Async/Await

```javascript
// Async function
async function loadData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// Usage
loadData()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Or await in another async function
async function init() {
  const data = await loadData();
  console.log(data);
}

// Parallel execution with async/await
async function loadAllData() {
  try {
    const [users, posts] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json())
    ]);
    return { users, posts };
  } catch (error) {
    console.error('Error:', error);
  }
}

// Sequential execution
async function loadSequential() {
  const users = await fetch('/api/users').then(r => r.json());
  const posts = await fetch('/api/posts').then(r => r.json());
  return { users, posts };
}
```

### Fetch API

```javascript
// GET request
async function getData(url) {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// POST request
async function postData(url, data) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error('Post error:', error);
    throw error;
  }
}

// With timeout
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeout);

  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    clearTimeout(id);
    return await response.json();
  } catch (error) {
    clearTimeout(id);
    if (error.name === 'AbortError') {
      throw new Error('Request timeout');
    }
    throw error;
  }
}
```

## State Management

### Simple State Manager

```javascript
class Store {
  constructor(initialState = {}) {
    this.state = initialState;
    this.listeners = [];
  }

  getState() {
    return { ...this.state };
  }

  setState(newState) {
    this.state = {
      ...this.state,
      ...newState
    };
    this.notify();
  }

  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }

  notify() {
    this.listeners.forEach(listener => {
      listener(this.state);
    });
  }
}

// Usage
const store = new Store({ count: 0 });

const unsubscribe = store.subscribe((state) => {
  console.log('State changed:', state);
  // Update UI
});

store.setState({ count: 1 });
store.setState({ count: 2 });

unsubscribe(); // Stop listening
```

### Observable Pattern

```javascript
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
    return () => {
      this.observers = this.observers.filter(obs => obs !== observer);
    };
  }

  notify(data) {
    this.observers.forEach(observer => observer(data));
  }
}

// Usage
const dataStream = new Observable();

const unsub = dataStream.subscribe((data) => {
  console.log('Received:', data);
});

dataStream.notify({ event: 'update', value: 123 });
```

## Browser APIs

### Local Storage

```javascript
// Store data
localStorage.setItem('user', JSON.stringify({ name: 'John' }));

// Retrieve data
const user = JSON.parse(localStorage.getItem('user'));

// Remove item
localStorage.removeItem('user');

// Clear all
localStorage.clear();

// Check if item exists
if (localStorage.getItem('user')) {
  // Item exists
}

// Helper functions
const storage = {
  set(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  },

  get(key) {
    const item = localStorage.getItem(key);
    try {
      return JSON.parse(item);
    } catch {
      return item;
    }
  },

  remove(key) {
    localStorage.removeItem(key);
  },

  clear() {
    localStorage.clear();
  }
};
```

### Intersection Observer

```javascript
// Lazy load images
const imageObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.add('loaded');
      imageObserver.unobserve(img);
    }
  });
}, {
  threshold: 0.1,
  rootMargin: '50px'
});

// Observe all images with data-src
document.querySelectorAll('img[data-src]').forEach(img => {
  imageObserver.observe(img);
});
```

### Mutation Observer

```javascript
// Watch for DOM changes
const observer = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    if (mutation.type === 'childList') {
      console.log('Children changed:', mutation.addedNodes);
    }
    if (mutation.type === 'attributes') {
      console.log('Attribute changed:', mutation.attributeName);
    }
  });
});

// Start observing
observer.observe(document.body, {
  childList: true,
  subtree: true,
  attributes: true,
  attributeOldValue: true
});

// Stop observing
observer.disconnect();
```

## Error Handling

### Try-Catch

```javascript
// Synchronous errors
try {
  const result = riskyOperation();
  console.log(result);
} catch (error) {
  console.error('Error:', error.message);
} finally {
  console.log('Cleanup');
}

// Async errors
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
    // Rethrow or handle
    throw new Error('Failed to load data');
  }
}

// Custom error classes
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

function validate(data) {
  if (!data.email) {
    throw new ValidationError('Email required');
  }
}

try {
  validate({});
} catch (error) {
  if (error instanceof ValidationError) {
    console.error('Validation failed:', error.message);
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Performance Tips

```javascript
// 1. Cache DOM queries
const element = document.querySelector('.my-element');
// Reuse element instead of querying again

// 2. Use document fragments for multiple appends
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}
document.body.appendChild(fragment); // Single reflow

// 3. Debounce expensive operations
const expensiveOperation = debounce(() => {
  // Expensive calculation
}, 300);

// 4. Use event delegation
document.body.addEventListener('click', (e) => {
  if (e.target.matches('.button')) {
    // Handle click
  }
});

// 5. Lazy initialization
class LazyComponent {
  constructor(element) {
    this.element = element;
    // Don't initialize until needed
  }

  init() {
    if (this.initialized) return;
    // Heavy initialization here
    this.initialized = true;
  }
}
```
