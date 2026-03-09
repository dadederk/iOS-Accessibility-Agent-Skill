# Accessibility Playbook

Use this playbook for common mistakes, Accessibility Inspector warnings, core patterns, version-specific APIs, and verification checklists. Pair this with framework-specific references (VoiceOver, Dynamic Type, etc.) for deeper guidance.

## Contents

- [Common Mistakes Playbook](#common-mistakes-playbook)
- [Common Accessibility Inspector Warnings](#common-accessibility-inspector-warnings)
- [Core Patterns Reference](#core-patterns-reference)
- [Testing Workflow](#testing-workflow)
- [iOS Version-Specific APIs](#ios-version-specific-apis)
- [Common Scenarios → Quick Navigation](#common-scenarios--quick-navigation)
- [Best Practices Summary](#best-practices-summary)
- [Verification Checklist (After Making Changes)](#verification-checklist-after-making-changes)
- [Review Checklist (Quick Check)](#review-checklist-quick-check)
- [Sources](#sources)

## Common Mistakes Playbook

When you see these patterns (framed as user-experience issues), suggest the fix. Examples show both UIKit and SwiftUI where applicable.

### VoiceOver reads nothing or reads "button"
**Cause:** Element has no label or an empty label. Very common with icon-only buttons.
**Fix:** Add `accessibilityLabel` with a concise description.
```swift
// UIKit — Icon button without label: VoiceOver reads "button"
let closeButton = UIButton(type: .system)
closeButton.setImage(UIImage(systemName: "xmark"), for: .normal)
// Good:
closeButton.accessibilityLabel = "Close"

// SwiftUI — Bad: VoiceOver reads "button"
Button(action: close) { Image(systemName: "xmark") }

// SwiftUI — Good: VoiceOver reads "Close, button"
Button(action: close) { Image(systemName: "xmark") }
    .accessibilityLabel("Close")
```

### VoiceOver reads each element separately in a cell and navigation is tedious
**Cause:** Elements not grouped; user swipes through every label and button.
**Fix:** Group the cell as a single element with custom actions for buttons.
```swift
// UIKit
cell.isAccessibilityElement = true
cell.accessibilityLabel = "\(title), \(subtitle)"
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Add to cart") { _ in self.addToCart(); return true }
]

// SwiftUI — Single swipe; combine children so label is built from inner views' accessibility
HStack {
    AsyncImage(url: imageURL)
    VStack(alignment: .leading) { Text(title); Text(subtitle) }
}
.accessibilityElement(children: .combine)
// Inner views' labels/values contribute to the group announcement

// SwiftUI — Or single swipe with custom actions
NavigationLink { ... } label: { content }
    .accessibilityAction(named: "Add to cart") { addToCart() }
```

### VoiceOver reads a custom control as many separate elements or buttons
**Cause:** Each part (star, thumb, increment/decrement) is a separate focusable element.
**Fix:** **Prefer [`.accessibilityRepresentation`](https://developer.apple.com/documentation/swiftui/view/accessibilityrepresentation(representation:)) (iOS 16+) when a similar native control exists.** Otherwise group as one element: use **adjustable** (rating, stepper) when the control has increment/decrement semantics; use **custom actions** or a single **button** when it doesn't (e.g. a custom picker with discrete options).
```swift
// SwiftUI — Best: use representation if a native control fits (e.g. Stepper)
CustomRatingView(rating: $rating)
    .accessibilityRepresentation {
        Stepper("Rating", value: $rating, in: 1...5)
    }

// SwiftUI — If custom implementation required: single element, adjustable
customControl
    .accessibilityElement(children: .ignore)
    .accessibilityLabel("Rating")
    .accessibilityValue("\(rating) of 5")
    .accessibilityAdjustableAction { direction in ... }

// UIKit — Same idea: one element, custom actions or adjustable
view.isAccessibilityElement = true
view.accessibilityLabel = "Rating"
view.accessibilityValue = "\(rating) of 5"
view.accessibilityTraits = .adjustable
// Implement accessibilityIncrement() / accessibilityDecrement()
```

### Text doesn't scale with Dynamic Type
**Cause:** Fixed font size used.
**Fix:** Use text styles.
```swift
// UIKit
label.font = UIFont.preferredFont(forTextStyle: .body)
label.adjustsFontForContentSizeCategory = true

// SwiftUI
Text("Content")
    .font(.body)  // Scales automatically
```

### Layout breaks at large text sizes
**Cause:** Horizontal layout can't accommodate larger text.
**Fix:** Use adaptive layout based on `dynamicTypeSize` (or `preferredContentSizeCategory`) when you want deterministic behavior across repeated items. Use `ViewThatFits` when you want a local fallback based on actual fit in a specific part of the layout, or when the fallback is more complex than just flipping stack axis.
```swift
// SwiftUI — Adaptive stack: flip axis at accessibility sizes (iOS 16+ [AnyLayout](https://developer.apple.com/documentation/swiftui/anylayout))
@Environment(\.dynamicTypeSize) private var dynamicTypeSize

var body: some View {
    let layout = dynamicTypeSize.isAccessibilitySize ? AnyLayout(VStackLayout()) : AnyLayout(HStackLayout())
    layout { content }
}

// SwiftUI — Same idea, conditional stack (all iOS versions)
@Environment(\.dynamicTypeSize) var dynamicTypeSize
var body: some View {
    if dynamicTypeSize.isAccessibilitySize { VStack { content } } else { HStack { content } }
}

// SwiftUI — Fit-based fallback for a local layout block
ViewThatFits {
    HStack { content } // Preferred compact layout
    VStack { content } // Fallback when horizontal doesn't fit
}

// UIKit
if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
    stackView.axis = .vertical
} else {
    stackView.axis = .horizontal
}
```

Prefer deterministic rules for repeated list/grid items; otherwise different rows may resolve to different layouts depending on content length.

### Toast/snackbar disappears before VoiceOver reaches it
**Cause:** Ephemeral feedback with no announcement.
**Fix:** Post an announcement and consider persistent alternatives.
```swift
// UIKit
UIAccessibility.post(notification: .announcement, argument: message)

// SwiftUI
var announcement = AttributedString(message)
announcement.accessibilitySpeechAnnouncementPriority = .high
AccessibilityNotification.Announcement(announcement).post()
```

### Voice Control can't find or activate a button
**Cause:** Label differs from visible text, or Voice Control needs intuitive names.
**Fix:** Voice Control defaults to using the `accessibilityLabel` for recognition. Provide additional alternatives with `accessibilityInputLabels` when you need synonyms or shorter commands (e.g., "Remove", "Delete" for "Remove User").
```swift
// SwiftUI
Button("Remove User") { remove() }
    .accessibilityInputLabels(["Remove User", "Remove", "Delete"])

// UIKit
button.accessibilityLabel = "Remove User"
button.accessibilityUserInputLabels = ["Remove User", "Remove", "Delete"]
```

### Custom interactive controls using tap gestures not exposed to assistive tech as buttons
**Cause:** View uses `onTapGesture` (SwiftUI) or `UITapGestureRecognizer` (UIKit) but isn't exposed as a button to VoiceOver.
**Fix:** **Prefer a native `Button` (SwiftUI) or `UIButton` (UIKit)** for better accessibility out-of-the-box. If you must use a custom view with gestures, make it an accessibility element and add the button trait.
```swift
// SwiftUI — Best: use a native Button
Button { select() } label: { HStack { Text("Option") } }

// SwiftUI — If custom view required: add trait and label manually
HStack { Text("Option") }
    .onTapGesture { select() }
    .accessibilityAddTraits(.isButton)
    .accessibilityLabel("Option")

// UIKit — Best: use UIButton
// If custom view required: make it accessible
customView.isAccessibilityElement = true
customView.accessibilityTraits.insert(.button)
customView.accessibilityLabel = "Option name"
customView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(handleTap)))
```

### Selection state not conveyed
**Cause:** Selection indicator (checkmark, highlight, radio button, checkbox) is visual only; VoiceOver doesn't know it's selected.
**Fix:** Add the `.selected` trait.
```swift
// SwiftUI
.accessibilityAddTraits(isSelected ? .isSelected : [])

// UIKit
accessibilityTraits = isSelected ? accessibilityTraits.union(.selected) : accessibilityTraits.subtracting(.selected)
```

### VoiceOver users can't navigate by headings
**Cause:** Section titles or headings aren't marked with the header trait; Headings rotor doesn't list them.
**Fix:** Add the `.header` trait to section titles and headings.
```swift
// SwiftUI
Text("Section Title")
    .accessibilityAddTraits(.isHeader)

// UIKit
sectionTitleLabel.accessibilityTraits.insert(.header)
```

### Decorative image can be reached with VoiceOver
**Cause:** Image that doesn't convey meaningful information is still in the accessibility tree; VoiceOver users have to swipe past it.
**Fix:** Hide decorative images from assistive technologies.
```swift
// SwiftUI
Image("decoration")
    .accessibilityHidden(true)

// UIKit
decorativeImageView.isAccessibilityElement = false
```

## Common Accessibility Inspector Warnings

When Accessibility Inspector (Xcode > Window > Accessibility Inspector > Audit) reports issues, it provides suggested fixes. The most common warnings overlap with the Common Mistakes Playbook above. Use this section as a quick reference for Inspector-specific guidance.

### "Element has no label"
→ See **VoiceOver reads nothing or reads "button"** above.

### "Text doesn't support Dynamic Type"
→ See **Text doesn't scale with Dynamic Type** above.

### "Contrast ratio below 4.5:1" (or 3:1 for large text)
**Fix:** Use semantic colors or increase contrast.
```swift
// UIKit
label.textColor = .label  // Adapts to Light/Dark + Increase Contrast

// SwiftUI
Text("Content")
    .foregroundStyle(.primary)
```
→ See `good-practices.md#color-contrast`

### "Touch target size below 44x44 points"
**Fix:** Ensure minimum 44×44 points; allow larger for Dynamic Type.
```swift
// UIKit
button.translatesAutoresizingMaskIntoConstraints = false
NSLayoutConstraint.activate([
    button.widthAnchor.constraint(greaterThanOrEqualToConstant: 44),
    button.heightAnchor.constraint(greaterThanOrEqualToConstant: 44)
])

// SwiftUI
Button(action: action) {
    Image(systemName: "info")
        .padding(12)
}
.contentShape(Rectangle())
```
→ See `good-practices.md#touch-target-size`

### "Element has a label but no traits"
→ See **Custom interactive controls using tap gestures not exposed to assistive tech as buttons** above.

### "Element is not accessible"
**Fix:** Make it an accessibility element.
```swift
// UIKit
customView.isAccessibilityElement = true
customView.accessibilityLabel = "Description"

// SwiftUI - usually automatic, but check:
customView
    .accessibilityLabel("Description")
```

## Core Patterns Reference

**Important:** Always use **localized strings** for accessibility labels, values, and hints. Match your project's localization patterns (e.g., `NSLocalizedString("close_button", comment: "")` in UIKit, or `Text("close_button")` with `.xcstrings` in SwiftUI).

### When to Use Each Property

**accessibilityLabel** — Name of the element
```swift
// UIKit
closeButton.accessibilityLabel = "Close"

// SwiftUI
Button("") { close() }
    .accessibilityLabel("Close")
```

**accessibilityValue** — Current state
```swift
// UIKit
slider.accessibilityValue = "50 percent"

// SwiftUI
Slider(value: $value, in: 0...100)
    .accessibilityValue("\(Int(value)) percent")
```

**accessibilityHint** — Extra context (use sparingly; only for non-obvious actions)
```swift
// UIKit
deleteButton.accessibilityHint = "Removes the item from your list"

// SwiftUI
Button("Delete") { delete() }
    .accessibilityHint("Removes the item from your list")
```

**accessibilityTraits** — Role and state
```swift
// UIKit
sectionTitle.accessibilityTraits.insert(.header)

// SwiftUI
Text("Section Title")
    .accessibilityAddTraits(.isHeader)
```

### Common Patterns (UIKit)

**Grouping elements:**
```swift
cardView.isAccessibilityElement = true
cardView.accessibilityLabel = "\(title), \(subtitle)"
cardView.accessibilityTraits = .button
```

**Custom actions:**
```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Delete") { _ in
        self.delete()
        return true
    }
]
```

**Adjustable controls:**
```swift
// For controls with increment/decrement (sliders, steppers, etc.)
customControl.accessibilityTraits = .adjustable
customControl.accessibilityLabel = "Volume"
customControl.accessibilityValue = "\(volume)%"

// Override these methods
override func accessibilityIncrement() {
    volume = min(volume + 10, 100)
    accessibilityValue = "\(volume)%"
}

override func accessibilityDecrement() {
    volume = max(volume - 10, 0)
    accessibilityValue = "\(volume)%"
}
```

**Moving focus:**
```swift
UIAccessibility.post(notification: .layoutChanged, argument: errorLabel)
```

### Common Patterns (SwiftUI)

**Grouping elements:**
```swift
HStack {
    Image(systemName: "star.fill")
    Text("Favorite")
}
.accessibilityElement(children: .combine)
```

**Custom actions:**
```swift
.accessibilityAction(named: "Delete") {
    deleteItem()
}
```

**Adjustable controls:**
```swift
.accessibilityAdjustableAction { direction in
    switch direction {
    case .increment: value += 1
    case .decrement: value -= 1
    @unknown default: break
    }
}
```

**Moving focus (iOS 15+ with [`AccessibilityFocusState`](https://developer.apple.com/documentation/swiftui/accessibilityfocusstate)):**
```swift
@AccessibilityFocusState private var isFocused: Bool

// Move focus to element
Button("Submit") { submit() }
    .accessibilityFocused($isFocused)

// Trigger focus
isFocused = true
```

## Testing Workflow

Canonical testing guidance lives in:
- `testing-manual.md` for assistive-technology and settings workflows
- `testing-automated.md` for audits, UI tests, and CI guardrails

Use this quick sequence in day-to-day development:
1. **During development:** Run Accessibility Inspector checks.
2. **Before PR:** Validate VoiceOver and Dynamic Type end-to-end on key flows.
3. **Regression prevention:** Add or update automated checks where they provide signal.
4. **Before release:** Run the full manual testing checklist.
5. **Continuous improvement:** Include feedback from users with disabilities when possible.

## iOS Version-Specific APIs

Some accessibility features require specific iOS versions. Check deployment target before recommending these APIs; provide fallbacks when possible.

### iOS 13+
- [Large Content Viewer (`UILargeContentViewerItem`)](https://developer.apple.com/documentation/uikit/uilargecontentvieweritem), [`UILargeContentViewerInteraction`](https://developer.apple.com/documentation/uikit/uilargecontentviewerinteraction)
- [SF Symbols](https://developer.apple.com/sf-symbols/)
- [`preferredContentSizeCategory.isAccessibilityCategory`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/2897384-isaccessibilitycategory)

### iOS 14+
- [Switch Control custom action images (`UIAccessibilityCustomAction.init(name:image:actionHandler:)`)](https://developer.apple.com/documentation/uikit/uiaccessibilitycustomaction/init(name:image:actionhandler:))

### iOS 15+
- [`AccessibilityFocusState`](https://developer.apple.com/documentation/swiftui/accessibilityfocusstate) for programmatic focus management (SwiftUI)
- [`.accessibilityRotor`](https://developer.apple.com/documentation/swiftui/view/accessibilityrotor(_:entries:entryid:entrylabel:)) for custom rotors (SwiftUI)

### iOS 16+
- [`.accessibilityRepresentation`](https://developer.apple.com/documentation/swiftui/view/accessibilityrepresentation(representation:)) for custom control alternatives (SwiftUI)
- [`.accessibilityActions { }` syntax](https://developer.apple.com/documentation/swiftui/view/accessibilityactions(content:))

### iOS 17+
- [`.sensoryFeedback()`](https://developer.apple.com/documentation/swiftui/view/sensoryfeedback(_:trigger:)) for haptic responses

## Common Scenarios → Quick Navigation

Use this table to quickly find solutions for specific problems (framed as user-experience issues):

| Scenario | Common Mistakes Section | Reference File |
|----------|------------------------|----------------|
| Button doesn't work with VoiceOver | VoiceOver reads nothing or "button" | `voiceover-*.md` |
| Cell requires many swipes | VoiceOver reads each element separately | `voiceover-*.md` (Grouping) |
| Custom control reads many "button"s | VoiceOver reads a custom control as many separate elements | `voiceover-*.md` (Adjustable) |
| Text truncates at large sizes | Text doesn't scale with Dynamic Type | `dynamic-type-*.md` |
| Layout breaks at large text | Layout breaks at large text sizes | `dynamic-type-*.md` (Adaptation) |
| Toast disappears too fast | Toast/snackbar disappears before VoiceOver | `voiceover-*.md` (Announcements) |
| Voice Control can't find button | Voice Control can't find or activate button | `voice-control.md` |
| Images wrong with Smart Invert | Images look wrong with Smart Invert | `good-practices.md` |
| Custom view/button not activatable | VoiceOver can't activate custom view | `voiceover-*.md` (Traits) |
| Selection state not conveyed | Selection state not conveyed | `voiceover-*.md` (Selected trait) |
| VoiceOver can't navigate by headings | VoiceOver users can't navigate by headings | `voiceover-*.md` (Headers) |
| Decorative image in VoiceOver order | Decorative image can be reached with VoiceOver | `voiceover-*.md` (Hidden) |

## Best Practices Summary

**Goal:** The app should support assistive technologies without losing any content or functionality. Use the Common Mistakes Playbook and Anti-Patterns section together with this list.

1. **Label everything interactive** — Every button, control, and image needs a label or must be hidden
2. **Use traits correctly** — Traits communicate role; don't rely on labels alone
3. **Group related content** — Reduce swipe count and cognitive load
4. **Support Dynamic Type** — Use text styles, not fixed sizes
5. **Test with assistive tech** — Automation catches basics; manual testing finds real issues
6. **Move focus appropriately** — After navigation or errors, move VoiceOver focus
7. **Provide alternatives** — Custom gestures need accessible fallbacks
8. **Honor user settings** — Respect Reduce Motion, Increase Contrast, Bold Text
9. **Think multi-modal** — Don't rely on color alone; use icons, text, haptics
10. **Iterate with users** — Validate with real feedback and keep refining

## Verification Checklist (After Making Changes)

### For Agents (Automated Checks)

Use these checks when suggesting or applying accessibility changes:

- [ ] Build succeeds with no new warnings
- [ ] Run existing unit tests; no new failures
- [ ] No breaking API changes for the project's deployment target
- [ ] APIs used match the project's iOS version (see Project Capabilities)
- [ ] Pattern consistency: UIKit vs SwiftUI matches the file being edited; labels/traits follow existing project style
- [ ] Linter / static analysis shows no new errors

### For Developers (Manual Testing)

Use this checklist when verifying changes before PR or release. Agents can suggest these steps; developers perform them.

- [ ] **Accessibility Inspector** (Xcode > Window > Accessibility Inspector > Audit)
  - Color contrast ratios (≥4.5:1 for text, ≥3:1 for UI elements)
  - Touch target sizes (≥44×44 points)
  - All interactive elements have labels

- [ ] **VoiceOver**: Test end-to-end with VoiceOver enabled (on device when possible; consider Screen Curtain)
  - Navigate through the changed views
  - Verify labels, values, traits are correct
  - Test custom actions via Actions rotor
  - Test header navigation via Headings rotor
  - Confirm focus moves appropriately after state changes

- [ ] **Dynamic Type**: Test with largest accessibility size (Accessibility 5)
  - Text doesn't truncate
  - Layout adapts (horizontal → vertical if needed)
  - No loss of content or functionality

- [ ] **Voice Control**: Test key flows with Voice Control
  - Say "Show names" and verify labels appear
  - Say "Tap [element name]" for main interactive elements

- [ ] **Full Keyboard Access** (if applicable): Test keyboard navigation
  - Tab through all interactive elements
  - All interactive elements can be reached
  - Focus order is logical

- [ ] **User settings**: Reduce Motion, Increase Contrast, Bold Text, Button Shapes, Dark Mode — test that the app adapts and contrast remains sufficient

- [ ] **Documentation**: Update accessibility identifiers if changed (for UI tests); note manual testing requirements for QA; add code comments for workarounds or compromises

## Review Checklist (Quick Check)

### Labels and Traits
- [ ] All interactive elements have labels
- [ ] Labels are concise and don't include control type
- [ ] Traits match the role
- [ ] State changes update values (and traits when relevant, e.g. selected)

### Structure
- [ ] Related elements grouped
- [ ] Decorative elements hidden
- [ ] Navigation order is logical
- [ ] Focus moves after state changes

### Dynamic Type
- [ ] Text scales with system size
- [ ] Layout adapts for large sizes
- [ ] No truncation (unless intentional)

### Testing
- [ ] VoiceOver tested end-to-end
- [ ] Dynamic Type tested at accessibility sizes
- [ ] Voice Control can activate all buttons
- [ ] Keyboard navigation works

## Sources

- [Accessibility Up To 11](https://accessibilityupto11.com)
- [Developing Accessible iOS Apps](https://link.springer.com/book/10.1007/978-1-4842-5308-3)
