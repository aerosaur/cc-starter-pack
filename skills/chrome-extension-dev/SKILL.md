---
name: chrome-extension-dev
description: Expert guidance for building Chrome extensions using Manifest V3. Use when creating browser extensions, working with Chrome APIs (tabs, storage, bookmarks, windows), implementing content scripts, background service workers, popup UIs, or publishing to Chrome Web Store. Covers architecture patterns, security, permissions, and debugging.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Chrome Extension Development

Comprehensive toolkit for building modern Chrome extensions using Manifest V3, covering architecture, APIs, security, and deployment patterns.

## Core Capabilities

### Manifest V3 Fundamentals
- Manifest.json structure and configuration
- Service workers (replacing background pages)
- Content scripts and injection strategies
- Popup and options page development
- Permissions and host permissions
- Web-accessible resources

### Chrome APIs
- chrome.tabs - Tab management and communication
- chrome.storage - Local and sync storage
- chrome.runtime - Messaging and lifecycle
- chrome.action - Browser action and toolbar icons
- chrome.scripting - Dynamic script injection
- chrome.webNavigation - Navigation events
- chrome.bookmarks - Bookmark management
- chrome.windows - Window management
- chrome.alarms - Scheduled tasks

### Extension Architecture
- Message passing between components
- State management patterns
- Content script isolation and communication
- Service worker lifecycle management
- Error handling and debugging
- Performance optimization

## When to Use This Skill

Use this skill when:
- Building new Chrome extensions from scratch
- Migrating Manifest V2 extensions to V3
- Implementing browser automation or enhancement tools
- Working with Chrome APIs for tab/window management
- Creating productivity tools or content modifiers
- Publishing extensions to Chrome Web Store
- Debugging extension-specific issues
- Implementing OAuth or authentication flows in extensions

## Quick Start Guide

### Basic Extension Structure

```
extension/
├── manifest.json           # Extension configuration
├── background.js          # Service worker
├── content.js            # Content script
├── popup/
│   ├── popup.html        # Popup UI
│   ├── popup.js          # Popup logic
│   └── popup.css         # Popup styles
├── options/
│   ├── options.html      # Settings page
│   └── options.js        # Settings logic
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
└── lib/                  # Shared utilities
```

### Minimal Manifest V3

```json
{
  "manifest_version": 3,
  "name": "Extension Name",
  "version": "1.0.0",
  "description": "Extension description",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "permissions": [
    "storage",
    "activeTab"
  ],
  "host_permissions": [
    "https://*/*"
  ],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_idle"
    }
  ]
}
```

## Reference Documentation

### Chrome Extension Architecture

Detailed architecture patterns available in `references/extension-architecture.md`:

- Component communication patterns
- Service worker lifecycle and persistence
- Content script injection strategies
- Message passing between components
- State management across extension components
- Error handling and logging patterns

### Chrome APIs Reference

Comprehensive API documentation in `references/chrome-apis.md`:

- chrome.tabs API with examples
- chrome.storage API patterns
- chrome.runtime messaging
- chrome.scripting dynamic injection
- chrome.action toolbar integration
- chrome.webNavigation event handling
- Full API reference with code samples

### Security Best Practices

Security guidelines in `references/security-practices.md`:

- Content Security Policy (CSP) configuration
- Permissions best practices
- XSS prevention in extensions
- Secure message passing
- OAuth and authentication patterns
- Web-accessible resources security

### Chrome Web Store Publishing

Publishing guide in `references/publishing-guide.md`:

- Store listing requirements
- Privacy policy requirements
- Screenshot and asset specifications
- Review process and common rejections
- Manifest requirements for store submission
- Post-publication updates and versioning

## Common Workflows

### 1. Message Passing: Content Script ↔ Service Worker

```javascript
// content.js - Send message to service worker
chrome.runtime.sendMessage(
  { action: 'getData', url: window.location.href },
  (response) => {
    console.log('Response:', response);
  }
);

// background.js - Listen for messages
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'getData') {
    // Process request
    const data = { result: 'success' };
    sendResponse(data);
  }
  return true; // Required for async sendResponse
});
```

### 2. Storage API Pattern

```javascript
// Save to storage
chrome.storage.local.set({ key: 'value' }, () => {
  console.log('Saved');
});

// Get from storage
chrome.storage.local.get(['key'], (result) => {
  console.log('Value:', result.key);
});

// Listen for storage changes
chrome.storage.onChanged.addListener((changes, areaName) => {
  if (areaName === 'local' && changes.key) {
    console.log('Key changed:', changes.key.newValue);
  }
});

// Sync storage (synced across devices)
chrome.storage.sync.set({ settings: { theme: 'dark' } });
```

### 3. Dynamic Script Injection (MV3)

