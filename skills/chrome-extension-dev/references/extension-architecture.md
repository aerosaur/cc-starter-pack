# Chrome Extension Architecture Patterns

## Component Communication

### Message Passing Architecture

Chrome extensions consist of isolated components that communicate via message passing:

**Components:**
- Service Worker (background.js) - Event-driven, ephemeral
- Content Scripts - Injected into web pages
- Popup - Extension toolbar UI
- Options Page - Settings UI
- Offscreen Documents - Hidden pages for special tasks

**Communication Patterns:**

```javascript
// 1. One-time messages
chrome.runtime.sendMessage({ type: 'ACTION', data: {} }, (response) => {
  console.log(response);
});

// 2. Long-lived connections
const port = chrome.runtime.connect({ name: 'my-channel' });
port.postMessage({ data: 'hello' });
port.onMessage.addListener((msg) => {
  console.log('Received:', msg);
});

// 3. Content script → Service worker
chrome.runtime.sendMessage({ action: 'getData' }, (response) => {
  // Handle response
});

// 4. Service worker → Content script
chrome.tabs.sendMessage(tabId, { action: 'updateUI' }, (response) => {
  // Handle response
});

// 5. Content script → Content script (same page)
window.postMessage({ type: 'FROM_EXTENSION', data: {} }, '*');
```

### Service Worker Lifecycle

**Ephemeral Nature:**
- Service workers terminate after ~30 seconds of inactivity
- Chrome can wake them for events (messages, alarms, tab updates)
- Never rely on global variables persisting
- Always store state in chrome.storage

**Lifecycle Events:**

```javascript
// Extension installed/updated
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'install') {
    // First install
  } else if (details.reason === 'update') {
    // Extension updated
  }
});

// Service worker starting up
chrome.runtime.onStartup.addListener(() => {
  // Extension started (browser launch)
});

// Service worker suspending (cannot prevent)
// Use chrome.storage for persistence
```

**Keeping Service Worker Alive (Anti-pattern):**

```javascript
// ❌ BAD: Trying to keep service worker alive
setInterval(() => {
  console.log('keeping alive');
}, 20000);

// ✅ GOOD: Use alarms for scheduled tasks
chrome.alarms.create('myAlarm', { periodInMinutes: 1 });
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'myAlarm') {
    // Do periodic work
  }
});
```

## Content Script Patterns

### Injection Strategies

**Declarative (manifest.json):**

```json
{
  "content_scripts": [
    {
      "matches": ["https://example.com/*"],
      "js": ["content.js"],
      "css": ["content.css"],
      "run_at": "document_idle"
    }
  ]
}
```

**Programmatic (MV3):**

```javascript
// Inject script
chrome.scripting.executeScript({
  target: { tabId: tabId, allFrames: false },
  files: ['content.js']
});

// Inject function
chrome.scripting.executeScript({
  target: { tabId: tabId },
  func: () => {
    document.body.style.background = 'red';
  },
  args: [] // Arguments to pass to function
});

// Inject CSS
chrome.scripting.insertCSS({
  target: { tabId: tabId },
  files: ['styles.css']
});
```

### Isolated World vs Page Context

Content scripts run in an "isolated world":
- Separate JavaScript execution environment
- Shares DOM with page but not JavaScript variables
- Cannot directly access page's JavaScript objects

**Accessing Page Context:**

```javascript
// Content script injects code into page context
const script = document.createElement('script');
script.textContent = `
  // This code runs in page context
  console.log('Page variable:', window.pageVar);
`;
document.documentElement.appendChild(script);
script.remove();

// Communicate via DOM events
window.addEventListener('message', (event) => {
  if (event.source !== window) return;
  if (event.data.type === 'FROM_PAGE') {
    // Handle message from page
  }
});
```

## State Management

### Storage Patterns

**Local Storage (per-machine):**

```javascript
// Single key
await chrome.storage.local.set({ key: 'value' });
const result = await chrome.storage.local.get('key');

// Multiple keys
await chrome.storage.local.set({ key1: 'a', key2: 'b' });
const data = await chrome.storage.local.get(['key1', 'key2']);

// All keys
const allData = await chrome.storage.local.get(null);

// Remove keys
await chrome.storage.local.remove('key');
await chrome.storage.local.clear(); // Remove all
```

**Sync Storage (synced across devices):**

```javascript
// Limited to 100KB total, 8KB per item
await chrome.storage.sync.set({ settings: { theme: 'dark' } });
const { settings } = await chrome.storage.sync.get('settings');
```

**Storage Change Listener:**

```javascript
chrome.storage.onChanged.addListener((changes, areaName) => {
  for (let key in changes) {
    const change = changes[key];
    console.log(
      `Key "${key}" in ${areaName} changed:`,
      `Old: ${change.oldValue}`,
      `New: ${change.newValue}`
    );
  }
});
```

