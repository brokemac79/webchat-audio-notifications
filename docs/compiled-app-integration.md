# Integration with Compiled Web Apps

This guide covers integrating audio notifications into compiled/bundled web applications.

## Requirements

1. Copy files to your assets directory
2. Add scripts to HTML
3. Hook into message detection
4. Handle autoplay unlock

## Example Integration

### 1. Copy Assets

```bash
cp client/howler.min.js /path/to/your-app/assets/
cp client/notification.js /path/to/your-app/assets/
cp -r client/sounds /path/to/your-app/assets/
```

### 2. Modify HTML

```html
<!doctype html>
<html>
  <head>
    <title>Your App</title>
    <!-- Your app scripts -->
  </head>
  <body>
    <your-app></your-app>
    
    <!-- Webchat Audio Notifications -->
    <script src="./assets/howler.min.js"></script>
    <script src="./assets/notification.js"></script>
    <script>
      let notifier = null;
      let lastMessageCount = 0;
      
      window.addEventListener('DOMContentLoaded', async () => {
        // Initialize
        notifier = new WebchatNotifications({
          soundPath: './assets/sounds',
          soundName: 'level3',
          defaultVolume: 0.7,
          debug: true
        });
        
        await notifier.init();
        console.log('🔔 Notifications ready');
        
        // Wait for app to render
        setTimeout(() => {
          lastMessageCount = countMessages();
          console.log('✅ Monitoring messages (initial count:', lastMessageCount + ')');
          
          // Poll for new messages (200-500ms recommended)
          setInterval(checkForNewMessages, 200);
        }, 2000);
      });
      
      function countMessages() {
        // Adjust selector to match your app's DOM structure
        const messages = document.querySelectorAll('[role="article"], [class*="message"]');
        return messages.length;
      }
      
      function checkForNewMessages() {
        const currentCount = countMessages();
        if (currentCount > lastMessageCount) {
          console.log('🔔 New message detected!');
          if (notifier) {
            notifier.notify(); // Only plays if tab is hidden
          }
        }
        lastMessageCount = currentCount;
      }
      
      // Expose test function for first-time unlock
      window.testNotification = () => {
        if (notifier) notifier.test();
      };
    </script>
  </body>
</html>
```

### 3. Finding the Right DOM Selector

**Test in browser console:**

```javascript
// Try different selectors to find what matches your messages:
document.querySelectorAll('[role="article"]').length
document.querySelectorAll('[class*="message"]').length
document.querySelectorAll('[data-message]').length
document.querySelectorAll('.chat-message').length

// Use the selector that returns the correct message count
```

### 4. Shadow DOM Support

If your app uses Web Components with Shadow DOM:

```javascript
function countMessages() {
  const app = document.querySelector('your-app');
  
  // Check if shadow DOM exists
  if (app?.shadowRoot) {
    const messages = app.shadowRoot.querySelectorAll('[your-selector]');
    return messages.length;
  }
  
  // Fallback to light DOM
  const messages = app?.querySelectorAll('[your-selector]');
  return messages?.length || 0;
}
```

**Wait for Shadow DOM to initialize:**

```javascript
async function waitForShadowDOM(maxAttempts = 10) {
  for (let i = 0; i < maxAttempts; i++) {
    const app = document.querySelector('your-app');
    if (app?.shadowRoot) {
      return true;
    }
    await new Promise(resolve => setTimeout(resolve, 500));
  }
  return false;
}

// Use it:
const ready = await waitForShadowDOM();
if (ready) {
  // Start message detection
}
```

## First-Time Audio Unlock

Browsers block audio autoplay until user interaction. Users must:

**Option 1: Console command (simplest)**
```javascript
// Run once in browser console (F12):
window.testNotification()
```

**Option 2: Visible button**
```html
<button onclick="window.testNotification()" style="position:fixed;top:10px;right:10px;z-index:9999">
  🔔 Enable Notifications
</button>
```

