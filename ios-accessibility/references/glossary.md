# Accessibility Glossary

Quick reference for common accessibility terms and concepts.

## Assistive Technologies

**VoiceOver** — Apple's screen reader. Reads screen content aloud and allows navigation via gestures or keyboard. Users with blindness or low vision rely on VoiceOver to use iOS apps.

**Voice Control** — System-wide voice navigation and dictation. Recognizes accessibility labels and allows full device control without internet.

**Switch Control** — Scanning-based navigation for external switches. Users cycle through elements and activate with a switch press. Critical for users with severe motor disabilities.

**Full Keyboard Access** — Navigate and control iOS with an external keyboard. Useful for users who cannot use touch input.

**Dynamic Type** — System-wide text size control. Users choose from 12 sizes (7 standard + 5 accessibility sizes). Text scales from -3 to +5 relative to default. See Apple HIG for exact sizes: [iOS/iPadOS Dynamic Type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-Dynamic-Type-sizes) and [iOS/iPadOS larger accessibility type sizes](https://developer.apple.com/design/human-interface-guidelines/typography#iOS-iPadOS-larger-accessibility-type-sizes).

**Zoom** — System-wide screen magnification. Users can zoom in up to 15x. Different from pinch-to-zoom within apps.

**Large Content Viewer** — Tap-and-hold interface for elements that don't scale. Shows enlarged version in center of screen. iOS 13+ ([`UILargeContentViewerInteraction`](https://developer.apple.com/documentation/uikit/uilargecontentviewerinteraction)).

**Guided Access** — Locks the device to a single app and can restrict touch areas. Useful for education, kiosks, or focused tasks.

**Magnifier** — Camera-based zoom tool for viewing real-world content. Useful for reading printed text or signs.

**Assistive Touch (AssistiveTouch)** — On-screen menu for gestures and actions. Reduces need for complex multi-finger gestures.

## Accessibility Properties

**accessibilityLabel** — The name of an element. Answers "What is this?" Examples: "Close", "Play", "Settings". Should be concise and not include the control type.

**accessibilityValue** — The current state or value. Answers "What's its current state?" Examples: "50 percent", "On", "Line 3 of 10". Updates as state changes.

**accessibilityHint** — Description of the result of an action. Answers "What happens when I use it?" Examples: "Plays the audio", "Opens settings". Optional; use sparingly for non-obvious actions.

**accessibilityTraits** — Characteristics that describe the element's role and state. Examples: `.button`, `.header`, `.selected`, `.adjustable`. Multiple traits can be combined.

**accessibilityCustomActions** — Secondary actions available through VoiceOver's Actions rotor. Example: Delete, Share, Mark as Read. iOS 8+ ([`UIAccessibilityCustomAction`](https://developer.apple.com/documentation/uikit/uiaccessibilitycustomaction)).

**accessibilityElements** — Ordered array of accessible elements within a container. Used to control navigation order when automatic ordering isn't correct.

## Common Traits

**.button** — Tappable control that performs an action

**.header** — Section heading (shows in VoiceOver's Headings rotor)

**.selected** — Currently selected item in a group (picker, tabs, segmented control)

**.adjustable** — Value can be incremented/decremented (sliders, steppers, pickers)

**.link** — Opens a URL or navigates

**.isModal** — Modal dialog that restricts focus

**.updatesFrequently** — Value changes rapidly (prevents VoiceOver interruptions)

**.startsMediaSession** — Plays audio/video when activated

**.allowsDirectInteraction** — Passes touches through (for drawing, piano apps)

## Accessibility Features (iOS Settings)

**Reduce Motion** — Minimizes animations and parallax effects. Users enable to reduce vestibular triggers and motion sickness.

**Reduce Transparency** — Makes translucent backgrounds opaque. Improves contrast and reduces visual complexity.

**Increase Contrast** — Increases color contrast throughout the system. Helps users with low vision distinguish elements.

**Differentiate Without Color** — Adds shapes/icons alongside color to convey meaning. Essential for color-blind users.

**Bold Text** — Makes system fonts bolder. Requires app restart.

**Button Shapes** — Adds outlines/underlines to buttons. Helps identify interactive elements.

**Smart Invert Colors** — Inverts colors except for images and media. Alternative to Dark Mode for high contrast.

**Invert Colors** — Classic color inversion that affects all content, including images.

**Mono Audio** — Plays audio channels in mono. Useful for users with hearing loss in one ear.

**LED Flash for Alerts** — Flashes the camera LED for notifications (helpful for deaf or hard-of-hearing users).

**Audio Descriptions** — Narration that describes visual content in videos.

**Made for iPhone Hearing Aids** — Lets users stream audio directly to compatible hearing aids.

**Larger Accessibility Sizes** — Five text sizes beyond the standard Large (from XXL to Accessibility 5).

## Text Styles (Dynamic Type)

**Standard Sizes:**
- Large Title (34pt default)
- Title (28pt)
- Title 2 (22pt)
- Title 3 (20pt)
- Headline (17pt bold)
- Body (17pt) — base size
- Callout (16pt)
- Subheadline (15pt)
- Footnote (13pt)
- Caption (12pt)
- Caption 2 (11pt)

All scale proportionally with user's text size setting.

## Concepts

**Accessibility Element** — Any UI component that assistive technologies can interact with. By default, standard controls (buttons, labels) are accessibility elements; custom views are not.

**Focus** — The current element that VoiceOver or Full Keyboard Access is highlighting. Only one element has focus at a time.

**Rotor** — VoiceOver gesture (two-finger rotation) that reveals a context menu. Common rotors: Headings, Links, Form Controls, Landmarks, Custom Actions.

**Semantic Colors** — System colors that adapt to Light/Dark mode and Increase Contrast. Examples: `.label`, `.systemBackground`, `.secondaryLabel`.

**Hit Area** — The tappable region of a control. Should be at least 44×44 points for accessibility.

**Isolation Domain** — In Swift Concurrency context: main actor vs background actors. For accessibility: ensuring UI updates happen on main thread.

**Grouping** — Combining multiple elements into one accessibility element. Reduces swipe count and makes navigation easier.

**Accessibility Container** — A view that contains other accessibility elements and can control their order.

## Testing Terms

**Accessibility Inspector** — Xcode tool for inspecting accessibility properties and running audits. Window > Accessibility Inspector.

**Accessibility Identifier** — String used to identify elements in UI tests. Not read by VoiceOver; purely for automation.

**Environment Overrides** — Xcode feature to test different accessibility settings without changing system settings.

**Audit** — Accessibility Inspector feature that checks for common issues: missing labels, low contrast, small targets.

**Caption Panel** — VoiceOver feature showing speech output as text. Settings > Accessibility > VoiceOver > Caption Panel.

**Screen Curtain** — VoiceOver feature that turns off the display while keeping the phone functional. Three-finger triple-tap.

## Abbreviations

**AT** — Assistive Technology

**A11y** — Numeronym for "accessibility" (a + 11 letters + y)

**VO** — VoiceOver

**DT** — Dynamic Type

**WCAG** — Web Content Accessibility Guidelines (applies to iOS apps too)

**ARIA** — Accessible Rich Internet Applications (web standard; iOS equivalents exist)

## Related Terms

**Inclusive Design** — Design approach that considers diverse human abilities from the start, not as an afterthought.

**Universal Design** — Design usable by all people, to the greatest extent possible, without adaptation.

**Disability** — Mismatch between a person's abilities and their environment. Accessibility removes barriers.

**Permanent, Temporary, Situational** — Types of disabilities. Example: blind (permanent), eye injury (temporary), bright sunlight (situational).

**Social Model of Disability** — Framework that views disability as created by barriers in society, not by the person's impairment.

## Sources

- [Apple Developer Documentation — UIKit Accessibility](https://developer.apple.com/documentation/uikit/accessibility)
- [Apple Human Interface Guidelines — Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Accessibility Up To 11 — #365DaysIOSAccessibility](https://accessibilityupto11.com/365-days-ios-accessibility/)
