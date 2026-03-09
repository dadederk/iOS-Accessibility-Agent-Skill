# Dynamic Type — UIKit

UIKit implementation for Dynamic Type and scalable layouts.

For core concepts, see `dynamic-type.md`.

## Contents

- [Text Styles](#text-styles)
- [Custom Fonts](#custom-fonts)
- [Layout Adaptation](#layout-adaptation)
- [Scale Non-Text Elements](#scale-non-text-elements)
- [Large Content Viewer](#large-content-viewer)
- [Web Content](#web-content)
- [Examples](#example-adaptive-card)

## Text Styles

```swift
label.font = UIFont.preferredFont(forTextStyle: .body)
label.adjustsFontForContentSizeCategory = true
```

**Important:** Set `adjustsFontForContentSizeCategory = true` for automatic updates when the user changes text size.
For exact point sizes by text style and content size category, see Apple HIG: [iOS/iPadOS Dynamic Type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-Dynamic-Type-sizes).

## Custom Fonts

Use `UIFontMetrics` to scale custom fonts:

```swift
let customFont = UIFont(name: "Avenir-Medium", size: 17)!
let fontMetrics = UIFontMetrics(forTextStyle: .body)
label.font = fontMetrics.scaledFont(for: customFont)
label.adjustsFontForContentSizeCategory = true
```

## Multiline Labels

Allow text to wrap:

```swift
label.numberOfLines = 0
```

Avoid fixed height constraints.

If you must cap lines for product reasons, relax the cap at larger sizes (for example, double or triple it for accessibility categories):

```swift
func updateLineLimit(for category: UIContentSizeCategory) {
    switch category {
    case .accessibilityExtraExtraExtraLarge:
        titleLabel.numberOfLines = 6   // triple from 2
    case .accessibilityMedium, .accessibilityLarge, .accessibilityExtraLarge, .accessibilityExtraExtraLarge:
        titleLabel.numberOfLines = 4   // double from 2
    default:
        titleLabel.numberOfLines = 2
    }
}
```

## Detect Accessibility Sizes

For the larger accessibility categories and their reference sizes, see Apple HIG: [iOS/iPadOS larger accessibility type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-larger-accessibility-type-sizes).

```swift
if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
    // Accessibility size (one of the 5 largest)
}
```

### Compare size categories

```swift
if traitCollection.preferredContentSizeCategory >= .accessibilityLarge {
    // Large or larger
}
```

## Respond to Size Changes

### traitCollectionDidChange

```swift
override func traitCollectionDidChange(_ previous: UITraitCollection?) {
    super.traitCollectionDidChange(previous)
    if traitCollection.preferredContentSizeCategory != previous?.preferredContentSizeCategory {
        updateLayout()
    }
}
```

### Notification

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleSizeChange),
    name: UIContentSizeCategory.didChangeNotification,
    object: nil
)
```

## Layout Adaptation

At larger text sizes, switch from horizontal to vertical layouts so text can flow across the full screen width.

### Flip stack axis

```swift
func updateLayout() {
    stackView.axis = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
        ? .vertical
        : .horizontal
}
```

### Listen for changes

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    if previousTraitCollection?.preferredContentSizeCategory != traitCollection.preferredContentSizeCategory {
        updateLayout()
    }
}
```

### Example: Table cell

```swift
final class DrinkTableViewCell: UITableViewCell {
    @IBOutlet private weak var outerStackView: UIStackView!
    @IBOutlet private weak var drinkNameLabel: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        // Dynamic Type fonts
        drinkNameLabel.font = .preferredFont(forTextStyle: .body)
        drinkNameLabel.adjustsFontForContentSizeCategory = true
        
        updateLayout()
    }
    
    override func traitCollectionDidChange(_ previous: UITraitCollection?) {
        if previous?.preferredContentSizeCategory != traitCollection.preferredContentSizeCategory {
            updateLayout()
        }
    }
    
    private func updateLayout() {
        if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
            outerStackView.axis = .vertical
            outerStackView.alignment = .leading
            drinkNameLabel.numberOfLines = 0  // Unlimited lines
        } else {
            outerStackView.axis = .horizontal
            outerStackView.alignment = .center
            drinkNameLabel.numberOfLines = 1
        }
    }
}
```

### Example: Stepper control

```swift
class ExtraShotsView: UIView {
    @IBOutlet private weak var mainStackView: UIStackView!
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
            mainStackView.axis = .vertical
        } else {
            mainStackView.axis = .horizontal
        }
    }
}
```

### Fallback: Scroll view for oversized content

If large text still doesn’t fit, embed the screen in a scroll view for accessibility sizes:

```swift
func updateLayout() {
    if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        contentScrollView.isScrollEnabled = true
    } else {
        contentScrollView.isScrollEnabled = false
    }
}
```

### Switch to single column

```swift
let columns = traitCollection.preferredContentSizeCategory.isAccessibilityCategory ? 1 : 2
```

### Constraint sets

Create separate constraint sets and activate based on size:

```swift
var defaultConstraints: [NSLayoutConstraint] = []
var accessibilityConstraints: [NSLayoutConstraint] = []

func updateConstraints() {
    if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        NSLayoutConstraint.deactivate(defaultConstraints)
        NSLayoutConstraint.activate(accessibilityConstraints)
    } else {
        NSLayoutConstraint.deactivate(accessibilityConstraints)
        NSLayoutConstraint.activate(defaultConstraints)
    }
}
```

## Readable Content Guide

For long-form text, constrain to `readableContentGuide` for comfortable line length:

```swift
textView.leadingAnchor.constraint(equalTo: view.readableContentGuide.leadingAnchor).isActive = true
textView.trailingAnchor.constraint(equalTo: view.readableContentGuide.trailingAnchor).isActive = true
```

## Baseline Spacing

Use system baseline spacing instead of fixed constants:

```swift
subtitleLabel.firstBaselineAnchor.constraint(
    equalToSystemSpacingBelow: titleLabel.lastBaselineAnchor,
    multiplier: 1.0
).isActive = true
```

## Scale Non-Text Elements

Use `UIFontMetrics.scaledValue(for:)` for icons and other UI:

```swift
let baseHeight: CGFloat = 20
let scaledHeight = UIFontMetrics.default.scaledValue(for: baseHeight)
progressView.heightAnchor.constraint(equalToConstant: scaledHeight).isActive = true
```

## Scale Images

```swift
imageView.adjustsImageSizeForAccessibilityContentSizeCategory = true
```

Use PDF/vector assets with **Preserve Vector Data** enabled.

### Prefer SF Symbols with text styles

SF Symbols scale like fonts and can be tied to a text style:

```swift
iconImageView.image = UIImage(systemName: "xmark.octagon")
iconImageView.preferredSymbolConfiguration = UIImage.SymbolConfiguration(textStyle: .body)
```

This keeps icon size in sync with the adjacent text.

## Large Content Viewer

For elements that don't scale (bars, compact controls), let users tap-and-hold to see enlarged content.

Use this when the UI **cannot scale with Dynamic Type** (navigation bars, tab bars, toolbars). If content can scale, prefer Dynamic Type.

### Tab bar assets for large preview

If you can’t provide vector PDFs, use a larger raster image for the preview:

```swift
// Provide a higher-resolution image for the large preview
tabBarItem.largeContentSizeImage = UIImage(named: "tab-large")
```

### Custom bar elements

Custom views need explicit titles/images:

```swift
customBarView.addInteraction(UILargeContentViewerInteraction())
customBarButton.showsLargeContentViewer = true
customBarTabView.showsLargeContentViewer = true
customBarTabView.largeContentTitle = "Videos"
customBarTabView.largeContentImage = UIImage(named: "play")
```

### Protocol implementation

```swift
class CustomTabItem: UIView, UILargeContentViewerItem {
    var showsLargeContentViewer: Bool { true }
    var largeContentTitle: String? { "Home" }
    var largeContentImage: UIImage? { UIImage(systemName: "house") }
}

// Add interaction to the container
tabBar.addInteraction(UILargeContentViewerInteraction())
```

### Example: Cart button with badge

```swift
class OrderButtonView: UIView {
    @IBOutlet private weak var orderButton: UIButton!
    
    private var numberOfItems: UInt = 0 {
        didSet {
            // Update Large Content Viewer with current count
            orderButton.largeContentTitle = "Cart, \(numberOfItems) items"
        }
    }
    
    func enableLargeContentViewer() {
        orderButton.showsLargeContentViewer = true
        orderButton.addInteraction(UILargeContentViewerInteraction())
    }
}
```

Users with Larger Accessibility Sizes can tap-and-hold to see the button's content displayed larger in the center of the screen.

## Web Content

In `WKWebView`, use Apple system fonts in CSS — they respect Dynamic Type automatically on Apple devices. Always include fallback fonts for cross-platform HTML:

```css
body {
    font: -apple-system-body;
}
h1 {
    font: -apple-system-headline;
    color: darkblue;
}
.footnote {
    font: -apple-system-footnote;
    color: gray;
}
```

The web content won't resize automatically when the user changes their text size preference. Listen for the `UIContentSizeCategory.didChangeNotification` and reload the page:

```swift
class WebViewController: UIViewController {
    @IBOutlet weak var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()
        loadContent()
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(contentSizeCategoryDidChange),
            name: UIContentSizeCategory.didChangeNotification,
            object: nil
        )
    }

    private func loadContent() {
        guard let baseURL = Bundle.main.resourceURL else { return }
        let fileURL = baseURL.appendingPathComponent("content.html")
        webView.loadFileURL(fileURL, allowingReadAccessTo: fileURL)
    }

    @objc private func contentSizeCategoryDidChange() {
        webView.reload()
    }
}
```

> A full list of Apple CSS font names is documented at webkit.org/blog/3709/using-the-system-font-in-web-content/.

## Interface Builder

Configure Dynamic Type in Interface Builder:
1. Select the label
2. In Attributes Inspector, choose a text style for Font
3. Check "Automatically Adjusts Font"

## Example: Adaptive Card

```swift
class CardView: UIView {
    let stackView = UIStackView()
    let imageView = UIImageView()
    let titleLabel = UILabel()
    let subtitleLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        
        titleLabel.font = UIFont.preferredFont(forTextStyle: .headline)
        titleLabel.adjustsFontForContentSizeCategory = true
        titleLabel.numberOfLines = 0
        
        subtitleLabel.font = UIFont.preferredFont(forTextStyle: .subheadline)
        subtitleLabel.adjustsFontForContentSizeCategory = true
        subtitleLabel.numberOfLines = 0
        
        stackView.axis = .horizontal
        stackView.spacing = 12
        stackView.addArrangedSubview(imageView)
        stackView.addArrangedSubview(titleLabel)
        stackView.addArrangedSubview(subtitleLabel)
        
        addSubview(stackView)
        updateLayout()
    }
    
    override func traitCollectionDidChange(_ previous: UITraitCollection?) {
        super.traitCollectionDidChange(previous)
        updateLayout()
    }
    
    func updateLayout() {
        stackView.axis = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
            ? .vertical
            : .horizontal
    }
}
```


## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://github.com/Apress/developing-accessible-iOS-apps
- https://github.com/dadederk/fromZeroToAccessible (Daniel Devesa Derksen-Staats and Rob Whitaker)
