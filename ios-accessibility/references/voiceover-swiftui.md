# VoiceOver — SwiftUI

SwiftUI implementation for VoiceOver accessibility.

For core concepts, see `voiceover.md`.

## Contents

- [Labels](#labels)
- [Values](#values)
- [Hints](#hints)
- [Traits](#traits)
- [Grouping](#grouping)
- [Navigation Order](#navigation-order)
- [Images](#images)
- [Custom Actions](#custom-actions)
- [Accessibility Custom Content](#accessibility-custom-content)
- [Adjustable Controls](#adjustable-controls)
- [Accessibility Representation](#accessibility-representation)
- [Gestures](#gestures)
- [Focus Management](#focus-management)
- [Announcements](#announcements)
- [Modal Dialogs](#modal-dialogs)
- [Custom Rotor](#custom-rotor)
- [Example: Grouped Card with Actions](#example-grouped-card-with-actions)

## Labels

### Basic label

```swift
Button(action: play) {
    Image(systemName: "play.fill")
}
.accessibilityLabel("Play")
```

### Labelled icon-only button

```swift
Button(action: play) {
    Label("Play", systemImage: "play.fill")
}
.labelStyle(.iconOnly)
```

This keeps a semantic text label while rendering an icon-only control.

### Text automatically used as label

```swift
Button("Submit", action: submit)
// accessibilityLabel is "Submit" automatically
```

### State-driven labels

When control meaning changes with state, update the label to reflect the current action:

```swift
struct PlayButton: View {
    @Binding var isPlaying: Bool
    
    var body: some View {
        Button(action: { isPlaying.toggle() }) {
            Image(systemName: isPlaying ? "pause.fill" : "play.fill")
        }
        .accessibilityLabel(isPlaying ? "Pause" : "Play")
    }
}
```

### Combine text from multiple views

Use `.combine` for automatic merging. If you need strict control over phrasing (for example, avoid punctuation pauses), use `.ignore` and set a manual label.

```swift
// Automatic merge
HStack {
    Text("Price")
    Text("$42.00")
}
.accessibilityElement(children: .combine)
// VoiceOver reads: "Price, $42.00"

// Manual utterance
HStack {
    Text("Price of")
    Text("$42.00")
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Price of $42.00")
```

### Badge with value pattern

When a visual badge shows a count, keep a stable label and put the changing part in the value:

```swift
// Before: confusing label changes
Button { showBasket.toggle() } label: {
    ZStack(alignment: .topTrailing) {
        Image(systemName: "cart.fill")
        if basket.orderCount > 0 {
            Text("\(basket.orderCount)")
                .background(.red)
                .clipShape(Capsule())
        }
    }
}
// VoiceOver reads: "cart.fill" or "3" depending on state — confusing

// After: consistent label with dynamic value
Button { showBasket.toggle() } label: {
    ZStack(alignment: .topTrailing) {
        Image(systemName: "cart.fill")
        if basket.orderCount > 0 {
            Text("\(basket.orderCount)")
                .background(.red)
                .clipShape(Capsule())
        }
    }
}
.accessibilityLabel("Cart")
.accessibilityValue("\(basket.orderCount) items")
// VoiceOver reads: "Cart, 3 items, button"
```

## Values

```swift
Slider(value: $volume, in: 0...10000)
    .accessibilityValue("\(Int(volume)) steps")
```

Values update automatically with bindings.

## Hints

```swift
Button("Delete", action: delete)
    .accessibilityHint("Removes the item from your list")
```

Hints are optional but help describe unusual controls:
Prefer hint phrasing that describes the result of the action (for example, "Removes the item from your list").

```swift
HStack {
    ForEach(1..<6) { value in
        Button { rating = value } label: {
            Image(systemName: value <= rating ? "hand.thumbsup.fill" : "hand.thumbsup")
        }
    }
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Rating")
.accessibilityValue("\(rating) thumbs up")
.accessibilityHint("Rates your drink from 1 to 5 thumbs up")
```

## Traits

### Header trait

Mark section headings for Rotor navigation:

```swift
Text("Settings")
    .font(.headline)
    .accessibilityAddTraits(.isHeader)
```

### Heading levels

For documents with hierarchical structure, use semantic heading levels for screen reader navigation (iOS 17+) with [`.accessibilityHeading(_:)`](https://developer.apple.com/documentation/swiftui/view/accessibilityheading(_:)):

```swift
Text("Main Title")
    .font(.largeTitle)
    .accessibilityAddTraits(.isHeader)
    .accessibilityHeading(.h1)

Text("Section")
    .font(.title2)
    .accessibilityAddTraits(.isHeader)
    .accessibilityHeading(.h2)

Text("Subsection")
    .font(.title3)
    .accessibilityAddTraits(.isHeader)
    .accessibilityHeading(.h3)
```

Available levels: `.h1` through `.h6`, plus `.unspecified` (iOS 17+, see [`.accessibilityHeading(_:)`](https://developer.apple.com/documentation/swiftui/view/accessibilityheading(_:))).

### Updates frequently trait

For values that change rapidly (timers, progress, live data), add `.updatesFrequently` so VoiceOver can batch updates:

```swift
Slider(value: $progress, in: 0...100)
    .accessibilityLabel("Progress")
    .accessibilityValue("Line \(currentLine) of \(totalLines)")
    .accessibilityAddTraits(.updatesFrequently)
```

This prevents VoiceOver from interrupting itself when values change quickly during playback or animation.

### Selected trait

For picker options, toggle states, and segmented controls:

```swift
ForEach(MilkOptions.allCases, id: \.self) { milk in
    Button {
        selectedMilk = milk
    } label: {
        HStack {
            Text(milk.rawValue)
            Spacer()
            Image(systemName: selectedMilk == milk ? "checkmark.circle" : "circle")
                .accessibilityHidden(true)  // Visual only
        }
    }
    .buttonStyle(.plain)
    .accessibilityAddTraits(selectedMilk == milk ? .isSelected : [])
}
```

### Common traits

| Trait | Use |
|-------|-----|
| `.isButton` | Tappable control |
| `.isHeader` | Section heading (Rotor navigation) |
| `.isSelected` | Currently selected option |
| `.isLink` | Opens URL |
| `.isImage` | Image content |
| `.isStaticText` | Non-interactive text |
| `.isModal` | Modal dialog |
| `.updatesFrequently` | Rapidly changing value |

### Use `.disabled()` instead of `.notEnabled`

In UIKit, `.notEnabled` is an explicit trait. In SwiftUI, use `.disabled(...)` and let the framework expose the disabled accessibility state and interaction behavior.

```swift
Button("Submit", action: submit)
    .disabled(!isValid)
```

## Grouping

### accessibilityElement(children:)

| Option | Behavior |
|--------|----------|
| `.ignore` | Single element; set label, and other properties, manually |
| `.combine` | Merge child labels, and other properties, automatically |
| `.contain` | Group semantically; children still individually accessible |

Prefer `.combine` first — it updates automatically when content changes.

### Group a custom control

UIKit usually requires adding `.adjustable` explicitly. In SwiftUI, `.accessibilityAdjustableAction` implicitly exposes adjustable behavior.

Prefer `.accessibilityRepresentation { }` when an existing standard control maps well to your custom control, so the interaction pattern stays familiar.

When multiple buttons form one logical control, group them:

```swift
// Before: VoiceOver reads 5 separate buttons
HStack {
    ForEach(1..<6) { value in
        Button { rating = value } label: {
            Image(systemName: value <= rating ? "hand.thumbsup.fill" : "hand.thumbsup")
        }
    }
}

// After: single adjustable control
HStack {
    ForEach(1..<6) { value in
        Button { rating = value } label: {
            Image(systemName: value <= rating ? "hand.thumbsup.fill" : "hand.thumbsup")
        }
    }
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Rating")
.accessibilityValue("\(rating) thumbs up")
.accessibilityAdjustableAction { direction in
    switch direction {
    case .increment:
        guard rating < 5 else { return }
        rating += 1
    case .decrement:
        guard rating > 1 else { return }
        rating -= 1
    @unknown default:
        break
    }
}
```

### Group a list row

Put everything inside NavigationLink's label for automatic grouping:

```swift
// Before: hidden NavigationLink, separate button
ZStack {
    HStack {
        Text(drink.name)
        Button("Add to cart") { basket.add(Order(drink: drink)) }
    }
    NavigationLink { DrinkDetail(drink: drink) } label: { EmptyView() }
        .opacity(0)
}

// After: proper grouping with custom action
NavigationLink {
    DrinkDetail(drink: drink)
} label: {
    HStack {
        Text(drink.name)
        Text(CurrencyFormatter.format(drink.basePrice))
    }
}
.accessibilityAction(named: "Add to cart") {
    basket.add(Order(drink: drink))
}
```

## Navigation Order

### Sort priority

Change reading order with priority (higher numbers read first):

```swift
VStack {
    Text("Read second").accessibilitySortPriority(1)
    Text("Read first").accessibilitySortPriority(2)
}
```

Default priority is 0. Use positive numbers for elements that should be read earlier.

## Images

### Decorative images

```swift
Image(decorative: "background-pattern")

// or:
Image("background")
    .accessibilityHidden(true)
```

### Smart Invert support

Prevent photos and meaningful images from inverting:

```swift
Image(imageName)
    .resizable()
    .accessibilityHidden(true)  // Decorative
    .accessibilityIgnoresInvertColors()  // Don't invert with Smart Invert
```

### Meaningful images

```swift
Image("profile-photo")
    .accessibilityLabel("Profile photo of Johnny Appleseed")
```

## Custom Actions

Expose hidden or secondary actions:

```swift
// Before: button inside cell is hard to reach
HStack {
    Text(drink.name)
    Button("Add to cart") { basket.add(Order(drink: drink)) }
}

// After: action accessible via VoiceOver
NavigationLink { DrinkDetail(drink: drink) } label: {
    Text(drink.name)
}
.accessibilityAction(named: "Add to cart") {
    basket.add(Order(drink: drink))
}
```

### Multiple actions

```swift
.accessibilityAction(named: "Delete") { delete() }
.accessibilityAction(named: "Share") { share() }
```

### iOS 16+ syntax

```swift
.accessibilityActions {
    Button("Delete", action: delete)
    Button("Share", action: share)
}
```

This syntax requires iOS 16+ ([Apple docs](https://developer.apple.com/documentation/swiftui/view/accessibilityactions(content:))).

## Accessibility Custom Content

Use this for supplementary details in data-rich UI without overloading the primary label/value:
Whenever possible, make the same information independently accessible elsewhere too (for example, in a details screen).

```swift
VStack(alignment: .leading) {
    Text("AAPL")
    Text("$182.34")
}
.accessibilityLabel("Apple stock")
.accessibilityValue("$182.34")
.accessibilityCustomContent("Daily change", value: "+1.8 percent")
.accessibilityCustomContent("52-week range", value: "124 to 199")
.accessibilityCustomContent("Risk", value: "Moderate", importance: .high)
```

Use custom content for secondary context. Keep primary action/state in label, value, traits, and actions.

## Adjustable Controls

Replace multiple buttons with a single adjustable control:

```swift
struct RatingView: View {
    @Binding var rating: Int

    var body: some View {
        HStack {
            ForEach(1...5, id: \.self) { star in
                Image(systemName: star <= rating ? "star.fill" : "star")
            }
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Rating")
        .accessibilityValue("\(rating) of 5")
        .accessibilityAdjustableAction { direction in
            switch direction {
            case .increment:
                rating = min(rating + 1, 5)
            case .decrement:
                rating = max(rating - 1, 1)
            @unknown default:
                break
            }
        }
    }
}
```

## Accessibility Representation

Replace complex custom controls with standard accessible equivalents:

```swift
// Before: custom plus/minus icons with tap gestures
HStack {
    Image(systemName: "minus.circle")
        .onTapGesture { shots = max(0, shots - 1) }
    Text("\(shots) shots")
    Image(systemName: "plus.circle")
        .onTapGesture { shots = min(4, shots + 1) }
}

// After: assistive tech sees a slider
HStack {
    Image(systemName: "minus.circle")
        .onTapGesture { shots = max(0, shots - 1) }
    Text("\(shots) shots")
    Image(systemName: "plus.circle")
        .onTapGesture { shots = min(4, shots + 1) }
}
.accessibilityRepresentation {
    Slider(value: $shots, in: 0...4, step: 1)
        .accessibilityLabel("Extra shots")
        .accessibilityValue("\(Int(shots)) shots")
}
```

Users see your custom UI; VoiceOver interacts with the Slider.

## Gestures

### Magic Tap

Two-finger double-tap triggers the primary action. Ideal for play/pause in media apps:

```swift
struct PlayerView: View {
    @State private var isPlaying = false
    
    var body: some View {
        VStack {
            // Player UI...
        }
        .accessibilityAction(.magicTap) {
            isPlaying.toggle()
        }
    }
}
```

Apply to the outermost view so Magic Tap works anywhere on screen:

```swift
NavigationStack {
    TranscriptView()
}
.accessibilityAction(.magicTap) {
    audioManager.isPlaying ? audioManager.pause() : audioManager.play()
}
```

### Escape

```swift
.accessibilityAction(.escape) {
    dismiss()
}
```

### Direct touch for real-time interactions

For fast, continuous interactions (for example, music apps, drawing canvases, and some game controls), direct touch can reduce friction:

```swift
JoystickView()
    .accessibilityDirectTouch(true, options: [.silentOnTouch])
```

Use this sparingly. Prefer regular VoiceOver navigation unless the interaction truly depends on real-time touch movement.

## Focus Management

### Move focus programmatically

```swift
@AccessibilityFocusState private var isFocused: Bool

var body: some View {
    VStack {
        TextField("Name", text: $name)
            .accessibilityFocused($isFocused)
        Button("Focus Field") {
            isFocused = true
        }
    }
}
```

### Focus on error

```swift
struct FormView: View {
    @State private var email = ""
    @State private var showError = false
    @AccessibilityFocusState private var isErrorFocused: Bool
    
    var body: some View {
        VStack {
            TextField("Email", text: $email)
            
            if showError {
                Text("Email is required")
                    .foregroundStyle(.red)
                    .accessibilityFocused($isErrorFocused)
            }
            
            Button("Submit") {
                if email.isEmpty {
                    showError = true
                    isErrorFocused = true  // Move VoiceOver to error
                }
            }
        }
    }
}
```

## Announcements

For ephemeral feedback like toasts, announce to VoiceOver:

```swift
// Before: toast appears briefly, VoiceOver users miss it
Text(message ?? "")
    .opacity(opacity)

// After: announce the message
func showToast(_ message: String) {
    if #available(iOS 17, *) {
        var announcement = AttributedString(message)
        announcement.accessibilitySpeechAnnouncementPriority = .high
        AccessibilityNotification.Announcement(announcement).post()
    } else {
        UIAccessibility.post(notification: .announcement, argument: message)
    }
}
```

**Note:** Toasts are challenging for accessibility. Consider persistent, and/or in-line, feedback alternatives when possible.

## Modal Dialogs

```swift
.accessibilityAddTraits(.isModal)
```

## Hide from VoiceOver

```swift
Image(decorative: "divider")

// or:
Image("divider")
    .accessibilityHidden(true)
```

## Custom Rotor

```swift
.accessibilityRotor("Headings") {
    ForEach(headings, id: \.id) { heading in
        AccessibilityRotorEntry(heading.title, id: heading.id)
    }
}
```

## Example: Grouped Card with Actions

This example combines grouping, traits, and custom actions in one card component:

```swift
struct CardView: View {
    let item: Item
    @Binding var isFavorite: Bool
    var onDelete: () -> Void
    
    var body: some View {
        VStack {
            Image(item.imageName)
                .accessibilityIgnoresInvertColors()
            Text(item.title)
            Text(item.subtitle)
        }
        .accessibilityElement(children: .combine)
        .accessibilityAddTraits(.isButton)
        .accessibilityAction(named: isFavorite ? "Remove from favorites" : "Add to favorites") {
            isFavorite.toggle()
        }
        .accessibilityAction(named: "Delete") {
            onDelete()
        }
    }
}
```

## Sources

- [Accessibility Up To 11 — #365DaysIOSAccessibility](https://accessibilityupto11.com/365-days-ios-accessibility/)
- [Accessibility Up To 11 — Blog](https://accessibilityupto11.com/blog/)
- [From Zero to Accessible](https://github.com/dadederk/fromZeroToAccessible) (Rob Whitaker and Daniel Devesa Derksen-Staats)