```javascript
// Inject script into active tab
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  chrome.scripting.executeScript({
    target: { tabId: tabs[0].id },
    func: () => {
      // Code to execute in page context
      document.body.style.backgroundColor = 'red';
    }
  });
});

// Inject CSS
chrome.scripting.insertCSS({
  target: { tabId: tabId },
  files: ['styles.css']
});
```

### 4. Tab Management

```javascript
// Create new tab
chrome.tabs.create({ url: 'https://example.com' }, (tab) => {
  console.log('Created tab:', tab.id);
});

// Update current tab
chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
  chrome.tabs.update(tabs[0].id, { url: 'https://example.com' });
});

// Close tab
chrome.tabs.remove(tabId);

// Listen for tab updates
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete') {
    console.log('Tab loaded:', tab.url);
  }
});
```

### 5. Context Menus

```javascript
// Create context menu item
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: 'myMenuItem',
    title: 'Search "%s"',
    contexts: ['selection']
  });
});

// Handle context menu clicks
chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === 'myMenuItem') {
    const searchTerm = info.selectionText;
    // Handle the selection
  }
});
```

## Best Practices

### Service Worker Management
- Service workers are ephemeral and can be terminated at any time
- Use chrome.alarms for scheduled tasks (setInterval unreliable)
- Store persistent state in chrome.storage
- Handle service worker wake-ups properly
- Avoid long-running operations

### Content Script Best Practices
- Minimize content script size for performance
- Use message passing for heavy operations
- Avoid polluting page global scope
- Use unique CSS class prefixes
- Clean up on page unload

### Permissions
- Request minimum necessary permissions
- Use optional_permissions for features users enable
- Avoid broad host_permissions when possible
- Justify all permissions in store listing
- Use activeTab permission for user-initiated actions

### Performance
- Lazy load resources when possible
- Minimize bundle size
- Use event pages pattern (MV3 service workers)
- Cache frequently accessed data
- Optimize content script injection

### Security
- Implement strict Content Security Policy
- Validate all user inputs
- Use HTTPS for all external requests
- Sanitize data from web pages
- Secure message passing with origin checks

## Debugging

### Chrome Extension DevTools

```javascript
// Console logging in different contexts
console.log('Service Worker'); // In background.js
console.log('Content Script'); // In content.js
console.log('Popup'); // In popup.js

// Inspect service worker
// chrome://extensions/ → Details → Service worker: "Inspect views"

// Inspect popup
// Right-click extension icon → Inspect popup

// View extension logs
// chrome://extensions/ → Errors button
```

### Common Issues

**Service worker not responding:**
- Check if service worker is active in chrome://extensions/
- Ensure message listeners return true for async responses
- Check for errors in service worker console

**Content script not injecting:**
- Verify matches pattern in manifest.json
- Check host_permissions are granted
- Ensure run_at timing is appropriate
- Check for CSP violations

**Storage not persisting:**
- Verify storage permissions in manifest
- Check storage quota limits
- Use chrome.storage, not localStorage (unreliable)

## Chrome Web Store Submission

### Requirements

1. **Manifest**: Valid Manifest V3 with complete metadata
2. **Icons**: 128x128px (required), 48x48px, 16x16px
3. **Screenshots**: 1280x800 or 640x400 (at least 1 required)
4. **Privacy Policy**: Required if handling user data
5. **Description**: Clear, accurate, under 132 characters (short)
6. **Permissions Justification**: Explain all permissions

### Common Rejection Reasons

- Overly broad permissions requests
- Missing or inadequate privacy policy
- Misleading descriptions or screenshots
- Keyword stuffing in description
- Functionality not working as described
- Security vulnerabilities
- Violations of Chrome Web Store policies

## Migration from Manifest V2 to V3

Key changes:
- Background pages → Service workers
- executeScript() → chrome.scripting.executeScript()
- webRequest → declarativeNetRequest
- Permissions split into permissions and host_permissions
- CSP moved to manifest key
- Action API unifies browser_action and page_action

## Troubleshooting

### Extension Not Loading
1. Check manifest.json syntax (use JSON validator)
2. Verify all file paths are correct
3. Check for permission conflicts
4. Look for errors in chrome://extensions/

### Messages Not Sending
1. Ensure runtime.onMessage listener exists
2. Return true from listener for async responses
3. Check sender.id matches extension ID
4. Verify message format is JSON-serializable

### Storage Issues
1. Check storage quota (chrome.storage.local: ~5MB)
2. Use chrome.storage, not localStorage
3. Verify storage permissions in manifest
4. Check for JSON serialization errors

## Resources

- Architecture Guide: `references/extension-architecture.md`
- Chrome APIs: `references/chrome-apis.md`
- Security Practices: `references/security-practices.md`
- Publishing Guide: `references/publishing-guide.md`
