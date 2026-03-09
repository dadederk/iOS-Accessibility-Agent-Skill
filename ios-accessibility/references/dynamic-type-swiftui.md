# Dynamic Type — SwiftUI

SwiftUI implementation for Dynamic Type and scalable layouts.

For core concepts, see `dynamic-type.md`.

## Contents

- [Text Styles](#text-styles)
- [Layout Adaptation](#layout-adaptation)
- [Scale Non-Text Elements](#scale-non-text-elements)
- [Large Content Viewer](#large-content-viewer)
- [Constrained Dynamic Type](#constrained-dynamic-type)
- [Testing](#testing)
- [Examples](#example-adaptive-card)

## Text Styles

SwiftUI automatically scales text with text styles:

```swift
Text("Hello")
    .font(.body)
```

No additional configuration needed — text scales automatically.
For exact point sizes by text style and content size category, see Apple HIG: [iOS/iPadOS Dynamic Type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-Dynamic-Type-sizes).

## All Text Styles

```swift
Text("Large Title").font(.largeTitle)
Text("Title").font(.title)
Text("Title 2").font(.title2)
Text("Title 3").font(.title3)
Text("Headline").font(.headline)
Text("Subheadline").font(.subheadline)
Text("Body").font(.body)
Text("Callout").font(.callout)
Text("Footnote").font(.footnote)
Text("Caption").font(.caption)
Text("Caption 2").font(.caption2)
```

## Custom Fonts

Scale custom fonts relative to a text style:

```swift
Text("Custom")
    .font(.custom("PressStart2P-Regular", size: 17, relativeTo: .body))
```

## Detect Accessibility Sizes

For the larger accessibility categories and their reference sizes, see Apple HIG: [iOS/iPadOS larger accessibility type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-larger-accessibility-type-sizes).

```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    if dynamicTypeSize.isAccessibilitySize {
        // Accessibility size (one of the 5 largest)
    }
}
```

### Compare sizes

```swift
if dynamicTypeSize >= .accessibility1 {
    // First accessibility size or larger
}
```

## Layout Adaptation

At larger text sizes, switch from horizontal to vertical layouts so text can flow across the full screen width.

### Flip stack axis

```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    let layout = dynamicTypeSize.isAccessibilitySize
        ? AnyLayout(VStackLayout())
        : AnyLayout(HStackLayout())
    
    layout {
        Image(systemName: "star")
        Text("Favorite")
    }
}
```

### Approach with Group

```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    Group {
        if dynamicTypeSize.isAccessibilitySize {
            VStack { content }
        } else {
            HStack { content }
        }
    }
}

@ViewBuilder
var content: some View {
    Image(systemName: "star")
    Text("Favorite")
}
```

### ViewThatFits

Let SwiftUI choose the best layout automatically:

```swift
ViewThatFits {
    HStack { content } // Try horizontal first
    VStack { content } // Fall back to vertical
}
```

Use this when layout should adapt to available space and actual content fit, not only to a specific `dynamicTypeSize` or size class threshold.
It is also a strong option when the fallback is more complex than simply switching `HStack` to `VStack` (for example, re-grouping content, changing hierarchy, or dropping non-essential decorative elements).

Great for local UI blocks that appear once (or only a few times) on screen.
For repeated rows in lists/grids, prefer a deterministic rule (for example, dynamic type threshold) so items do not switch layout inconsistently from one row to another.

### ScrollView for oversized content

Regardless of the complexity of the screen, consider wrapping the screen in a scroll view so there is always room for the content, even for accessibility sizes:

```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    Group {
        if dynamicTypeSize.isAccessibilitySize {
            ScrollView { content }
        } else {
            content
        }
    }
}
```

### Reusable AdaptiveStack component

Create a reusable component that considers both size class and Dynamic Type:

```swift
public struct AdaptiveStack<Content: View>: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @Environment(\.dynamicTypeSize) var dynamicTypeSize
    
    private let horizontalAlignment: HorizontalAlignment
    private let verticalAlignment: VerticalAlignment
    private let spacing: CGFloat?
    private let content: Content
    
    public init(
        horizontalAlignment: HorizontalAlignment = .center,
        verticalAlignment: VerticalAlignment = .center,
        spacing: CGFloat? = nil,
        @ViewBuilder content: () -> Content
    ) {
        self.horizontalAlignment = horizontalAlignment
        self.verticalAlignment = verticalAlignment
        self.spacing = spacing
        self.content = content()
    }
    
    public var body: some View {
        if dynamicTypeSize.isAccessibilitySize {
            VStack(alignment: horizontalAlignment, spacing: spacing) { content }
        } else {
            HStack(alignment: verticalAlignment, spacing: spacing) { content }
        }
    }
}
```

Usage:

```swift
AdaptiveStack(horizontalAlignment: .leading, spacing: 12) {
    Image(systemName: "info.circle")
        .font(.title2)
    VStack(alignment: .leading) {
        Text("About")
        Text("App information")
            .font(.caption)
    }
}
```

Extend with additional conditions like compact size class:

```swift
public enum AdaptiveCondition {
    case accessible           // Accessibility text sizes
    case compact              // Compact width
    case compactAccessible    // Both
}
```

Source: [SwiftUI Adaptive Stack Views - Use Your Loaf](https://useyourloaf.com/blog/swiftui-adaptive-stack-views/)

### Example: List row

```swift
struct DrinkTableRow: View {
    let drink: Drink
    @Environment(\.dynamicTypeSize.isAccessibilitySize) var accessibilitySize

    var body: some View {
        NavigationLink {
            DrinkDetail(drink: drink)
        } label: {
            // Adapt layout for large text
            if accessibilitySize {
                VStack(alignment: .leading) {
                    DrinkTableRowContent(drink: drink)
                }
            } else {
                HStack {
                    DrinkTableRowContent(drink: drink)
                }
            }
        }
    }
}
```

### Example: Stepper control

```swift
struct ExtraShotsView: View {
    @State private var shots = 0

    var body: some View {
        ViewThatFits {
            HStack {
                Image(systemName: "minus.circle")
                Text("\(shots) shots")
                Image(systemName: "plus.circle")
                Text("+ £\(shots * 0.50, format: .currency(code: "GBP"))")
            }
            VStack {
                HStack {
                    Image(systemName: "minus.circle")
                    Text("\(shots) shots")
                    Image(systemName: "plus.circle")
                }
                Text("+ £\(shots * 0.50, format: .currency(code: "GBP"))")
            }
        }
    }
}
```

## Multiline Text

SwiftUI `Text` wraps by default. For `TextField`:

```swift
TextField("Notes", text: $notes, axis: .vertical)
    .lineLimit(3...10)
```

If you must cap lines for product reasons, relax the cap at larger sizes (for example, double or triple it for accessibility sizes):

```swift
@Environment(\.dynamicTypeSize) private var dynamicTypeSize

private var titleLineLimit: Int {
    if dynamicTypeSize >= .accessibility3 { return 6 } // triple from 2
    if dynamicTypeSize.isAccessibilitySize { return 4 } // double from 2
    return 2
}

Text(title)
    .lineLimit(titleLineLimit)
```

## Scale Non-Text Elements

Use `ScaledMetric` for icons, spacing, and borders:

```swift
@ScaledMetric(relativeTo: .body) var iconSize: CGFloat = 24

Image(systemName: "star")
    .frame(width: iconSize, height: iconSize)
```

### Match scaling to text style

Use `relativeTo:` to tie scaling to specific text styles:

```swift
@ScaledMetric(relativeTo: .title3) private var borderWidth: CGFloat = 3.0
@ScaledMetric(relativeTo: .body) private var iconSize: CGFloat = 24
@ScaledMetric(relativeTo: .largeTitle) private var headerSpacing: CGFloat = 16
```

This ensures visual elements scale proportionally with their associated text.

### Example: Scaled border

```swift
struct TranscriptLineView: View {
    @ScaledMetric(relativeTo: .title3) private var baseBorderWidth: CGFloat = 3.0
    @Environment(\.legibilityWeight) private var legibilityWeight
    
    private var borderWidth: CGFloat {
        // Double border width when Bold Text is enabled
        legibilityWeight == .bold ? baseBorderWidth * 2 : baseBorderWidth
    }
    
    var body: some View {
        content
            .overlay(
                RoundedRectangle(cornerRadius: 8)
                    .strokeBorder(.tint, lineWidth: borderWidth)
            )
    }
}
```

## Relative Frame

Use container-relative sizing:

```swift
Text("Content")
    .containerRelativeFrame(.horizontal) { length, _ in
        length * 0.8
    }
```

## Example: Adaptive Card

```swift
struct CardView: View {
    let title: String
    let subtitle: String
    
    @Environment(\.dynamicTypeSize) var dynamicTypeSize
    @ScaledMetric(relativeTo: .body) var imageSize: CGFloat = 60
    
    var body: some View {
        Group {
            if dynamicTypeSize.isAccessibilitySize {
                VStack(alignment: .leading) { content }
            } else {
                HStack { content }
            }
        }
        .padding()
    }
    
    @ViewBuilder
    var content: some View {
        Image(systemName: "photo")
            .frame(width: imageSize, height: imageSize)
        
        VStack(alignment: .leading) {
            Text(title)
                .font(.headline)
            Text(subtitle)
                .font(.subheadline)
                .foregroundStyle(.secondary)
        }
    }
}
```

## Example: Adaptive Grid

```swift
struct AdaptiveGridView: View {
    let items: [Item]
    
    @Environment(\.dynamicTypeSize) var dynamicTypeSize
    
    var columns: [GridItem] {
        let count = dynamicTypeSize.isAccessibilitySize ? 1 : 2
        return Array(repeating: GridItem(.flexible()), count: count)
    }
    
    var body: some View {
        LazyVGrid(columns: columns) {
            ForEach(items) { item in
                ItemView(item: item)
            }
        }
    }
}
```

## Preview with Different Sizes

Preview all Dynamic Type sizes:

```swift
#Preview {
    ContentView()
}
```

Use the Xcode preview toolbar, go to "Variants" and "Dynamic Type Variaants" to preview the layout for all sizes.

### Explicit size in preview

```swift
#Preview {
    ContentView()
        .dynamicTypeSize(.accessibility3)
}
```

## Large Content Viewer

For bar items and other elements that don't scale, provide a Large Content Viewer:

```swift
Button { showBasket.toggle() } label: {
    ZStack(alignment: .topTrailing) {
        Image(systemName: "cart.fill")
        if basket.orderCount > 0 {
            Text("\(basket.orderCount)")
                .padding(5)
                .background(.red)
                .clipShape(Capsule())
        }
    }
}
.accessibilityShowsLargeContentViewer {
    Image(systemName: "cart.fill")
    Text("Cart, \(basket.orderCount) items")
}
```

Users with Larger Accessibility Sizes can tap-and-hold to see the enlarged content in the center of the screen.
Use high-quality vector assets (for example SF Symbols or vector PDFs) so enlarged previews stay crisp.

## Conditional Modifier Pattern

A reusable pattern for conditionally applying modifiers based on accessibility settings:

```swift
extension View {
    @ViewBuilder
    func `if`<Content: View>(_ condition: Bool, transform: (Self) -> Content) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}
```

Usage for accessibility modifiers:

```swift
@Environment(\.verticalSizeClass) private var verticalSizeClass

private var isLandscape: Bool {
    verticalSizeClass == .compact
}

var body: some View {
    Slider(value: $progress)
        .if(!isLandscape) { view in
            view.accessibilityShowsLargeContentViewer()
        }
}
```

This avoids duplicating views and keeps conditional logic readable.

## Constrained Dynamic Type

For elements that shouldn't scale beyond a certain size (this should be avoided, and you should have a very good reason or alternative to do it), constrain the Dynamic Type size:

```swift
Slider(value: $progress)
    .dynamicTypeSize(.large)  // Cap at Large size
    .accessibilityShowsLargeContentViewer()  // Provide alternative for larger sizes
```

### Example: Progress slider

```swift
Slider(value: $sliderValue, in: 0...duration)
    .accessibilityLabel("Progress")
    .accessibilityValue(currentLineText)
    .if(!isLandscape) { view in
        view.dynamicTypeSize(.large)
            .accessibilityShowsLargeContentViewer()
    }
```

Always pair constrained elements with Large Content Viewer so users with accessibility sizes can still access the information.

## Minimum Scale Factor

Allow text to shrink slightly before wrapping (use sparingly):

```swift
Text("Long title that might not fit")
    .minimumScaleFactor(0.8)
```

## Testing

### Environment Overrides

In Xcode's Debug Area toolbar, click Environment Overrides to change Dynamic Type size.

### Simulator shortcut

`Option + Command + +/-` to increase/decrease text size.


## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://accessibilityupto11.com/blog/
- https://github.com/dadederk/fromZeroToAccessible (Daniel Devesa Derksen-Staats and Rob Whitaker)
