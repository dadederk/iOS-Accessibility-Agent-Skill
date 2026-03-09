# Good Practices

Cross-cutting accessibility guidance: touch targets, color contrast, motion, transparency, haptics, and multi-modal design.

## Contents

- [Touch Target Size](#touch-target-size)
- [Color Contrast](#color-contrast)
- [Don't Rely on Color Alone](#dont-rely-on-color-alone)
- [Avoid Text in Images](#avoid-text-in-images)
- [Media Accessibility](#media-accessibility)
- [Hearing Accommodations](#hearing-accommodations)
- [Reduce Motion](#reduce-motion)
- [Reduce Transparency](#reduce-transparency)
- [Video Playback Preferences](#video-playback-preferences)
- [Semantic Colors](#semantic-colors)
- [Bold Text](#bold-text)
- [Button Shapes](#button-shapes)
- [Haptic Feedback](#haptic-feedback)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Multi-Modal Information](#multi-modal-information)
- [Multiple Input Paths](#multiple-input-paths)
- [Orientation Support](#orientation-support)
- [Avoid Ephemeral Feedback](#avoid-ephemeral-feedback)
- [Alt Text for Shared Images](#alt-text-for-shared-images)
- [Smart Invert](#smart-invert)
- [Invert Colors (Classic)](#invert-colors-classic)
- [Toggles and Switches](#toggles-and-switches)
- [Watchables and Wearables](#watchables-and-wearables)
- [Checklist](#checklist)

## Touch Target Size

Apple recommends a minimum tappable area of **44×44 points**.

Also try to keep targets **at least 32 points apart** to reduce accidental taps, especially for users with tremors or low vision.

If you can’t increase spacing, increase the hit area using insets or `contentShape`.

### Common violations

- Navigation bar buttons
- Custom toolbar icons
- Dismiss/close buttons
- Inline text links

### Fix small targets

Expand the hit area without changing appearance:

**UIKit**:
```swift
button.contentEdgeInsets = UIEdgeInsets(top: 12, left: 12, bottom: 12, right: 12)
```

Or override `point(inside:with:)`:
```swift
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
    bounds.insetBy(dx: -10, dy: -10).contains(point)
}
```

**SwiftUI**:
```swift
Button(action: dismiss) {
    Image(systemName: "xmark")
        .padding(12)
}
.contentShape(Rectangle())
```

## Color Contrast

### Minimum ratios (WCAG 2.1)

| Text size | Ratio |
|-----------|-------|
| Normal text (<18pt) | 4.5:1 |
| Large text (≥18pt or 14pt bold) | 3:1 |
| Non-text (icons, borders) | 3:1 |

### Test contrast

- **Accessibility Inspector**: Window > Color Contrast Calculator
- Online tools: WebAIM Contrast Checker

### High contrast support

Provide alternate colors for **Increase Contrast** setting:

**Asset Catalog**: Add High Contrast appearance variants.

**UIKit**:
```swift
if UIAccessibility.isDarkerSystemColorsEnabled {
    label.textColor = .label // Higher contrast
}
```

**SwiftUI** — check the environment:
```swift
@Environment(\.colorSchemeContrast) private var contrast

var buttonColor: Color {
    contrast == .increased ? .primary : .accentColor
}

var body: some View {
    Button(action: action) {
        Text(title)
    }
    .foregroundStyle(buttonColor)
}
```

Semantic system colors (`.primary`, `.secondary`) adapt automatically, but use `colorSchemeContrast` when you need custom behavior.

## Don't Rely on Color Alone

Users with color blindness or **Differentiate Without Color** enabled need additional cues.

### Bad

```swift
statusLabel.textColor = status == .error ? .red : .green
```

### Good

```swift
statusLabel.text = status == .error ? "⚠️ Error" : "✓ Success"
statusLabel.textColor = status == .error ? .systemRed : .systemGreen
```

Use icons, shapes, patterns, or text alongside color.

## Avoid Text in Images

Text baked into images is not readable by VoiceOver, not scalable with Dynamic Type, and not localizable.

Use real text whenever possible. If you must use an image that contains text:
- Provide a localized `accessibilityLabel`
- Replace the image per language when needed
- Prefer vector PDFs so images stay sharp at larger sizes

## Media Accessibility

If your app plays audio or video, provide multiple ways to access the content:

- **Captions/Subtitles** for all spoken content
- **SDH** (subtitles for the deaf and hard of hearing) when available
- **Audio descriptions** for visually important content
- **Transcripts** for long‑form audio/video

Use the system captions preference as the default when possible.

## Hearing Accommodations

For users who are deaf or hard of hearing, include alternatives to sound:

- **LED Flash for Alerts** (system setting) is a common way users notice notifications
- **Mono Audio** helps users with hearing loss in one ear
- **Audio balance** lets users favor left/right channels

If your app provides custom audio controls, avoid breaking system preferences.

### Check the setting

```swift
if UIAccessibility.shouldDifferentiateWithoutColor {
    // Add extra visual cues
}
```

## Reduce Motion

Users enable **Reduce Motion** to minimize vestibular triggers.

### Honor the setting

**UIKit**:
```swift
if UIAccessibility.isReduceMotionEnabled {
    // Use fade instead of slide
    // Disable parallax
    // Stop auto-playing animations
}
```

**SwiftUI** — environment value:
```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

.animation(reduceMotion ? nil : .default, value: isExpanded)
```

### Check during user interactions

For real-time checks during interactions (sliders, scrubbing), use the UIKit API:

```swift
.onChange(of: sliderValue) { _, newValue in
    let reduceMotion = UIAccessibility.isReduceMotionEnabled
    
    if !reduceMotion {
        // Animate UI updates during scrubbing
        withAnimation(.easeInOut(duration: 0.3)) {
            updateVisualFeedback(newValue)
        }
    } else {
        // Skip animation, update immediately
        updateVisualFeedback(newValue)
    }
}
```

### Example: Slider with conditional animation

```swift
Slider(value: $progress)
    .onChange(of: progress) { _, newValue in
        if !UIAccessibility.isReduceMotionEnabled {
            withAnimation { currentLineIndex = calculateLine(newValue) }
        } else {
            currentLineIndex = calculateLine(newValue)
        }
    }
```

### Observe changes

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleReduceMotion),
    name: UIAccessibility.reduceMotionStatusDidChangeNotification,
    object: nil
)
```

## Reduce Transparency

Simplify translucent backgrounds when **Reduce Transparency** is enabled:

**UIKit**:
```swift
override func awakeFromNib() {
    super.awakeFromNib()
    
    NotificationCenter.default.addObserver(
        self,
        selector: #selector(updateBackground),
        name: UIAccessibility.reduceTransparencyStatusDidChangeNotification,
        object: nil
    )
    updateBackground()
}

@objc private func updateBackground() {
    let opacity = UIAccessibility.isReduceTransparencyEnabled ? 1.0 : 0.9
    backgroundView.backgroundColor = UIColor.secondarySystemBackground.withAlphaComponent(opacity)
}
```

**Note:** `UIVisualEffectView` and system navigation bars already respect Reduce Transparency. Custom alpha on views does not—handle it manually.

**SwiftUI**:
```swift
@Environment(\.accessibilityReduceTransparency) var reduceTransparency

var body: some View {
    Text(message)
        .background(Color(UIColor.secondarySystemBackground)
            .opacity(reduceTransparency ? 1.0 : 0.90))
}
```

## Video Playback Preferences

### Auto-Play Video Previews

Respect the system Auto-Play preference for motion-heavy previews:

```swift
// Use system preference as the default
if UIAccessibility.isVideoAutoplayEnabled {
    startPreviewPlayback()
} else {
    showStaticThumbnail()
}

// Observe changes
NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleAutoplayPreferenceChange),
    name: UIAccessibility.videoAutoplayStatusDidChangeNotification,
    object: nil
)
```

If you also have an in-app preference, use the system setting as the default and let users override it explicitly.

### Closed Captions

If your app plays video, honor the system captions preference:

```swift
if UIAccessibility.isClosedCaptioningEnabled {
    enableClosedCaptions()
}
```

Prefer **SDH** (Subtitles for the Deaf and Hard of Hearing) when available.

## Semantic Colors

Use semantic system colors for better contrast and automatic Light/Dark mode support:

**UIKit**:
```swift
// Instead of:
label.textColor = UIColor.darkGray

// Use:
label.textColor = .secondaryLabel

// Common semantic colors:
// .label            - Primary text
// .secondaryLabel   - Secondary text (adapts to Dark mode + Increase Contrast)
// .systemBackground - Primary background
// .secondarySystemBackground - Secondary background
```

**SwiftUI**:
```swift
Text("Title")
    .foregroundStyle(.primary)
Text("Subtitle")
    .foregroundStyle(.secondary)
```

Using semantic colors gives you Light/Dark mode and Increase Contrast support (4 combinations) for free.

## Bold Text

When **Bold Text** is enabled, system fonts become bolder automatically. Custom fonts and non-text elements need manual handling.

**UIKit**:
```swift
if UIAccessibility.isBoldTextEnabled {
    label.font = UIFont(name: "Avenir-Heavy", size: 17)
} else {
    label.font = UIFont(name: "Avenir-Medium", size: 17)
}
```

For SF Symbols, use weighted variants:

```swift
let config = UIImage.SymbolConfiguration(weight: UIAccessibility.isBoldTextEnabled ? .bold : .regular)
imageView.preferredSymbolConfiguration = config
```

**SwiftUI** — use `legibilityWeight` environment:
```swift
@Environment(\.legibilityWeight) private var legibilityWeight

var fontWeight: Font.Weight {
    legibilityWeight == .bold ? .bold : .regular
}

Text("Content")
    .fontWeight(fontWeight)
```

### Scale non-text elements with Bold Text

Increase border widths, icon weights, and other visual elements:

```swift
@Environment(\.legibilityWeight) private var legibilityWeight
@ScaledMetric(relativeTo: .body) private var baseBorderWidth: CGFloat = 2.0

private var borderWidth: CGFloat {
    legibilityWeight == .bold ? baseBorderWidth * 2 : baseBorderWidth
}

var body: some View {
    content
        .overlay(
            RoundedRectangle(cornerRadius: 8)
                .strokeBorder(.tint, lineWidth: borderWidth)
        )
}
```

System fonts adapt automatically — use `legibilityWeight` for custom visual elements.

## Button Shapes

When **Button Shapes** is enabled, buttons show underlines or borders. Standard buttons handle this automatically; custom buttons may need attention.

**UIKit**:
```swift
if UIAccessibility.buttonShapesEnabled {
    // Add visual border or underline to custom buttons
}
```

**SwiftUI**:
```swift
@Environment(\.accessibilityShowButtonShapes) private var showButtonShapes

var body: some View {
    Button(action: onTap) {
        Text(title)
            .padding()
    }
    .overlay {
        if showButtonShapes {
            RoundedRectangle(cornerRadius: 8)
                .strokeBorder(.primary, lineWidth: 1)
        }
    }
}
```

### Example: Custom list row buttons

```swift
struct TranscriptLineView: View {
    @Environment(\.accessibilityShowButtonShapes) private var showButtonShapes
    
    var body: some View {
        Button(action: onTap) {
            content
                .background(backgroundColor)
                .overlay(buttonShapeOverlay)
        }
        .buttonStyle(.plain)
    }
    
    @ViewBuilder
    private var buttonShapeOverlay: some View {
        if showButtonShapes {
            RoundedRectangle(cornerRadius: 8)
                .strokeBorder(.primary, lineWidth: 1)
        }
    }
}
```

## Haptic Feedback

Use haptics to reinforce important events — but never as the only feedback channel.

```swift
let generator = UINotificationFeedbackGenerator()
generator.notificationOccurred(.success) // .warning, .error
```

```swift
let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()
```

Use `.success` for completions, `.warning` for caution, `.error` for failures.

When feedback style matters (for example, rhythm games, timers, coaching cues), give users control over channels (audio, haptic, visual) instead of forcing one output mode.

## Keyboard Shortcuts

Provide shortcuts for key actions on iPad and external keyboards.

**UIKit**:
```swift
override var keyCommands: [UIKeyCommand]? {
    [
        UIKeyCommand(title: "Refresh", action: #selector(refresh), input: "r", modifierFlags: .command),
        UIKeyCommand(title: "Search", action: #selector(search), input: "f", modifierFlags: .command)
    ]
}
```

**SwiftUI**:
```swift
Button("Refresh", action: refresh)
    .keyboardShortcut("r", modifiers: .command)
```

Shortcuts appear when the user holds the Command key.

## Multi-Modal Information

Convey important information through multiple channels:

| Channel | Example |
|---------|---------|
| Visual | Error icon |
| Text | "Password is too short" |
| Color | Red text |
| Haptic | Error feedback |
| Sound | Alert tone |

Never rely on a single channel.

## Multiple Input Paths

For time-sensitive or precision-heavy flows (for example, games, media controls, drawing tools), avoid forcing one interaction method.

Provide at least two reliable input paths where possible:
- Touch gestures and on-screen controls
- Hardware keyboard shortcuts
- External controllers or alternate navigation patterns

This improves access for users who cannot perform one specific gesture pattern consistently.

## Orientation Support

Support both portrait and landscape when possible. Don't force users to rotate their devices.

```swift
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
    .all
}
```

Some users mount devices in fixed orientations.

## Avoid Ephemeral Feedback

Snackbars and toasts that disappear quickly are problematic:

- VoiceOver users may miss them
- Zoom users can't see them
- Slow readers can't finish them

For critical information, use:
- Persistent banners
- Confirmation dialogs
- Inline error messages

## Alt Text for Shared Images

If your app lets users share images, provide a way to add alt text:

```swift
let attachment = UIDragItem(itemProvider: provider)
attachment.localObject = ["image": image, "altText": altText]
```

Platforms like Twitter and Slack do this well.

## Smart Invert

Prevent images and media from inverting with Smart Invert:

```swift
imageView.accessibilityIgnoresInvertColors = true
videoPlayer.accessibilityIgnoresInvertColors = true
```

## Invert Colors (Classic)

Some users enable **Invert Colors** (Settings > Accessibility > Display & Text Size). If you use custom color combinations, you may want to adjust them when inversion is enabled.

```swift
if UIAccessibility.isInvertColorsEnabled {
    // Adjust custom colors if needed
    contentView.backgroundColor = .systemBackground
}

NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleInvertColorsChange),
    name: UIAccessibility.invertColorsStatusDidChangeNotification,
    object: nil
)

@objc private func handleInvertColorsChange() {
    // Update custom colors when setting changes
}
```

Avoid relying on color alone—use icons and labels for context, so inversion does not remove meaning.

## Toggles and Switches

Group switches with their labels:

**UIKit**:
```swift
// Place switch in table view cell accessory
cell.accessoryView = toggle
cell.accessibilityLabel = "Notifications"
cell.accessibilityValue = toggle.isOn ? "On" : "Off"
```

**SwiftUI**:
```swift
Toggle("Notifications", isOn: $notificationsEnabled)
```

The standard `Toggle` handles accessibility automatically.

### Announce setting changes

When a toggle affects other settings or has side effects, announce the change:

```swift
Toggle("Respect Assistive Technology Settings", isOn: $prefersATSettings)
    .onChange(of: prefersATSettings) { _, newValue in
        // Update related settings
        if newValue {
            customVoiceEnabled = false
        }
        
        // Announce the change
        announceChange(newValue 
            ? "Custom voice disabled" 
            : "Custom voice enabled"
        )
    }

private func announceChange(_ message: String) {
    Task { @MainActor in
        // Small delay ensures VoiceOver finishes reading the toggle
        try? await Task.sleep(nanoseconds: 100_000_000)
        UIAccessibility.post(notification: .announcement, argument: message)
    }
}
```

This helps VoiceOver users understand cascading effects of their choices.

## Watchables and Wearables

### Assistive Touch on Apple Watch

Users can navigate with hand gestures (pinch, clench). If your watch app supports VoiceOver, Assistive Touch likely works too.

### Quick actions

Implement a quick action for the most important task:

```swift
.accessibilityQuickAction(style: .prompt) {
    Button("Play") { play() }
}
```

Triggered with a double-pinch gesture.

## Checklist

- [ ] Touch targets at least 44×44 points
- [ ] Color contrast meets minimums (4.5:1 for text)
- [ ] Increase Contrast honored for custom colors
- [ ] Information conveyed in multiple modes (not just color)
- [ ] Reduce Motion honored (including during interactions)
- [ ] Reduce Transparency honored
- [ ] Bold Text supported for custom fonts and borders
- [ ] Button Shapes honored for custom buttons
- [ ] Haptics used for key events
- [ ] Keyboard shortcuts for main actions
- [ ] Both orientations supported
- [ ] Ephemeral messages replaced with persistent alternatives
- [ ] Images/video ignore Smart Invert
- [ ] Toggles grouped with labels
- [ ] Setting changes announced when they have side effects

## Sources

- [Accessibility Up To 11 — #365DaysIOSAccessibility](https://accessibilityupto11.com/365-days-ios-accessibility/)
- [From Zero to Accessible](https://github.com/dadederk/fromZeroToAccessible) (Daniel Devesa Derksen-Staats and Rob Whitaker)
