# Dynamic Type

Core concepts for scalable text and adaptive layouts on iOS.

## What Is Dynamic Type

Dynamic Type lets users choose their preferred text size. iOS scales text automatically when you use text styles.

## Text Styles

iOS provides semantic text styles that scale together:

| Style | Typical Use |
|-------|-------------|
| `.largeTitle` | Screen titles |
| `.title`, `.title2`, `.title3` | Section headings |
| `.headline` | Emphasized body text |
| `.subheadline` | Secondary labels |
| `.body` | Main content |
| `.callout` | Supplementary descriptions |
| `.footnote` | Tertiary info |
| `.caption`, `.caption2` | Metadata, timestamps |

Use text styles instead of fixed font sizes.
For exact point sizes by text style and content size category, see Apple HIG: [iOS/iPadOS Dynamic Type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-Dynamic-Type-sizes).

Text styles do not scale linearly with a fixed ratio across categories. Avoid assumptions like "title is always X times body" at larger sizes.

## Larger Accessibility Sizes

Beyond the 7 standard sizes, iOS offers 5 additional Accessibility Sizes. Users enable them in **Settings > Accessibility > Display & Text Size > Larger Text**.
Reference sizes are documented in Apple HIG: [iOS/iPadOS larger accessibility type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-larger-accessibility-type-sizes).

Always test with these larger sizes — they reveal layout issues that standard sizes don't.

### Test the worst case

Dynamic Type issues often show up in edge screens like empty states, error views, and popovers.

- Test at **Accessibility 5** (largest size)
- Test with **longer localized strings** (e.g., German, Spanish)
- Check **loading/error** screens, not just primary flows

## Layout Adaptation

At larger sizes, layouts often need to change:
- Horizontal stacks become vertical
- Multi-column grids become single column
- More lines of text are allowed
- Elements may need to wrap

## Large Content Viewer

Navigation bars, tab bars, and toolbars don't scale with Dynamic Type. Users can long-press to see a magnified label via Large Content Viewer.

## Testing

1. Use Control Center's Text Size control
   - Test app-specific overrides as well (Text Size can be adjusted per app from Control Center)
2. Test with Larger Accessibility Sizes enabled
3. Use Xcode's Environment Overrides
4. Use Accessibility Inspector's Settings tab
5. Try the Double-Length Pseudolanguage for stress testing

## Implementation

For UIKit implementation, see `dynamic-type-uikit.md`.

For SwiftUI implementation, see `dynamic-type-swiftui.md`.

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://github.com/Apress/developing-accessible-iOS-apps
