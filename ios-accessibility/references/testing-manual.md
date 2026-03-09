# Manual Testing

Testing iOS accessibility with assistive technologies and device settings.

## Why Manual Testing

Automated tools catch basic issues but cannot evaluate:
- Whether labels make sense in context
- If navigation order follows a logical flow
- Whether the experience is actually usable
- How complex interactions feel with assistive tech

Always combine automated checks with hands-on testing.

## Quick Setup

### Accessibility Shortcut

Triple-click the side (or home) button to toggle an assistive technology.

Configure in **Settings > Accessibility > Accessibility Shortcut**.

Recommended: Add VoiceOver, Voice Control, and Full Keyboard Access.

Tip: choose a small set of shortcuts so you can toggle quickly without cycling through too many options.

### Control Center

Add **Accessibility Shortcuts** to Control Center for faster toggling.

**Settings > Control Center > Customize Controls > Accessibility Shortcuts**

### Siri

- "Turn on VoiceOver"
- "Turn off VoiceOver"
- "Turn on Voice Control"

## VoiceOver Testing

### Core gestures

| Gesture | Action |
|---------|--------|
| Tap | Select element |
| Double-tap | Activate |
| Swipe right | Next element |
| Swipe left | Previous element |
| Two-finger rotate | Change rotor mode |
| Swipe up/down | Adjust (if adjustable) or navigate by rotor |
| Two-finger double-tap | Magic Tap |
| Two-finger scrub (Z shape) | Escape |
| Three-finger swipe | Scroll |

### Screen Curtain

Triple-tap with three fingers to black out the screen while VoiceOver remains active. Forces non-visual testing.

Use Screen Curtain to validate that core flows work without visual cues. If it feels hard with Screen Curtain, the UI likely needs simplification or better structure.

### Caption Panel

Enable in **Settings > Accessibility > VoiceOver > Caption Panel** to see VoiceOver output at the bottom of the screen.

### What to verify

1. Every interactive element is reachable by swiping
2. Labels describe the element clearly
3. Traits match the role (button, header, etc.)
4. Navigation order follows the task flow
5. Focus moves to new content after state changes
6. Errors are announced
7. Custom gestures have accessible alternatives

### Advanced gestures

| Gesture | Action |
|---------|--------|
| Single tap with two fingers | Pause/resume speech |
| Double-tap with three fingers | Mute/unmute VoiceOver speech |
| Triple-tap with one finger | Long press |
| Four-finger tap at top | First element |
| Four-finger tap at bottom | Last element |

## Voice Control Testing

### Enable

**Settings > Accessibility > Voice Control**

Or say "Turn on Voice Control" to Siri.

### Testing

1. Say "Show names" to overlay accessibility labels
2. Try speaking button labels to activate them
3. Say "Show grid" for precise tapping
4. Say "Show actions" on an element to see custom actions

### What to verify

1. Labels match visible text (or are intuitive)
2. No duplicate labels for different controls
3. All interactive elements can be activated by voice
4. Custom actions are discoverable

## Full Keyboard Access Testing

### Enable

**Settings > Accessibility > Keyboards > Full Keyboard Access**

### Navigation

| Key | Action |
|-----|--------|
| Tab | Next element |
| Shift + Tab | Previous element |
| Space | Activate |
| Escape | Dismiss |
| Tab + Z | Show actions |

### What to verify

1. All interactive elements are focusable
2. Focus order is logical
3. Focus indicator is visible
4. Actions can be triggered from keyboard
5. Keyboard shortcuts are documented and discoverable

### Simulator testing

Enable Full Keyboard Access in the simulator. Use your Mac keyboard to navigate.

## Switch Control Testing

### Enable

**Settings > Accessibility > Switch Control**

### What to verify

1. All elements are reachable via scanning
2. Grouped elements reduce scan steps
3. Custom actions appear in the menu

## Zoom Testing

### Enable

**Settings > Accessibility > Zoom**

### Gestures

| Gesture | Action |
|---------|--------|
| Double-tap with three fingers | Toggle zoom |
| Double-tap with three fingers + drag | Adjust zoom level |
| Three-finger drag | Pan while zoomed |

