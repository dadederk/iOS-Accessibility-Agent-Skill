# Switch Control

Switch Control enables users with motor impairments to navigate iOS using external switches, head movements, or other adaptive hardware.

## How It Works

Switch Control scans through elements one at a time. When the desired element is highlighted, the user activates a switch to select it.

### Scanning modes

| Mode | Behavior |
|------|----------|
| Auto Scan | Cursor moves automatically through elements |
| Manual Scan | User triggers each move |
| Step Scan | Multi-switch navigation (next/previous/select) |

## Enable Switch Control

**Settings > Accessibility > Switch Control**

## Impact on Development

Good VoiceOver support generally means good Switch Control support. The same accessibility properties apply:
- Labels
- Traits
- Grouping
- Custom actions

## Reduce Scanning Steps

### Group related elements

Fewer elements means fewer scan steps to reach the target.

**UIKit**:
```swift
containerView.isAccessibilityElement = true
containerView.accessibilityLabel = "\(title), \(subtitle)"
```

**SwiftUI**:
```swift
HStack {
    Image(systemName: "star.fill")
    Text("Favorite")
}
.accessibilityElement(children: .combine)
```

### Semantic grouping

Use container types to organize controls:

```swift
toolbar.accessibilityContainerType = .semanticGroup
```

Switch Control recognizes groups and offers "Scan this group" actions.

## Custom Actions

Switch Control surfaces custom actions through its menu system.

```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(
        name: "Delete",
        image: UIImage(systemName: "trash"),
        actionHandler: { _ in
            self.delete()
            return true
        }
    ),
    UIAccessibilityCustomAction(
        name: "Share",
        image: UIImage(systemName: "square.and.arrow.up"),
        actionHandler: { _ in
            self.share()
            return true
        }
    )
]
```

Images appear in the Switch Control menu (iOS 14+) with [`UIAccessibilityCustomAction.init(name:image:actionHandler:)`](https://developer.apple.com/documentation/uikit/uiaccessibilitycustomaction/init(name:image:actionhandler:)).

## Traits

Ensure traits are correct:
- `.button` for tappable elements
- `.selected` for current selection
- `.notEnabled` for disabled controls
- `.adjustable` for steppers and sliders

Missing traits confuse navigation.

## Testing

The easiest low-friction option is to use a Bluetooth keyboard:

1. Go to **Settings > Accessibility > Switch Control**
2. Add a switch, choose **Keyboard** as the source
3. Map the **Space bar** to "Select Item" — one key scans; pressing selects
4. Or configure two keys: one for "Move to Next Item", another for "Select Item"

If you don't have a Bluetooth keyboard, you can test using head movements:

- **Settings > Accessibility > Switch Control > Switches**: add a switch using **Left Head Movement** (move to next) and **Right Head Movement** (select)
- Note: enabling this disables other screen touches, so keep the Accessibility Shortcut handy to exit

Once enabled:
1. Let Switch Control scan through your app
2. Verify all elements are reachable and in a logical order
3. Confirm custom actions appear in the menu (reduces step count)
4. Check that grouping reduces unnecessary scan steps

### What to verify

- All interactive elements are reachable
- Grouping reduces unnecessary scan steps
- Custom actions are discoverable (complex actions appear in the Switch Control menu rather than requiring manual navigation)
- Traits match element behavior

## visionOS

Switch Control is supported in visionOS. Building accessible spatial experiences benefits from the same patterns:
- Clear labels
- Logical grouping
- Exposed actions

## Common Issues

| Problem | Solution |
|---------|----------|
| Too many scan steps | Group related elements |
| Action hidden behind gesture | Add custom action |
| Element unreachable | Set `isAccessibilityElement = true` |
| Trait missing | Add appropriate traits |

## Checklist

- [ ] Elements grouped to reduce scan steps
- [ ] Custom actions provided for secondary features
- [ ] Traits match behavior
- [ ] Tested with Switch Control enabled

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://accessibilityupto11.com/blog/
