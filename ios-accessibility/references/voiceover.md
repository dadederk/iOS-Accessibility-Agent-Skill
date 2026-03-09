# VoiceOver

Core concepts for VoiceOver accessibility on iOS.

## What Is VoiceOver

VoiceOver is a screen reader that speaks interface elements aloud and enables gesture-based navigation, mainly but not exclusively, for users who are blind or have low vision.

## Focus and Activation Basics

- **Single tap** moves focus to the tapped element and reads it.
- **Double‑tap** activates the focused element (button, switch, link).

This is why labels, values and traits matter: VoiceOver relies on them to describe what the element is and how to interact with it.

## Explore vs Flick Navigation

VoiceOver users navigate in two main ways:

- **Explore by touch** — drag a finger around the screen to hear what is under it.
- **Flick navigation** — swipe left/right to move through elements (previous/next) in a structured order.

When you group elements well and set a logical order, flick navigation becomes fast and predictable.

## Core Accessibility Properties

The core properties for accessible elements (label is the only one that should be mandatory, unless there is a very good reason for it not to - in which case it should at least have a value):

| Property | Purpose | Example |
|----------|---------|---------|
| **Label** | Name of the element | "Rating" |
| **Value** | Current state | "5 stars" |
| **Hint** | Result of action, or extra context | "Rates how much you like the conference from 1 to 5 stars" |
| **Traits** | Role and state | adjustable |

### Labels

- Be concise and descriptive (take into account context)
- Don't include the control type ("Play" not "Play button")
- Localize for all supported languages
- Match visible text when possible (helps Voice Control too)

### Values

Use for controls with changing state:
- Sliders: "50 percent"
- Toggles: "On" / "Off"
- Steppers: "3 of 5"

### Hints

Use sparingly — only when the action could use extra context on top of label and traits.
When possible, phrase hints as outcomes rather than commands, for example: "Deletes this item from your list."

### Traits

Communicate role and state:
- **Button** — double-tap to activate
- **Header** — section heading (enables rotor navigation)
- **Selected** — item currently chosen
- **Adjustable** — swipe up/down to change value
- **Link** — opens a URL
- **Image** — image content
- **Disabled (not enabled)** — unavailable control

Use the framework-specific API names from the implementation files:
- SwiftUI: `references/voiceover-swiftui.md`
- UIKit: `references/voiceover-uikit.md`

## Grouping

Group related elements to:
- Reduce swipe count
- Provide better context
- Simplify navigation

Patterns:
- **Automatic merge (SwiftUI `.combine`)** — children are combined into one spoken element
- **Manual merge** — treat the group as one element and provide label/value/traits yourself
- **Semantic grouping / grouped traversal** — keep children individually accessible while preserving group context

SwiftUI API names for these patterns are `.combine`, `.ignore`, and `.contain`. UIKit uses `isAccessibilityElement` for manual merging, and `shouldGroupAccessibilityChildren` for grouped traversal (children stay separate, but VoiceOver goes through that group before moving on). Use explicit `accessibilityElements` ordering when reading order needs to be customized.

## Navigation Order

VoiceOver navigates left-to-right, top-to-bottom by default. Override only when the visual layout doesn't match the logical task flow.

## Custom Actions

Expose secondary actions (delete, share, etc.) without cluttering the interface. Actions appear in VoiceOver's Actions rotor. Ideally these actions should be independently accessible too, for example, in a details screen or an overflow menu.

## Supplementary Content

For data-rich UI, provide additional context as custom accessibility content instead of overloading the main label/value. Keep the most important state in label/value/traits first, then add secondary details as supplementary content. Whenever possible, make that same information independently accessible elsewhere (for example, in a details screen).

## Focus and Announcements

**Move focus after:**
- **Screen transitions** — only if the new screen is a custom overlay that significantly covers the current screen, or blocks interaction on the screen underneath (e.g., a custom modal pop-up)
- **New content appearing** — after an action is taken (layout changed)

**Announce:**
- Important events that users might miss (error states, long-running tasks finishing)

## Gestures

| Gesture | Action |
|---------|--------|
| Tap | Select element |
| Double-tap | Activate |
| Swipe right/left | Navigate next/previous element |
| Swipe up/down | Adjust or navigate by rotor |
| Two-finger rotate | Change rotor mode |
| Two-finger double-tap | Magic Tap (primary action) |
| Two-finger scrub | Escape (dismiss) |
| Three-finger swipe | Scroll |
| Three-finger double-tap | Mute/unmute VoiceOver speech |
| Three-finger triple-tap | Screen Curtain |

## Testing

**Enable VoiceOver:**
- **Accessibility Shortcut** (fastest if configured):
  - Triple-click side button (or Home button on older devices)
  - If multiple features are enabled in Accessibility Shortcut, select VoiceOver from the menu
  - Configure at: Settings > Accessibility > Accessibility Shortcut > VoiceOver
- **Settings**: Settings > Accessibility > VoiceOver > toggle on
- **Siri**: "Hey Siri, turn on VoiceOver" (or "turn off VoiceOver")

**Testing steps:**
1. Navigate through every screen
2. Verify labels, values, traits, and order
3. All content is conveyed and every action can be performed
4. Test custom actions via Actions rotor
5. If you feel confident, test with Screen Curtain (three-finger triple-tap) to experience it without visual cues

## Implementation

For UIKit implementation details, see `voiceover-uikit.md`.

For SwiftUI implementation details, see `voiceover-swiftui.md`.

## Sources

- https://accessibilityupto11.com/365-days-ios-accessibility/
- https://github.com/Apress/developing-accessible-iOS-apps
