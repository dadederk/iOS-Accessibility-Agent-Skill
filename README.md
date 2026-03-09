# iOS Accessibility Agent Skill

Expert guidance for any AI coding tool that supports the Agent Skills open format: accessible iOS development, VoiceOver, Dynamic Type, assistive technologies, and inclusive design practices.

## ⚠️ Important Disclaimers

### Work in Progress

This skill is a work in progress and has not been exhaustively tested. It is provided as-is, and **it is the responsibility of the developer** to thoroughly test and validate any changes suggested through the use of this skill. Ensure that all accessibility implementations work as expected and intended for your users before shipping to production.

### Accessibility Is Not Deterministic

Unlike many other iOS engineering tasks, accessibility is **not deterministic**. There is rarely a single "correct" fix for an accessibility problem. Multiple approaches can be valid, each with different trade-offs. Evaluate options in context and choose what works best for your users.

**This skill is an aid, not a substitute for intentional and thoughtful design.** Use it as a tool to learn more about accessibility and guide implementation decisions, but do not treat it as a magic solution. Real accessibility requires continuous learning, testing with assistive technologies, and, whenever possible, feedback from users with disabilities.

**Your feedback helps improve this skill.** If you run into issues, share your prompt and a sample of the output (whatever you are comfortable sharing) so we can refine these guidelines. Positive feedback is also useful to understand what is working well.

## Who This Is For

- Developers building accessible iOS apps with UIKit or SwiftUI
- Teams wanting to embed accessibility into their development workflow
- Anyone building for, or debugging issues on, VoiceOver, Dynamic Type layouts, or any other assistive technologies, accessibility features, and inclusive design.

## How to Use This Skill

### Option A: Using skills.sh (recommended)

Install this skill with a single command:

```bash
npx skills add https://github.com/dadederk/iOS-Accessibility-Agent-Skill --skill ios-accessibility
```

Then use the skill in your AI agent, for example:

> Use the ios-accessibility skill and review the current view to analyze and find opportunities to improve its VoiceOver support.

### Option B: Manual Install

1. **Clone** this repository
2. **Install or symlink** the `ios-accessibility/` folder following your tool's skills installation docs
3. **Use your AI tool** and ask it to use the "ios-accessibility" skill for accessibility tasks

#### Where to Save Skills