### What to verify

1. Content remains usable when zoomed
2. Actions don't trigger outside the viewport
3. Ephemeral feedback (toasts) is visible

## Magnifier Testing

Magnifier uses the camera to zoom real-world content. If your app helps users capture or read text, consider:

1. Whether your UI works alongside Magnifier
2. Avoiding critical UI elements in areas often covered by the camera preview

## Guided Access Testing

Guided Access locks the device to a single app and can restrict touch areas. If your app is used in education, kiosks, or focused tasks:

1. Ensure the core flow works without system gestures
2. Avoid reliance on multitasking gestures for critical actions
3. If your UI depends on specific screen regions, verify they are not restricted in Guided Access settings
4. Consider time‑limit scenarios if the app is used for timed activities

## Dynamic Type Testing

### Simulator shortcut

`Option + Command + +` or `-` to increase/decrease text size.

### Environment Overrides

In Xcode's Debug Area toolbar, click the Environment Overrides button to change Dynamic Type without leaving the debugger.

Use this panel to simulate:
- Dark Mode
- Increase Contrast
- Reduce Motion
- Reduce Transparency
- Bold Text
- Button Shapes
- Grayscale
- Differentiate Without Color

### Accessibility Inspector

**Xcode > Open Developer Tool > Accessibility Inspector**

Use the Settings tab to change text size on a running simulator or device.

### SwiftUI variants preview

Preview all sizes at once in Xcode's canvas.

### What to verify

1. Text scales with system size
2. No truncation at large sizes (unless intentional)
3. Layout adapts for accessibility sizes
4. Multiline text is readable

### Double-length pseudolanguage

Edit scheme > Options > App Language > Double-Length Pseudolanguage

Stress-tests layout with longer strings.

## Reduce Motion Testing

### Enable

**Settings > Accessibility > Motion > Reduce Motion**

### What to verify

1. Parallax effects are disabled
2. Slide animations become fades
3. Auto-playing animations stop or slow down

## Increase Contrast Testing

### Enable

**Settings > Accessibility > Display & Text Size > Increase Contrast**

### What to verify

1. Text contrast improves
2. Asset variants (if provided) are used
3. No elements become unreadable

## Testing on Device vs Simulator

| Feature | Device | Simulator |
|---------|--------|-----------|
| VoiceOver | Full support | Limited (Mac VoiceOver) |
| Voice Control | Full support | Not supported |
| Full Keyboard Access | Full support | Full support |
| Switch Control | Full support | Limited |
| Dynamic Type | Full support | Full support |
| Haptics | Full support | Not available |

**Recommendation**: Test on device for VoiceOver and Voice Control. Simulator is fine for layout and Dynamic Type.

## Accessibility Inspector

### Launch

**Xcode > Open Developer Tool > Accessibility Inspector**

### Features

| Tab | Purpose |
|-----|---------|
| Inspection | View accessibility properties of any element |
| Audit | Run automated checks for common issues |
| Settings | Change Dynamic Type, Reduce Motion, etc. |

### Connect to device

Accessibility Inspector works with simulators and physical devices. Useful for inspecting other apps.

### Notifications log

**Window > Show Notifications** to see accessibility notifications (announcements, focus changes).

## Testing Checklist

### VoiceOver
- [ ] All elements reachable
- [ ] Labels are clear
- [ ] Traits are correct
- [ ] Order is logical
- [ ] Focus moves after state changes
- [ ] Errors are announced

### Dynamic Type
- [ ] Text scales
- [ ] No truncation
- [ ] Layout adapts for large sizes

### Other
- [ ] Reduce Motion honored
- [ ] Increase Contrast works
- [ ] Voice Control can activate all buttons
- [ ] Full Keyboard Access navigates everything

## Tools and Apps

- **Accessibility Inspector**: Built into Xcode
- **ScreenReader app** by @JanJaapdeGroot: Learn VoiceOver gestures
- **Voice Control "Show names"**: Overlay labels

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://github.com/Apress/developing-accessible-iOS-apps
