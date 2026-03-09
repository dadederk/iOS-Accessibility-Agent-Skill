# Automated Testing

Automated accessibility testing for iOS: UI tests, static analysis, and Accessibility Inspector.

## What Automation Can Do

Automated tools are good for:
- Detecting missing labels
- Catching regressions after code changes
- Enforcing baseline rules
- Flagging obvious issues (contrast, target size)

## What Automation Cannot Do

Automation cannot evaluate:
- Whether a label makes sense in context
- If navigation order is logical
- Whether the experience is actually usable
- How complex interactions feel

**Always pair automation with manual testing.**

## UI Testing for Accessibility

### Use accessibilityIdentifier for tests

`accessibilityIdentifier` is for test automation. `accessibilityLabel` is read by VoiceOver.

```swift
// In production code
submitButton.accessibilityIdentifier = "submit-button"
submitButton.accessibilityLabel = "Submit order"
```

```swift
// In UI test
let submitButton = app.buttons["submit-button"]
XCTAssertTrue(submitButton.exists)
```

### Assert on accessibility properties

```swift
let cell = app.cells["order-cell"]
XCTAssertEqual(cell.label, "Order #1234, $42.00")
XCTAssertTrue(cell.accessibilityTraits.contains(.button))
```

### Test VoiceOver experience programmatically

Query the accessibility tree:

```swift
let app = XCUIApplication()
let elements = app.descendants(matching: .any).allElementsBoundByAccessibilityElement

for element in elements {
    if element.isHittable && element.label.isEmpty {
        XCTFail("Unlabeled element: \(element)")
    }
}
```

### Test Dynamic Type

Launch with accessibility size:

```swift
let app = XCUIApplication()
app.launchArguments += ["-UIPreferredContentSizeCategoryName", "UICTContentSizeCategoryAccessibilityExtraExtraLarge"]
app.launch()
```

## Accessibility Inspector Audit

### Launch

**Xcode > Open Developer Tool > Accessibility Inspector**

### Run audit

1. Select target (simulator or device)
2. Click the Audit tab
3. Click "Run Audit"

### Common audit findings

| Issue | Fix |
|-------|-----|
| Missing label | Add `accessibilityLabel` |
| Insufficient contrast | Increase color contrast |
| Small touch target | Expand hit area to 44×44 |
| Missing traits | Add appropriate traits |
| Image label is file name | Provide a meaningful label or hide decorative images |

### Per-element inspection

1. Click the crosshair button
2. Click an element in the simulator
3. View its accessibility properties

### VoiceOver preview

Use the speaker icon to hear how VoiceOver would read the current screen. You can step through elements or play all.

### Color Contrast Calculator

In Accessibility Inspector: **Window > Color Contrast Calculator** for quick contrast checks.

### Notifications log

**Window > Show Notifications** to see posted accessibility notifications.

## SwiftUI Accessibility Inspector

In Xcode's Inspectors panel (right sidebar), the Accessibility section shows:
- Label
- Value
- Traits
- Identifier

Select a view in the canvas to see its accessibility info.

## SwiftLint Rules

Add lint rules to catch common issues:

```yaml
# .swiftlint.yml
custom_rules:
  image_accessibility:
    regex: 'Image\s*\(\s*\"[^\"]+\"\s*\)'
    message: "Image should have accessibilityLabel or use Image(decorative:)"
```

Community rules also exist for accessibility enforcement.

## Automated Contrast Checking

### Accessibility Inspector

**Window > Show Color Contrast Calculator**

Enter foreground and background colors to check ratio.

### Programmatic checking

```swift
extension UIColor {
    func contrastRatio(with other: UIColor) -> CGFloat {
        let l1 = relativeLuminance
        let l2 = other.relativeLuminance
        let lighter = max(l1, l2)
        let darker = min(l1, l2)
        return (lighter + 0.05) / (darker + 0.05)
    }
    
    var relativeLuminance: CGFloat {
        var r: CGFloat = 0, g: CGFloat = 0, b: CGFloat = 0
        getRed(&r, green: &g, blue: &b, alpha: nil)
        
        func adjust(_ c: CGFloat) -> CGFloat {
            c <= 0.03928 ? c / 12.92 : pow((c + 0.055) / 1.055, 2.4)
        }
        
        return 0.2126 * adjust(r) + 0.7152 * adjust(g) + 0.0722 * adjust(b)
    }
}
```

### Unit test for contrast

```swift
func testButtonContrastMeetsMinimum() {
    let foreground = UIColor.white
    let background = UIColor.systemBlue
    let ratio = foreground.contrastRatio(with: background)
    XCTAssertGreaterThanOrEqual(ratio, 4.5, "Text contrast should be at least 4.5:1")
}
```

## Environment Overrides

In Xcode's Debug Area toolbar, click the Environment Overrides button to test:
- Dynamic Type sizes
- Increase Contrast
- Reduce Motion
- Reduce Transparency
- Bold Text

Changes apply immediately without rebuilding.

## Continuous Integration

### Run UI tests in CI

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 15' \
    -testPlan AccessibilityTests
```

### Accessibility test plan

Create a dedicated test plan for accessibility checks. Run it on every PR.

## Limitations

| Tool | Coverage |
|------|----------|
| Accessibility Inspector Audit | ~30% of issues |
| UI tests | Regressions, presence of labels |
| SwiftLint | Code patterns only |

The remaining 70%+ requires manual testing.

## Checklist

- [ ] `accessibilityIdentifier` used for test automation
- [ ] `accessibilityLabel` assertions in UI tests
- [ ] Accessibility Inspector audit passes
- [ ] Contrast checked for custom colors
- [ ] Environment Overrides tested in debug
- [ ] Manual testing performed

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
