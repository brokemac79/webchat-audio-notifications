# 🔔 Webchat Audio Notifications

Browser audio notifications for Moltbot/Clawdbot webchat. Get notified when new messages arrive - but only when the tab is in the background.

## ✨ Features

- 🔔 **Smart notifications** - Only plays sound when tab is hidden
- 🎚️ **Volume control** - Adjustable notification volume (0-100%)
- 🔕 **Easy toggle** - Enable/disable with one click
- 🎵 **Multiple sounds** - Choose between subtle notification or louder alert sound
- 💾 **Persistent preferences** - Settings saved in localStorage
- 📱 **Mobile-friendly** - Graceful handling of mobile restrictions
- 🚫 **Autoplay handling** - Respects browser autoplay policies
- ⏱️ **Cooldown** - Prevents notification spam (3s between alerts)
- 🐞 **Debug mode** - Optional logging for troubleshooting

## 🎯 Quick Start

### 1. Test the POC

Open `examples/test.html` in your browser to try it out:

```bash
cd webchat-audio-notifications/examples
python3 -m http.server 8080
# Open http://localhost:8080/test.html
```

**Test steps:**
1. Click "Enable Notifications" if prompted
2. Switch to another tab
3. Click "Trigger Notification" (or have someone trigger it)
4. You should hear a sound! 🔊

### 2. Basic Integration

```html
<!-- Load Howler.js -->
<script src="path/to/howler.min.js"></script>

<!-- Load WebchatNotifications -->
<script src="path/to/notification.js"></script>

<script>
  // Initialize
  const notifier = new WebchatNotifications({
    soundPath: './sounds/notification',
    defaultVolume: 0.7
  });
  
  await notifier.init();
  
  // Trigger notification when new message arrives
  socket.on('message', (msg) => {
    notifier.notify();
  });
</script>
```

## 📚 API Documentation

### Constructor

```javascript
const notifier = new WebchatNotifications(options);
```

**Options:**
```javascript
{
  soundPath: './sounds/notification',  // Path to sound files directory
  soundName: 'notification',           // Sound file name ('notification' or 'alert')
  defaultVolume: 0.7,                  // Volume level (0.0 to 1.0)
  cooldownMs: 3000,                    // Min time between notifications (ms)
  enableButton: true,                  // Show enable prompt if autoplay blocked
  debug: false                         // Enable console logging
}
```

### Methods

#### `init()`
Initialize the notification system. Must be called after Howler.js loads.

```javascript
await notifier.init();
```

#### `notify(eventType?)`
Trigger a notification (only plays if tab is hidden).

```javascript
notifier.notify();           // Default notification
notifier.notify('message');  // Message notification (future: different sounds)
```

#### `test()`
Play notification sound immediately (ignores tab state, useful for testing).

```javascript
notifier.test();
```

#### `setEnabled(enabled)`
Enable or disable notifications.

```javascript
notifier.setEnabled(true);   // Enable
notifier.setEnabled(false);  // Disable
```

#### `setVolume(volume)`
Set notification volume (0.0 to 1.0).

```javascript
notifier.setVolume(0.5);  // 50% volume
notifier.setVolume(1.0);  // 100% volume
```

#### `setSound(soundName)`
Change notification sound.

```javascript
notifier.setSound('notification');  // Subtle chime
notifier.setSound('alert');         // Louder, more prominent
```

#### `getSettings()`
Get current settings.

```javascript
const settings = notifier.getSettings();
// Returns: { enabled, volume, soundName, isMobile, initialized }
```

## 🌐 Browser Compatibility

| Browser | Version | Support | Notes |
|---------|---------|---------|-------|
| Chrome | 92+ | ✅ Full | Strictest autoplay policy |
| Firefox | 90+ | ✅ Full | Slightly more permissive |
| Safari | 15+ | ✅ Full | Requires WebKit prefixes (handled) |
| Edge | 92+ | ✅ Full | Chromium-based |
| Mobile Chrome | Latest | ⚠️ Limited | Requires user gesture per play |
| Mobile Safari | Latest | ⚠️ Limited | iOS restrictions apply |

**Overall compatibility:** 92% of users (based on Web Audio API support)

## ⚙️ Configuration Examples

### Simple Setup

```javascript
const notifier = new WebchatNotifications();
await notifier.init();
notifier.notify();  // That's it!
```