### State Architecture Patterns

**1. Service Worker as Source of Truth:**

```javascript
// background.js
let state = {
  count: 0,
  items: []
};

chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  switch (msg.action) {
    case 'getState':
      sendResponse(state);
      break;
    case 'updateState':
      state = { ...state, ...msg.data };
      // Notify all listeners
      chrome.runtime.sendMessage({ type: 'stateChanged', state });
      sendResponse({ success: true });
      break;
  }
  return true;
});
```

**2. Storage as Source of Truth:**

```javascript
// Any component can update
async function updateState(updates) {
  const current = await chrome.storage.local.get('state');
  const newState = { ...current.state, ...updates };
  await chrome.storage.local.set({ state: newState });
}

// All components listen for changes
chrome.storage.onChanged.addListener((changes, area) => {
  if (area === 'local' && changes.state) {
    const newState = changes.state.newValue;
    // Update UI accordingly
  }
});
```

## Error Handling

### Service Worker Error Handling

```javascript
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  (async () => {
    try {
      const result = await someAsyncOperation(msg.data);
      sendResponse({ success: true, data: result });
    } catch (error) {
      console.error('Error:', error);
      sendResponse({ success: false, error: error.message });
    }
  })();
  return true; // Required for async
});
```

### Content Script Error Handling

```javascript
// Wrap in try-catch to prevent page script errors
try {
  // Extension code
  chrome.runtime.sendMessage({ data: 'test' }, (response) => {
    if (chrome.runtime.lastError) {
      console.error('Message error:', chrome.runtime.lastError);
      return;
    }
    // Handle response
  });
} catch (error) {
  console.error('Content script error:', error);
}
```

### Runtime Error Detection

```javascript
// Check if extension context is still valid
function isContextValid() {
  return chrome.runtime?.id !== undefined;
}

// Use before any chrome.* API calls in content scripts
if (isContextValid()) {
  chrome.runtime.sendMessage({ ... });
}
```

## Performance Optimization

### Lazy Loading

```javascript
// Load heavy scripts only when needed
async function loadHeavyFeature() {
  if (!window.heavyFeature) {
    await import('./heavy-feature.js');
  }
  return window.heavyFeature;
}
```

### Batching Operations

```javascript
// ❌ BAD: Multiple storage writes
for (let item of items) {
  await chrome.storage.local.set({ [item.id]: item });
}

// ✅ GOOD: Single batch write
const batch = {};
for (let item of items) {
  batch[item.id] = item;
}
await chrome.storage.local.set(batch);
```

### Content Script Optimization

```javascript
// Minimize content script size
// Load only when needed
chrome.action.onClicked.addListener((tab) => {
  chrome.scripting.executeScript({
    target: { tabId: tab.id },
    files: ['content.js'] // Loaded on demand
  });
});
```

## Advanced Patterns

### Offscreen Documents (MV3)

For audio, canvas, or DOM operations not available in service workers:

```javascript
// background.js
async function createOffscreen() {
  await chrome.offscreen.createDocument({
    url: 'offscreen.html',
    reasons: ['AUDIO_PLAYBACK'],
    justification: 'Playing notification sound'
  });
}

// Communicate with offscreen document
chrome.runtime.sendMessage({ target: 'offscreen', action: 'play' });
```

### Extension Page Communication

```javascript
// Get all extension views
chrome.extension.getViews({ type: 'popup' }).forEach((view) => {
  // Direct access to window object
  view.updateUI();
});
```

### Tab Communication

```javascript
// Message all tabs
chrome.tabs.query({}, (tabs) => {
  tabs.forEach((tab) => {
    chrome.tabs.sendMessage(tab.id, { action: 'update' });
  });
});

// Message specific tab
chrome.tabs.sendMessage(tabId, { data: 'test' }, (response) => {
  if (chrome.runtime.lastError) {
    // Tab doesn't have content script or was closed
    console.log('Cannot message tab');
  }
});
```

## Debugging Patterns

### Logging

```javascript
// Service worker logs
console.log('[SW]', 'Message'); // chrome://extensions/ → Inspect

// Content script logs
console.log('[CS]', 'Message'); // Page DevTools console

// Popup logs
console.log('[Popup]', 'Message'); // Right-click icon → Inspect
```

### Error Tracking

```javascript
// Global error handler in service worker
self.addEventListener('error', (event) => {
  console.error('Uncaught error:', event.error);
});

// Global error handler in content scripts
window.addEventListener('error', (event) => {
  console.error('Content script error:', event.error);
});
```

### Performance Monitoring

```javascript
// Measure operation time
const start = performance.now();
await expensiveOperation();
const duration = performance.now() - start;
console.log(`Operation took ${duration}ms`);
```
