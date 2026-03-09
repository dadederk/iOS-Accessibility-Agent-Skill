# Full Keyboard Access

Full Keyboard Access enables users to navigate and interact with iOS using only a hardware keyboard.

## How It Works

Users navigate through focusable elements with Tab and activate with Space. A visible focus indicator shows the current element.

Full Keyboard Access is especially important on iPadOS with hardware keyboards.

## Enable Full Keyboard Access

**Settings > Accessibility > Keyboards > Full Keyboard Access**

## Key Commands

| Key | Action |
|-----|--------|
| Tab | Move to next element |
| Shift + Tab | Move to previous element |
| Space | Activate focused element |
| Escape | Dismiss or go back |
| Tab + Z | Show custom actions |
| Arrow keys | Navigate within controls |

## Development Impact

Elements that work with VoiceOver typically work with Full Keyboard Access. Focus on:
- Logical focus order
- Visible focus indicator
- Keyboard shortcuts for key actions

## Focus Order

Focus follows the accessibility order. Use the same techniques as VoiceOver:

**UIKit**:
```swift
view.accessibilityElements = [headerLabel, searchField, listView]
```

**SwiftUI**:
```swift
VStack {
    Text("First").accessibilitySortPriority(2)
    Text("Second").accessibilitySortPriority(1)
}
```

## Keyboard Shortcuts

Provide shortcuts for frequently used actions.

### UIKit

```swift
override var keyCommands: [UIKeyCommand]? {
    [
        UIKeyCommand(
            title: "Refresh",
            action: #selector(refresh),
            input: "r",
            modifierFlags: .command
        ),
        UIKeyCommand(
            title: "Search",
            action: #selector(search),
            input: "f",
            modifierFlags: .command
        )
    ]
}
```

### SwiftUI

```swift
Button("Refresh", action: refresh)
    .keyboardShortcut("r", modifiers: .command)
```

Shortcuts appear when the user holds the Command key.

## Input Labels

Help users find elements by alternate names:

**UIKit**:
```swift
button.accessibilityUserInputLabels = ["Settings", "Preferences", "Config"]
```

**SwiftUI**:
```swift
.accessibilityInputLabels(["Settings", "Preferences", "Config"])
```

This helps keyboard users who search by name.

## Custom Actions

Custom actions are accessible via Tab + Z:

```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Delete", actionHandler: { _ in
        self.delete()
        return true
    })
]
```

## Grouping

Group related elements to reduce Tab stops:

**UIKit**:
```swift
containerView.isAccessibilityElement = true
containerView.accessibilityLabel = "\(title), \(subtitle)"
```

**SwiftUI**:
```swift
.accessibilityElement(children: .combine)
```

## Testing

### Simulator

Enable Full Keyboard Access in the simulator's Settings app. Use your Mac keyboard to navigate.

### Device

Connect a hardware keyboard (Bluetooth or Smart Connector).

### What to verify

1. All interactive elements are focusable
2. Focus order is logical
3. Focus indicator is visible
4. Space activates the focused element
5. Escape dismisses modals
6. Keyboard shortcuts work

## Common Issues

| Problem | Solution |
|---------|----------|
| Element not focusable | Ensure `isAccessibilityElement = true` |
| Focus order confusing | Set `accessibilityElements` order |
| Focus indicator hidden | Avoid clipping or overlays |
| Action requires touch | Add keyboard shortcut or custom action |

## Checklist

- [ ] All interactive elements focusable
- [ ] Focus order follows task flow
- [ ] Keyboard shortcuts for main actions
- [ ] Custom actions exposed via Tab + Z
- [ ] Tested in simulator and on device

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://accessibilityupto11.com/blog/