### Advanced Setup

```javascript
const notifier = new WebchatNotifications({
  soundPath: '/assets/sounds/notification',
  defaultVolume: 0.8,
  cooldownMs: 5000,  // 5 second cooldown
  debug: true        // Enable logging
});

await notifier.init();

// Hook into your message system
chatClient.on('newMessage', () => notifier.notify());
chatClient.on('mention', () => notifier.notify('mention'));  // Future: different sound
```

### With User Controls

```html
<!-- Volume slider -->
<input type="range" min="0" max="100" value="70" 
       onchange="notifier.setVolume(this.value / 100)">

<!-- Sound selector -->
<select onchange="notifier.setSound(this.value)">
  <option value="notification">🔔 Notification</option>
  <option value="alert">🚨 Alert</option>
</select>

<!-- Enable/disable toggle -->
<button onclick="notifier.setEnabled(true)">Enable 🔔</button>
<button onclick="notifier.setEnabled(false)">Disable 🔕</button>

<!-- Test button -->
<button onclick="notifier.test()">Test Sound 🔊</button>
```

## 🚨 Troubleshooting

### No sound playing?

**1. Check browser autoplay policy:**
- Click anywhere on the page first (browser may require user interaction)
- Look for the enable notification prompt
- Check browser console for errors

**2. Verify tab is hidden:**
- Notifications only play when tab is in background
- Use `notifier.test()` to test regardless of tab state

**3. Check volume:**
```javascript
console.log(notifier.getSettings().volume);  // Should be > 0
notifier.setVolume(1.0);  // Try max volume
```

**4. Verify files are accessible:**
- Open browser console
- Check Network tab for 404 errors on sound files
- Ensure sound files are in the correct path

### Sound plays when tab is active?

This shouldn't happen - it's a bug! The `document.hidden` check should prevent this.

**Debug steps:**
1. Enable debug mode: `new WebchatNotifications({ debug: true })`
2. Check console for "Tab is visible, skipping notification" message
3. If not appearing, there may be a Page Visibility API issue

### Mobile not working?

iOS Safari and mobile browsers have strict audio restrictions:
- Requires user gesture for EACH audio play (not just once)
- Background tab audio may be blocked entirely
- Consider using visual notifications (flashing favicon) on mobile

**Detect mobile:**
```javascript
const settings = notifier.getSettings();
if (settings.isMobile) {
  console.log('Mobile detected - audio may be limited');
}
```

## 📦 File Structure

```
webchat-audio-notifications/
├── client/
│   ├── notification.js       # Main notification class
│   ├── howler.min.js         # Howler.js library (36KB)
│   └── sounds/
│       ├── notification.mp3  # Default notification sound
│       ├── alert.mp3         # Alternative sound
│       └── SOUNDS.md         # Sound attribution
├── examples/
│   └── test.html            # Standalone test page
├── docs/
│   └── integration.md       # Integration guide
├── README.md                # This file
└── SKILL.md                 # ClawdHub metadata
```

## 🔐 Privacy & Security

- **No external requests** - All assets loaded locally
- **localStorage only** - Preferences stored in user's browser
- **No tracking** - Zero analytics or telemetry
- **No permissions required** - Works with standard Web Audio API

## 📄 License

MIT License - see LICENSE file for details.

## 🙏 Credits

- **Audio library:** [Howler.js](https://howlerjs.com/) by James Simpson (MIT License)
- **Sound files:** [Mixkit.co](https://mixkit.co/) (Royalty-free, commercial use allowed)
- **Created for:** [Moltbot/Clawdbot](https://github.com/moltbot/moltbot) community

## 🤝 Contributing

Found a bug? Have a feature request? 

1. Test with `examples/test.html` first
2. Enable debug mode to see console logs
3. Open an issue with browser version and console output

## 🚀 Next Steps

- [ ] WebM sound format support (smaller files)
- [ ] Per-event sound customization (different sounds for mentions, DMs, etc.)
- [ ] Visual notification fallback (flashing favicon)
- [ ] System notifications API integration
- [ ] Settings UI component
- [ ] Do Not Disturb mode (time-based)

---

**Status:** ✅ POC v1.0.0 - Ready for testing  
**Last updated:** 2026-01-28  
**Maintained by:** @brokemac79
