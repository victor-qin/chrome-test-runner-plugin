---
name: chrome-testing
description: >
  Knowledge base for QA testing web applications in Chrome using the Claude
  in Chrome browser tools. Auto-loaded when conversations involve testing,
  QA, stress testing, bug finding, or browser-based test execution.
---

# Chrome Testing Knowledge

## Available Telemetry via Claude in Chrome

These are the tools available when Claude Code is connected to Chrome (`claude --chrome`):

| Tool | What it captures | Key params |
|------|-----------------|------------|
| `read_network_requests` | All HTTP/fetch/XHR requests with method, URL, status, timing | `clear: true` to flush, `urlPattern` to filter |
| `read_console_messages` | console.log/warn/error/info with timestamps | `clear: true` to flush, `onlyErrors: true`, `pattern: "regex"` |
| `javascript_tool` | Execute arbitrary JS to inspect page state | tabId, text (JS code) |
| `computer: screenshot` | Current viewport capture | tabId |
| `gif_creator` | Record animated GIF of interactions | start/stop/export |
| `read_page` | Accessibility tree / DOM structure | `filter: "interactive"` for just buttons/inputs |

### Telemetry Capture Pattern

Always use this flush-capture cycle for clean per-step data:

```
1. read_network_requests(clear: true)  ← flush old data
2. read_console_messages(clear: true)  ← flush old data
3. [perform your action]
4. computer: wait 1-2 seconds
5. read_network_requests()             ← capture fresh data for this step
6. read_console_messages()             ← capture fresh data for this step
```

## Common Bug Patterns

### Race Conditions
- **Symptom**: Intermittent 500 errors, data corruption, out-of-order operations
- **Where**: Rapid-fire form submissions, concurrent API calls, optimistic UI updates
- **Test**: Send 3+ identical requests within 500ms, watch for failures
- **Signal in telemetry**: Same endpoint, mixed 200/500 status codes

### Stale State
- **Symptom**: UI shows outdated data after mutations
- **Where**: After create/update/delete, after navigation, after tab switching
- **Test**: Mutate data → navigate away → come back. Does it show new data?
- **Signal**: Missing re-fetch requests after mutation

### Memory / Performance Leaks
- **Symptom**: App gets slower over time
- **Where**: Components mounting/unmounting, event listeners, WebSocket handlers
- **Test**: Repeat an action 20+ times, monitor DOM node count and response times
- **JS check**: `document.querySelectorAll('*').length` — compare early vs late

### WebSocket Issues
- **Symptom**: Real-time features stop updating
- **Where**: After idle periods, after network disruption
- **Test**: Check console for WS close events, verify reconnection
- **JS check**: Look for WebSocket objects in performance entries

### Silent Failures
- **Symptom**: Action appears to work but data doesn't persist
- **Where**: Optimistic UI that doesn't verify server response
- **Test**: Perform action → refresh page → check if change persisted
- **Signal**: POST returns 200 but response body is empty or error

## Console Message Triage

### Always flag as bugs:
- `Uncaught TypeError` / `Uncaught ReferenceError`
- `Unhandled Promise rejection`
- React: `Cannot update a component while rendering a different component`
- React: `Each child in a list should have a unique "key" prop` (if data-dependent)
- `CORS policy` errors
- Any `Error:` in red

### Flag as warnings:
- `componentWillMount` / `componentWillReceiveProps` deprecation
- `findDOMNode is deprecated`
- React: `Can't perform a React state update on an unmounted component`
- Performance warnings about large payloads

### Usually ignorable:
- `Download the React DevTools`
- Favicon 404s (`/favicon.ico`)
- Source map warnings
- Third-party script errors (analytics, etc.)

## Network Health Quick Reference

| Metric | Good | Concerning | Bad |
|--------|------|------------|-----|
| Error rate | < 1% | 1-5% | > 5% |
| Avg response | < 200ms | 200ms-1s | > 1s |
| Payload per request | < 100KB | 100KB-1MB | > 1MB |
| Duplicate requests | 0 | 1-2 | 3+ identical |

## Useful javascript_tool Snippets

```javascript
// Performance navigation timing
JSON.stringify(performance.getEntriesByType('navigation')[0], null, 2)
```

```javascript
// Count DOM nodes (tracks memory pressure)
document.querySelectorAll('*').length
```

```javascript
// Find all event listeners (estimate via element count with handlers)
document.querySelectorAll('[onclick],[onchange],[onsubmit],[onkeydown],[onkeyup]').length
```

```javascript
// Check for leaked intervals/timeouts (inject early, read later)
window.__intervals = window.__intervals || 0;
const origSetInterval = window.setInterval;
window.setInterval = (...args) => { window.__intervals++; return origSetInterval(...args); };
window.__intervals
```

```javascript
// Get all failed resource loads
performance.getEntriesByType('resource')
  .filter(r => r.responseStatus >= 400 || r.responseStatus === 0)
  .map(r => ({ url: r.name, status: r.responseStatus, type: r.initiatorType }))
```
