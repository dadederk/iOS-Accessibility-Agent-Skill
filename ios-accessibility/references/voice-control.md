# Voice Control

Voice Control allows users to navigate and interact using only their voice.

## How It Works

Voice Control recognizes accessibility labels and lets users speak them to activate controls.

```
User: "Tap Settings"
→ Activates button with accessibilityLabel "Settings"
```

Voice Control can work offline and relies heavily on the labels you provide. Clear, concise labels lead to faster, more reliable activation.

## Enable Voice Control

**Settings > Accessibility > Voice Control**

Or say "Turn on Voice Control" to Siri.

## Key Commands

| Command | Action |
|---------|--------|
| "Show names" | Overlay labels on all elements |
| "Show numbers" | Overlay numbers on all elements |
| "Tap [label]" | Activate element by name |
| "Tap [number]" | Activate numbered element |
| "Show grid" | Display a grid for precise tapping |
| "Show actions" | Show custom actions for focused element |
| "Scroll down/up" | Scroll the screen |
| "Go back" | Navigate back |

## Label Best Practices

### Match visible text

If a button shows "Submit", the accessibility label should be "Submit" — not "Send" or "Submit button".

```swift
// Button shows "Settings"
settingsButton.accessibilityLabel = "Settings" // ✓ Matches
```

### Avoid duplicates

Multiple elements with the same label cause ambiguity. Voice Control falls back to showing numbers.

### Use input labels for alternatives

When the visible text doesn't match what users might say:

**UIKit**:
```swift
gearButton.accessibilityLabel = "Settings"
gearButton.accessibilityUserInputLabels = ["Settings", "Preferences", "Options", "Gear", "Cog"]
```

**SwiftUI**:
```swift
Button(action: openSettings) {
    Image(systemName: "gear")
}
.accessibilityLabel("Settings")
.accessibilityInputLabels(["Settings", "Preferences", "Options", "Gear", "Cog"])
```

Users can say any of these alternatives.

## Testing with Voice Control

### "Show names"

Reveals all accessibility labels overlaid on the screen. Quickly identifies:
- Missing labels (no overlay appears)
- Duplicate labels (same text on multiple elements)
- Confusing labels (text that doesn't match the visual)

Use this during development to validate the labels users will actually speak.

### "Show actions"

Focus on an element and say "Show actions" to see custom accessibility actions.

## Custom Actions

Voice Control exposes custom actions. Users can say "Show actions for [element]" then activate by name.

```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Delete", actionHandler: { _ in
        self.delete()
        return true
    })
]
```

User: "Show actions for Message"
→ "Delete" appears
User: "Tap Delete"

## Accessibility Values

Values improve Voice Control usability for stateful controls:

```swift
slider.accessibilityLabel = "Volume"
slider.accessibilityValue = "50 percent"
```

User: "What is Volume?"
→ Voice Control announces "50 percent"

## Common Issues

| Problem | Solution |
|---------|----------|
| Element has no label | Add `accessibilityLabel` |
| Label doesn't match visible text | Update label or use input labels |
| Multiple elements with same label | Make labels unique or use context |
| Icon-only button | Add descriptive label |
| Custom gesture required | Provide accessible alternative |

## Checklist

- [ ] Labels match visible text when possible
- [ ] No duplicate labels for different controls
- [ ] Icon-only buttons have descriptive labels
- [ ] Input labels provided for non-obvious names
- [ ] Custom actions exposed for secondary features
- [ ] Tested with "Show names" command

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://accessibilityupto11.com/blog/
