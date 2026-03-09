# VoiceOver — UIKit

UIKit implementation for VoiceOver accessibility.

For core concepts, see `voiceover.md`.

## Contents

- [Labels](#labels)
- [Values](#values)
- [Hints](#hints)
- [Traits](#traits)
- [Adjustable Controls](#adjustable-controls)
- [Grouping](#grouping)
- [Custom Actions](#custom-actions)
- [Accessibility Custom Content](#accessibility-custom-content)
- [Gestures](#gestures)
- [Notifications](#notifications)
- [Modal Views](#modal-views)
- [Advanced APIs](#accessibility-frame)
- [Examples](#example-complete-table-cell)

## Labels

### Basic label

```swift
closeButton.accessibilityLabel = "Close"
```

### Attributed labels for pronunciation

Switch languages mid-label:
Use whenever we know the language. Some use-case examples are: language learning apps, lyrics of songs in different languages...

```swift
let label = NSMutableAttributedString(
    string: "¡Hola! ",
    attributes: [.accessibilitySpeechLanguage: "es-ES"]
)
label.append(NSAttributedString(string: "means Hello!"))
view.accessibilityAttributedLabel = label
```

IPA pronunciation:
Use as last resort. Users tend to be used to quirk pronunciations from the screen reader. Use-case example: correct the pronunciation for your brand's name

```swift
let label = NSMutableAttributedString(string: "Watch ")
label.append(NSAttributedString(
    string: "live",
    attributes: [.accessibilitySpeechIPANotation: "laɪv"]
))
view.accessibilityAttributedLabel = label
```

Spell out:
Use-case examples: codes, phone numbers... where it makes sense to announce content character by character. Some developers separate characters with spaces to achieve this (C O O L E E A R 5 2 3). This is an anti-pattern, as it can create unnecessary verbosity for braille readers.

```swift
let label = NSAttributedString(
    string: "COOLEEAR523",
    attributes: [.accessibilitySpeechSpellOut: true]
)
view.accessibilityAttributedLabel = label
```

Read punctuation:
Use-case examples: announce all characters of a piece of programming code, examples in a grammar app...

```swift
let label = NSAttributedString(
    string: "let greeting = \"Hello, world!\"; print(greeting)",
    attributes: [.accessibilitySpeechPunctuation: true]
)
view.accessibilityAttributedLabel = label
```

### Labels: key rules

- **Context without verbosity**: give enough context to understand the element, but avoid repeating what VoiceOver already says (e.g., traits). Don't add "button" to a button's label — VoiceOver already says "Button".
- **Localize**: use `NSLocalizedString` so labels work across all supported languages.
- **Update on state change**: if an element changes meaning (e.g., a Follow button becomes Unfollow), update the label immediately.
- **Avoid redundancy in the same view**: if VoiceOver will read a heading and the button below already refers to that context, you don't need to repeat it.

```swift
// State change: label must reflect current state
followButton.accessibilityLabel = isFollowing
    ? NSLocalizedString("Unfollow", comment: "")
    : NSLocalizedString("Follow", comment: "")

// Context-sensitive add button — generic vs specific
addButton.accessibilityLabel = NSLocalizedString("Add song", comment: "")
// Not just "Add" when it's ambiguous

// Avoid redundancy in a player — context is already clear
playButton.accessibilityLabel = "Play"     // ✅
playButton.accessibilityLabel = "Play song" // ❌ redundant in a music player
```

### Formatters for readable labels

When abbreviating, use built-in formatters and styles, if possible

```swift
// Duration
let formatter = DateComponentsFormatter()
formatter.unitsStyle = .spellOut
formatter.allowedUnits = [.hour, .minute]
durationLabel.accessibilityLabel = formatter.string(from: 3660)
// "1 hour, 1 minute"

// Measurements
let measurement = Measurement<UnitLength>(value: 42, unit: .kilometers)
let measureFormatter = MeasurementFormatter()
measureFormatter.unitStyle = .long
distanceLabel.accessibilityLabel = measureFormatter.string(from: measurement)
// "42 kilometers"
```

### Compose complex labels

Useful when composing a label from multiple pieces of complex information.

```swift
let components: [String?] = [title, verifiedBadge, date, text, altText]
let accessibilityLabel = components
    .compactMap { $0 }
    .filter { !$0.isEmpty }
    .joined(separator: ", ")
cell.accessibilityLabel = accessibilityLabel
```

### Accessibility value patterns

When a component has a variable state, use value instead of label (see next section). Example: badge shows a count, use label + value:

```swift
// Before: badge label changes or is just a number
numberOfItemsLabel.accessibilityLabel = "\(count)"

// After: consistent label with dynamic value
orderButton.accessibilityLabel = "Cart"
orderButton.accessibilityValue = count > 0 ? "\(count) items" : nil
numberOfItemsLabel.isAccessibilityElement = false  // Hide the badge
```

## Values

The default value for `UISlider` is a percentage. That's fine for a volume control, but not for a price range or a playback progress bar. Always use the format that's most meaningful to the user:

```swift
// Volume: percentage makes sense
slider.accessibilityValue = "\(Int(slider.value * 100)) percent"

// Price range: say the actual price, not a percentage
let formatter = NumberFormatter()
formatter.numberStyle = .currency
priceSlider.accessibilityValue = formatter.string(from: NSNumber(value: priceSlider.value))
// VoiceOver says "£450,000" not "50%"

// Playback progress: say minutes and seconds
let formatter = DateComponentsFormatter()
formatter.unitsStyle = .full
formatter.allowedUnits = [.minute, .second]
playbackSlider.accessibilityValue = formatter.string(from: TimeInterval(playbackSlider.value))
// VoiceOver says "2 minutes, 30 seconds" not "25%"

toggle.accessibilityValue = toggle.isOn ? "On" : "Off"
```

Update value whenever state changes:

```swift
var rating: Int = 0 {
    didSet {
        accessibilityValue = "\(rating + 1) thumbs up"
    }
}
```

For reusable components, you can also override the value:

```swift
final class RatingView: UIView {
    var rating: Int = 0

    override var accessibilityValue: String? {
        get { "\(rating + 1) thumbs up" }
        set {}
    }
}
```

## Hints

Hints are **optional** and are read after a pause — after the label, trait, and value. Experienced users can skip them, so hints won't slow down your power users. Use them when an element's purpose or interaction is non-obvious; skip them when the label already tells the full story.

**Rules of thumb:**
- Start with a verb — describe what *happens*, not what the element *is*
- Don't repeat the label or trait in the hint
- Localize just like labels
- Keep them concise

```swift
// Good: starts with a verb, adds context
draggableHandle.accessibilityHint = NSLocalizedString(
    "Double tap and hold, wait for the sound, then drag to rearrange.",
    comment: ""
)

// Good: explains a non-obvious interaction
miniPlayer.accessibilityHint = NSLocalizedString(
    "Double tap to expand to full screen.",
    comment: ""
)

// Custom control needing context
accessibilityHint = "Rates your drink from 1 to 5 thumbs up"

// Standard control with extra context
deleteButton.accessibilityHint = "Removes the item from your list"
```

Avoid hints that just restate the obvious:

```swift
// Bad: VoiceOver already says "Button"
playButton.accessibilityHint = "Tap to play"  // ❌

// Good: adds genuine context
playButton.accessibilityHint = "Plays the episode from the beginning" // ✅
```

## Traits

### Header trait

Mark section headings for Rotor navigation:

```swift
sectionHeader.accessibilityTraits.insert(.header)
```

### Selected trait

For custom picker options, toggle states, and segmented controls:

```swift
var isToggled: Bool = false {
    didSet {
        if isToggled {
            selectionIconImageView.image = UIImage(systemName: "checkmark.circle.fill")
            accessibilityTraits.insert(.selected)
        } else {
            selectionIconImageView.image = UIImage(systemName: "circle")
            accessibilityTraits.remove(.selected)
        }
    }
}
```

### Common operations

Prefer adding/removing traits, to fully re-assigning.

```swift
// Single trait
sectionTitle.accessibilityTraits = .header

// Multiple traits
cell.accessibilityTraits = [.button, .selected]

// Add trait
button.accessibilityTraits.insert(.notEnabled)

// Remove trait
button.accessibilityTraits.remove(.selected)
```

## Adjustable Controls

For custom sliders, steppers, pickers — group and make adjustable:

```swift
// Before: separate buttons that are hard to understand
class ExtraShotsView: UIView {
    @IBOutlet private weak var removeShotButton: UIButton!
    @IBOutlet private weak var addShotButton: UIButton!
    @IBOutlet private weak var numberOfShotsLabel: UILabel!
}

// After: single adjustable control
class ExtraShotsView: UIView {
    private var numberOfShots = 0
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        isAccessibilityElement = true
        accessibilityLabel = "Extra shots"
        accessibilityTraits.insert(.adjustable)
    }

    override func accessibilityIncrement() {
        guard numberOfShots < 4 else { return }
        numberOfShots += 1
        updateAccessibilityValue()
    }

    override func accessibilityDecrement() {
        guard numberOfShots > 0 else { return }
        numberOfShots -= 1
        updateAccessibilityValue()
    }
    
    private func updateAccessibilityValue() {
        accessibilityValue = "\(numberOfShots) shots"
    }
}
```

### Rating control example

```swift
class RaterView: UIView {
    private var maxRate: UInt = 5
    
    // Scale icons with Dynamic Type
    private var icon = UIImage(
        systemName: "hand.thumbsup",
        withConfiguration: UIImage.SymbolConfiguration(textStyle: .body)
    )
    
    var rating: Int = 0 {
        didSet {
            accessibilityValue = "\(rating + 1) thumbs up"
        }
    }

    private func setUp() {
        // Group as single element
        isAccessibilityElement = true
        accessibilityLabel = "Rating"
        accessibilityTraits = .adjustable
        accessibilityHint = "Rates your drink from 1 to 5 thumbs up"
    }

    override func accessibilityIncrement() {
        guard rating < maxRate - 1 else { return }
        pressButton(at: rating + 1)
    }

    override func accessibilityDecrement() {
        guard rating > 0 else { return }
        pressButton(at: rating - 1)
    }
}
```

## Grouping

### Make container the accessibility element

When a cell has multiple elements, group for easier navigation. Make sure that interactive elements in the group are individually accessible elsewhere, for example, a details screen.

```swift
// Before: VoiceOver reads label, price, button separately
final class DrinkTableViewCell: UITableViewCell {
    @IBOutlet private weak var drinkNameLabel: UILabel!
    @IBOutlet private weak var priceLabel: UILabel!
    @IBOutlet private weak var buyButton: UIButton!
}

// After: single grouped element
final class DrinkTableViewCell: UITableViewCell {
    override func awakeFromNib() {
        super.awakeFromNib()
        
        isAccessibilityElement = true
        accessibilityTraits.insert(.button)
    }
    
    override var accessibilityLabel: String? {
        get {
            [drinkNameLabel.accessibilityLabel, priceLabel.accessibilityLabel]
                .compactMap { $0 }
                .joined(separator: ", ")
        }
        set {}
    }
}
```

### Group traversal without merging

Use this when you want VoiceOver to traverse a container's children as a group before moving to elements outside the group. This does not merge children into one element.

```swift
containerView.shouldGroupAccessibilityChildren = true
```

### Explicit element order

```swift
view.accessibilityElements = [playButton, shareButton, moreOptionsButton]
```

### Pair related labels (columns)

When visual layout uses columns (e.g., label + value), group pairs so VoiceOver reads them together:

```swift
// Preferred when you have real subviews/rows:
followersRow.isAccessibilityElement = true
followersRow.accessibilityLabel = "Followers"
followersRow.accessibilityValue = "550"

followingRow.isAccessibilityElement = true
followingRow.accessibilityLabel = "Following"
followingRow.accessibilityValue = "340"

postsRow.isAccessibilityElement = true
postsRow.accessibilityLabel = "Posts"
postsRow.accessibilityValue = "750"

statsStackView.shouldGroupAccessibilityChildren = true

// For custom-drawn content, build explicit accessibility elements:
let followersElement = UIAccessibilityElement(accessibilityContainer: statsView)
followersElement.accessibilityLabel = "Followers"
followersElement.accessibilityValue = "550"
followersElement.accessibilityFrameInContainerSpace = followersFrame

let followingElement = UIAccessibilityElement(accessibilityContainer: statsView)
followingElement.accessibilityLabel = "Following"
followingElement.accessibilityValue = "340"
followingElement.accessibilityFrameInContainerSpace = followingFrame

let postsElement = UIAccessibilityElement(accessibilityContainer: statsView)
postsElement.accessibilityLabel = "Posts"
postsElement.accessibilityValue = "750"
postsElement.accessibilityFrameInContainerSpace = postsFrame

statsView.accessibilityElements = [followersElement, followingElement, postsElement]
```

### Hide from VoiceOver

```swift
// Single element
decorativeImage.isAccessibilityElement = false

// Entire subtree
backgroundView.accessibilityElementsHidden = true
```

## Custom Actions

Expose hidden or secondary actions:

```swift
// Before: buy button inside cell can't be reached directly when cell is grouped

// After: exposed as custom action
override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
    get {
        [
            UIAccessibilityCustomAction(
                name: "Add to cart",
                image: UIImage(systemName: "cart.badge.plus")
            ) { [weak self] _ in
                self?.buyDrink()
                return true
            }
        ]
    }
    set {}
}
```

### Multiple actions with images

Images appear in Switch Control menus (iOS 14+) with [`UIAccessibilityCustomAction.init(name:image:actionHandler:)`](https://developer.apple.com/documentation/uikit/uiaccessibilitycustomaction/init(name:image:actionhandler:)):

```swift
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Delete") { _ in
        self.deleteItem()
        return true
    },
    UIAccessibilityCustomAction(
        name: "Share",
        image: UIImage(systemName: "square.and.arrow.up")
    ) { _ in
        self.shareItem()
        return true
    }
]
```

## Accessibility Custom Content

Use this for supplementary information in data-rich UI (for example, charts, financial cards, advanced stats).

Keep this content optional and concise: users can configure VoiceOver verbosity so hints about extra content may be reduced or disabled.
Whenever possible, make the same information independently accessible elsewhere too (for example, in a details screen).

```swift
let trend = UIAccessibilityCustomContent(
    label: "Trend",
    value: "Upward over last 30 days"
)
trend.importance = .default

let confidence = UIAccessibilityCustomContent(
    label: "Confidence",
    value: "High"
)
confidence.importance = .high

summaryCard.accessibilityCustomContent = [trend, confidence]
```

Use `.high` only for data that is essential in context.

## Gestures

### Double-tap activation

Only necessary if the view doesn't support this already, for example a custom component inside a view with a gesture recognizer to activate it

```swift
override func accessibilityActivate() -> Bool {
    performMainAction()
    return true
}
```

### Activation point

By default, VoiceOver activates the center of the focused element. For custom-drawn controls, or unusual hit regions, set `accessibilityActivationPoint` in screen coordinates:

```swift
let pointInView = CGPoint(x: knobFrame.midX, y: knobFrame.midY)
customControl.accessibilityActivationPoint = customControl.convert(pointInView, to: nil)
```

### Magic Tap

For the main functionality of a screen. For power users. Examples: start/stop a timer, play/pause a game...

```swift
override func accessibilityPerformMagicTap() -> Bool {
    togglePlayPause()
    return true
}
```

### Escape

Gesture for power users, that means: Back. Only needed for custom modals and overlays. 

```swift
override func accessibilityPerformEscape() -> Bool {
    dismiss(animated: true)
    return true
}
```

### Direct touch for real-time interactions

For fast, continuous interactions (for example, music apps, drawing canvases, and some game controls), direct touch can reduce friction:

```swift
joystickView.isAccessibilityElement = true
joystickView.accessibilityTraits.insert(.allowsDirectInteraction)
```

Use this sparingly. Prefer regular VoiceOver navigation unless the interaction truly depends on real-time touch movement.

## Detecting Assistive Technologies

Use these checks sparingly. Most accessibility improvements should be unconditional — don't reserve good experiences only for users with a specific technology enabled. That said, there are legitimate use cases:

- Adapting UI elements that are normally ephemeral to be persistent for VoiceOver users
- Optimizing expensive label/action building behind a guard (e.g., in large lists)
- Coordinating behavior specific to one assistive technology

```swift
// Check state at a point in time
if UIAccessibility.isVoiceOverRunning {
    // e.g., keep tooltip visible
}

if UIAccessibility.isSwitchControlRunning {
    // e.g., also build custom actions for Switch Control
}
```

Always also observe changes — the user may enable or disable assistive technology while using your app:

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(voiceOverDidChange),
    name: UIAccessibility.voiceOverStatusDidChangeNotification,
    object: nil
)

@objc private func voiceOverDidChange() {
    if UIAccessibility.isVoiceOverRunning {
        // Update UI
    }
}
```

> **Ask yourself first:** Can I just make this element always accessible, so all users benefit? Custom actions, proper labels, and persistent feedback are good for everyone.

## Notifications

### Move focus

```swift
// Major screen change (plays sound)
UIAccessibility.post(notification: .screenChanged, argument: newView)

// Content updated (no sound)
UIAccessibility.post(notification: .layoutChanged, argument: errorLabel)
```

### Announce messages

For ephemeral feedback. Example: toasts

```swift
// Before: toast appears briefly, VoiceOver users miss it

// After: announce with high priority
func present(inView view: UIView) {
    guard let text = toastTitleLabel.text else { return }
    
    UIView.animate(withDuration: 0.2) { self.alpha = 1.0 } completion: { _ in
        if #available(iOS 17, *) {
            var announcement = AttributedString(text)
            announcement.accessibilitySpeechAnnouncementPriority = .high
            AccessibilityNotification.Announcement(announcement).post()
        } else {
            UIAccessibility.post(notification: .announcement, argument: text)
        }
        
        UIView.animate(withDuration: 0.2, delay: 3.0) { self.alpha = 0.0 }
    }
}
```

**Note:** Toasts are challenging for accessibility. Consider in-line persistent (or dismissable via user interaction) feedback alternatives when possible.

### Queue announcements

Set `.accessibilitySpeechQueueAnnouncement` to `true` to enqueue follow-up announcements. When false, the ongoing announcement gets interrupted.
```swift
let first = NSAttributedString(
    string: "Downloaded file A.jpeg",
    attributes: [.accessibilitySpeechQueueAnnouncement: true]
)
let second = NSAttributedString(
    string: "Downloaded file B.jpeg",
    attributes: [.accessibilitySpeechQueueAnnouncement: true]
)

UIAccessibility.post(notification: .announcement, argument: first)
UIAccessibility.post(notification: .announcement, argument: second)
```

### Page scrolled

If you scroll on the user’s behalf, notify VoiceOver:

```swift
scrollView.setContentOffset(newOffset, animated: true)
UIAccessibility.post(notification: .pageScrolled, argument: scrollView)
```

## Modal Views

Only needed for custom modals or overlays that should block interaction with the elements underneath.

```swift
alertView.accessibilityViewIsModal = true
UIAccessibility.post(notification: .screenChanged, argument: alertView)
```

## Smart Invert

Prevent photos and meaningful images from inverting:

```swift
drinkImageView.accessibilityIgnoresInvertColors = true
```

## Accessibility Frame

Expand focus area:

```swift
let expandedFrame = button.bounds.insetBy(dx: -20, dy: -20)
button.accessibilityFrame = button.convert(expandedFrame, to: nil)
```

## UIAccessibilityElement

For custom drawing or grouping across view hierarchies:

```swift
let element = UIAccessibilityElement(accessibilityContainer: chartView)
element.accessibilityLabel = "Sales chart"
element.accessibilityFrame = chartView.convert(chartRect, to: nil)
element.accessibilityTraits = .image
chartView.accessibilityElements = [element]
```

## Container Types

```swift
tabBar.accessibilityContainerType = .semanticGroup
tabBar.accessibilityLabel = "Tab bar"
```

## Example: Complete Table Cell

```swift
final class DrinkTableViewCell: UITableViewCell {
    @IBOutlet private weak var outerStackView: UIStackView!
    @IBOutlet private weak var drinkImageView: UIImageView!
    @IBOutlet private weak var drinkNameLabel: UILabel!
    @IBOutlet private weak var priceLabel: UILabel!
    
    private var drink: Drink?

    override func awakeFromNib() {
        super.awakeFromNib()
        
        // Dynamic Type
        drinkNameLabel.font = .preferredFont(forTextStyle: .body)
        priceLabel.font = .preferredFont(forTextStyle: .body)
        
        // Semantic colors for contrast
        priceLabel.textColor = .secondaryLabel
        
        // Group cell
        isAccessibilityElement = true
        accessibilityTraits.insert(.button)
        
        // Smart Invert
        drinkImageView.accessibilityIgnoresInvertColors = true
        
        updateLayout()
    }
    
    override var accessibilityLabel: String? {
        get {
            [drinkNameLabel.accessibilityLabel, priceLabel.accessibilityLabel]
                .compactMap { $0 }
                .joined(separator: ", ")
        }
        set {}
    }
    
    override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
        get {
            [UIAccessibilityCustomAction(name: "Add to cart", target: self, selector: #selector(buyDrink))]
        }
        set {}
    }
    
    override func traitCollectionDidChange(_ previous: UITraitCollection?) {
        if previous?.preferredContentSizeCategory != traitCollection.preferredContentSizeCategory {
            updateLayout()
        }
    }
    
    private func updateLayout() {
        if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
            outerStackView.axis = .vertical
            drinkNameLabel.numberOfLines = 0
        } else {
            outerStackView.axis = .horizontal
            drinkNameLabel.numberOfLines = 1
        }
    }
}
```

## Example: Toggle Cell (UISwitch + UITableViewCell)

A very common pattern: a settings row with a title, optional subtitle, and a toggle. The classic mistake is leaving both the cell and the switch as separate accessible elements, so VoiceOver reads each one individually and the user has to swipe twice to interact.

Make the cell itself the single accessible element that mirrors the switch's behavior:

```swift
// After: cell acts as a single accessible toggle
final class SwitchTableViewCell: UITableViewCell {
    @IBOutlet private weak var titleLabel: UILabel!
    @IBOutlet private weak var subtitleLabel: UILabel!
    @IBOutlet private weak var settingSwitch: UISwitch!

    override func awakeFromNib() {
        super.awakeFromNib()
        // The cell is the accessible element; the switch is decorative
        isAccessibilityElement = true
        settingSwitch.isAccessibilityElement = false
    }

    // Combine title + subtitle for context
    override var accessibilityLabel: String? {
        get {
            [titleLabel.text, subtitleLabel.text]
                .compactMap { $0 }
                .filter { !$0.isEmpty }
                .joined(separator: ". ")
        }
        set {}
    }

    // Mirror the switch's traits ("switch button")
    override var accessibilityTraits: UIAccessibilityTraits {
        get { settingSwitch.accessibilityTraits }
        set {}
    }

    // Mirror the switch's value ("on" / "off")
    override var accessibilityValue: String? {
        get { settingSwitch.accessibilityValue }
        set {}
    }

    // Double-tap toggles the switch
    override func accessibilityActivate() -> Bool {
        settingSwitch.isOn.toggle()
        settingSwitch.sendActions(for: .valueChanged)
        return true
    }
}
```

VoiceOver now reads: *"Enable notifications. On. Switch."* — one element, clear state, immediately actionable.

> **Common mistake:** Leaving both the cell and the switch as accessible elements means VoiceOver reads the label twice and the user has to navigate past redundant elements.

## Example: Form Error (Focus, Not Announcement)

Use `.layoutChanged` when the update is tied to a visible element that should receive focus (such as an inline form error).

```swift
func showError(_ message: String) {
    errorLabel.text = message
    errorLabel.isHidden = false
    UIAccessibility.post(notification: .layoutChanged, argument: errorLabel)
}
```

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://github.com/Apress/developing-accessible-iOS-apps
- https://github.com/dadederk/fromZeroToAccessible (Daniel Devesa Derksen-Staats and Rob Whitaker)