- **Codex:** [Where to save skills](https://codex.openai.com/docs/skills)
- **Claude Code:** [Using Skills](https://docs.anthropic.com/claude-code/skills)
- **Cursor:** [Enabling Skills](https://docs.cursor.com/skills)

## What This Skill Offers

### Platform-Specific Guidance

- **VoiceOver (UIKit)**: Labels, traits, grouping, custom actions, notifications
- **VoiceOver (SwiftUI)**: Accessibility modifiers, element grouping, focus management
- **Dynamic Type (UIKit)**: Text styles, `UIFontMetrics`, layout adaptation
- **Dynamic Type (SwiftUI)**: Scaled metrics, adaptive layouts, environment values

### Other Assistive Technologies Support

- **Voice Control**: Input labels, naming consistency, testing with "Show names"
- **Switch Control**: Scanning, grouping, custom actions
- **Full Keyboard Access**: Focus order, keyboard shortcuts

### Cross-Cutting Best Practices

- Touch target sizes (44×44 points minimum)
- Color contrast ratios and Increase Contrast
- Reduce Motion, Reduce Transparency, Bold Text, etc. support
- Multi-modal information (color, shape, iconography, text, sound, haptics...)

### Testing Guidance

- Manual testing with assistive technologies
- Automated testing with Accessibility Inspector and UI tests
- Limitations of automation (manual testing is essential)

### Inclusive Design Mindset

- Be thoughtful and context-aware: compare options and validate with real users whenever possible
- Shift left: consider accessibility during design
- Cross-assistive technology benefits: implement once, benefit broadly

## Scope and Non-Goals

This skill is focused on iOS app accessibility guidance (UIKit and SwiftUI), including implementation and testing patterns for major Apple assistive technologies.

This skill does not try to be:

- A legal compliance service or certification process
- A replacement for testing with real users and assistive technologies
- A complete guide for non-iOS platforms (web, Android, desktop)

## Complementary Accessibility Skills

This skill was developed in parallel with other accessibility agent skills in the community. It is fair to recognize their great work and point you to these complementary skills.
Both of the following skills were released before this one.

- [swift-accessibility-skill](https://github.com/PasqualeVittoriosi/swift-accessibility-skill) by Pasquale Vittoriosi.
  If you need stronger coverage for topics outside this skill's core iOS scope (for example: **macOS accessibility**, **Accessibility Nutrition Labels**, and **WCAG**), consider this one.
- [apple-accessibility-skills](https://github.com/rgmez/apple-accessibility-skills) by Roberto Gómez

## Accessibility Nutrition Labels

For **Accessibility Nutrition Labels**, use Apple’s official guidance:

- [Overview of Accessibility Nutrition Labels](https://developer.apple.com/help/app-store-connect/manage-app-accessibility/overview-of-accessibility-nutrition-labels/)
- [Manage Accessibility Nutrition Labels](https://developer.apple.com/help/app-store-connect/manage-app-accessibility/manage-accessibility-nutrition-labels)

## Skill Structure

The skill is organized as follows:

```text
ios-accessibility/                  # Main skill folder
├── SKILL.md                        # Entry point and behavior contract
└── references/                     # Detailed implementation guides
    ├── voiceover.md                # Core VoiceOver concepts and testing
    ├── voiceover-uikit.md          # UIKit VoiceOver implementation patterns
    ├── voiceover-swiftui.md        # SwiftUI VoiceOver implementation patterns
    ├── dynamic-type.md             # Core Dynamic Type concepts and scaling model
    ├── dynamic-type-uikit.md       # UIKit Dynamic Type implementation patterns
    ├── dynamic-type-swiftui.md     # SwiftUI Dynamic Type implementation patterns
    ├── voice-control.md            # Voice Control patterns and pitfalls
    ├── switch-control.md           # Switch Control scanning/grouping guidance
    ├── full-keyboard-access.md     # Keyboard navigation and focus behavior
    ├── good-practices.md           # Cross-cutting accessibility best practices
    ├── concepts-and-culture.md     # Inclusive design mindset and team culture
    ├── testing-manual.md           # Manual testing workflows by assistive tech
    ├── testing-automated.md        # Automated testing and inspector guidance
    ├── playbook.md                 # Quick fixes, checklists, and common mistakes
    ├── glossary.md                 # Accessibility terminology reference
    └── resources.md                # Sources and further reading
```

## Sources

- [Accessibility Up To 11](https://accessibilityupto11.com/365-days-ios-accessibility/) by Daniel Devesa Derksen-Staats
- [Mobile A11y](https://mobilea11y.com) by Rob Whitaker
- [From Zero to Accessible](https://github.com/dadederk/fromZeroToAccessible) workshop by Rob Whitaker and Daniel Devesa Derksen-Staats
- [Fostering an Accessibility Culture](https://www.smashingmagazine.com/2025/04/fostering-accessibility-culture/) (Smashing Magazine)
- [Developing Accessible iOS Apps](https://link.springer.com/book/10.1007/978-1-4842-5308-3) (Apress)
- [Create with Swift — Make It Accessible](https://www.createwithswift.com/make-it-accessible)

## Contributing

Contributions are welcome! This repository follows the [Agent Skills open format](https://agentskills.io).

## Validation

To validate the skill against the Agent Skills reference format, use the `skills-ref` validator (for example, `npx skills-ref ios-accessibility/`).

## About the Author

Created by Daniel Devesa Derksen-Staats, an iOS engineer specializing in accessibility. Author of [Developing Accessible iOS Apps](https://link.springer.com/book/10.1007/978-1-4842-5308-3) and creator of the [Accessibility Up To 11](https://accessibilityupto11.com) series.

## License

This skill is open-source and available under the MIT License. See [LICENSE](LICENSE) for details.