**Option 3: First-message prompt**
```javascript
let audioUnlocked = false;

function checkForNewMessages() {
  const currentCount = countMessages();
  if (currentCount > lastMessageCount) {
    if (!audioUnlocked) {
      // Show unlock prompt on first message
      if (confirm('Enable audio notifications?')) {
        notifier.test(); // Unlocks audio
        audioUnlocked = true;
      }
    } else {
      notifier.notify();
    }
  }
  lastMessageCount = currentCount;
}
```

## Debugging

### Check if notifications are initializing:

```javascript
// Should see in console:
// "🔔 Notifications ready"
// "✅ Monitoring messages (initial count: X)"
```

### Test sound system:

```javascript
window.testNotification()  // Should play immediately
```

### Check message detection:

```javascript
// Should see when messages arrive:
// "🔔 New message detected!"

// Verify selector works:
function countMessages() {
  const messages = document.querySelectorAll('[your-selector]');
  console.log('Messages found:', messages.length);
  return messages.length;
}
countMessages()
```

### Check tab visibility:

```javascript
// In console:
console.log('Tab hidden:', document.hidden);

// Or watch for changes:
document.addEventListener('visibilitychange', () => {
  console.log('Tab visibility changed:', document.hidden ? 'HIDDEN' : 'VISIBLE');
});
```

## Common Issues

### Sound doesn't play on new messages

**Check:**
1. Console shows "🔔 New message detected!" when messages arrive
2. Tab is actually hidden (minimized or switched away)
3. Audio was unlocked (ran `window.testNotification()` once)

**Fix:**
- Verify your message selector is correct
- Check polling interval (too slow = missed messages)
- Ensure tab is truly hidden (not just on another monitor)

### "Autoplay blocked" error

**Cause:** Browser security policy

**Fix:** User must interact with page once:
```javascript
window.testNotification()
```

### Messages not detected

**Check selector:**
```javascript
// Count should match visible messages:
document.querySelectorAll('[your-selector]').length
```

**Check timing:**
- Wait 2-3 seconds after page load before starting detection
- Use longer timeout if your app is slow to render

### Polling performance concerns

**Recommendations:**
- 200ms: Responsive, minimal performance impact
- 500ms: Good balance (default in most examples)
- 1000ms: Conservative, may miss rapid messages

**Don't go below 100ms** - unnecessary CPU usage

## Production Checklist

- [ ] MP3 files copied to assets directory
- [ ] Scripts added to HTML
- [ ] Message selector verified (correct count)
- [ ] Polling interval appropriate for your app
- [ ] Audio unlock documented for users
- [ ] Tested in multiple browsers
- [ ] Tested with tab hidden (actual use case)
- [ ] Console logs clean (no errors)

## Example: Real-World Integration

Here's a complete example based on Clawdbot's Control UI:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Clawdbot Control</title>
    <link rel="stylesheet" href="./assets/app.css">
    <script type="module" src="./assets/app.js"></script>
  </head>
  <body>
    <clawdbot-app></clawdbot-app>
    
    <!-- Audio Notifications -->
    <script src="./assets/howler.min.js"></script>
    <script src="./assets/notification.js"></script>
    <script>
      let notifier = null;
      let lastMessageCount = 0;
      
      window.addEventListener('DOMContentLoaded', async () => {
        notifier = new WebchatNotifications({
          soundPath: './assets/sounds',
          soundName: 'level3',
          defaultVolume: 0.7
        });
        
        await notifier.init();
        
        setTimeout(() => {
          lastMessageCount = countMessages();
          setInterval(checkForNewMessages, 200);
        }, 2000);
      });
      
      function countMessages() {
        const app = document.querySelector('clawdbot-app');
        if (!app) return 0;
        return app.querySelectorAll('[role="article"]').length;
      }
      
      function checkForNewMessages() {
        const current = countMessages();
        if (current > lastMessageCount && notifier) {
          notifier.notify();
        }
        lastMessageCount = current;
      }
      
      window.testNotification = () => notifier?.test();
    </script>
  </body>
</html>
```

## Support

For issues specific to compiled app integration:
1. Check console for error messages
2. Verify file paths are correct
3. Test `window.testNotification()` works
4. Confirm message selector matches your DOM

For general issues, see main [README.md](../README.md) troubleshooting section.
