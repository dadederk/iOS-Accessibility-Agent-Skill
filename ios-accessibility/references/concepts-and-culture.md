# Concepts and Culture

Accessibility mindset, inclusive design principles, and organizational practices.

## Contents

- [Accessibility Is Non-Deterministic](#accessibility-is-non-deterministic)
- [Inclusive Design](#inclusive-design)
- [Models of Disability](#models-of-disability)
- [Accessibility Is a Practice](#accessibility-is-a-practice)
- [Culture and Allies](#culture-and-allies)
- [Common Misconceptions](#common-misconceptions)
- [Shift Left](#shift-left)
- [Cross-Assistive Technology Benefits](#cross-assistive-technology-benefits)
- [Quotes to Remember](#quotes-to-remember)
- [Resources for Continued Learning](#resources-for-continued-learning)
- [Organizational Practices](#organizational-practices)
- [Checklist for Culture](#checklist-for-culture)
- [Sources](#sources)

## Accessibility Is Non-Deterministic

There is no universal "correct" accessible experience. Accessibility is about users and their contexts:
- Different disabilities require different adaptations
- The same user may have different needs in different situations
- Different approaches may have different trade-offs for different assistive technologies

**Good practices are defaults, not absolutes.** Validate with real users and assistive technologies, then refine based on feedback.

### Start with empathy

If a flow is hard with VoiceOver or Switch Control, it’s often hard visually too. Accessibility testing can surface general UX issues earlier.

### Four accessibility areas

Apple divides its accessibility features in four categories:
- **Vision** (blindness, low vision, color blindness)
- **Hearing** (deafness, hard of hearing)
- **Motor** (limited dexterity, tremors)
- **Cognitive** (learning, attention, memory)

## Inclusive Design

Inclusive design considers the full range of human diversity from the start — not as an afterthought.

### Principles

1. **Recognize exclusion**: Identify who is being left out and why
2. **Solve for one, extend to many**: Solutions for specific disabilities often benefit everyone
3. **Learn from diversity**: Get people with disabilities involved as much as possible

### Language and naming

Names and labels shape how inclusive a product feels.

- Prefer neutral, descriptive names over ability-loaded labels
- Avoid framing users as "advanced" vs "limited" based on access needs
- Keep option names specific to behavior (for example, "Guided mode", "Relaxed timing", "High contrast cues")

### Example

Curb cuts were designed for wheelchair users but benefit parents with strollers, travelers with luggage, and delivery workers with carts.

## Models of Disability

### Medical model

Views disability as a defect in the individual that needs fixing.

### Social model

Views disability as a mismatch between the person and their environment. The barrier is external, not internal.

Accessibility work often aligns with the social model: remove barriers in products rather than "fix" users.

## Accessibility Is a Practice

> "Accessibility is both a state and a practice." - Sommer Panage

Treating accessibility as a one-time audit leads to regression. Embedding accessibility into design and engineering systems creates lasting improvement.

### Outcomes over one-off outputs

An accessibility backlog or an audit report is useful, but not the goal. The goal is users completing real tasks successfully with assistive technologies.

Use audits as inputs for prioritization and planning, not as an endpoint.

### Habits that stick

- Consider accessibility during design, not just at the end (shift left)
- Improve shared components to scale impact
- Automate for guardrails, not full coverage
- Test with assistive technologies regularly

### Prioritize by impact

When time is limited:

- Start with top user journeys (onboarding, auth, checkout, media playback, settings)
- Fix blockers before polish
- Prioritize shared components and design tokens for broad impact
- Ship in iterations, then re-test and refine

## Culture and Allies

Accessibility specialists can't do it alone. Building a culture means:
- Training designers and engineers on fundamentals
- Creating safe spaces (Slack channels, office hours) for questions
- Celebrating wins and recognizing contributions
- Embedding accessibility into onboarding

### Leadership and buy-in

Culture change works best with both bottom-up and top-down support:

- Bottom-up: champions demonstrate practical fixes and mentor others
- Top-down: leadership protects time and includes accessibility in quality expectations
- Communication: explain impact in user outcomes and product risk, not only compliance language

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| "Accessibility is about improving the experience for VoiceOver users" | There are many assistive technologies, accessibility features, and good practices to take into account |
| "Accessibility is expensive" | Retrofitting is expensive; building accessible from the start is not |
| "Automation covers it" | not all issues are detectable; accessibility often requires judgment |
| "Compliance equals usable" | Passing audits doesn't guarantee a good experience |

## Shift Left

Address accessibility earlier in the process:

| Stage | Action |
|-------|--------|
| Design | Annotate mockups with labels, traits... consider color contrast, how does the interface work for accessibility text sizes |
| Development | Implement accessibility as you build |
| Code review | Check accessibility properties |
| Testing | Manual testing with assistive tech |
| Audit | Validation, not just discovery |

The later issues are found, the more expensive they are to fix.

## Cross-Assistive Technology Benefits

APIs like `accessibilityLabel`, `accessibilityTraits`, and custom actions benefit multiple technologies:
- VoiceOver
- Voice Control
- Switch Control
- Full Keyboard Access
- Braille displays

Implement once, benefit broadly. Start with VoiceOver — it covers most fundamentals.

## Quotes to Remember

> "We have one job, and that's to make our apps work. And if you are not implementing accessibility features, you are forgetting about making it work for a lot of people."
> — @NovallSwift

> "Awareness is the biggest problem here."
> — Marco Arment (on accessibility in the Apple ecosystem)

## Resources for Continued Learning

- [Accessibility Up To 11 — Resources](https://accessibilityupto11.com/resources/)
- [Accessibility Up To 11 — #365DaysIOSAccessibility archive](https://accessibilityupto11.com/365-days-ios-accessibility/)
- [Fostering An Accessibility Culture (Smashing Magazine)](https://www.smashingmagazine.com/2025/04/fostering-accessibility-culture/)
- [WWDC Accessibility videos (Apple)](https://developer.apple.com/videos/frameworks/accessibility)
- [Mobile A11y](https://mobilea11y.com)
- Global Accessibility Awareness Day events

## Organizational Practices

### Champions network

Train advocates across teams who can answer basic questions and escalate complex issues. For larger organizations, a lightweight accessibility guild can coordinate work across teams.

### Accessible content

Accessibility isn't just code. Documentation, marketing, and support content need attention too.

### Release notes

Call out accessibility improvements in release notes. It signals commitment and invites feedback from users who rely on these features.

### Onboarding and training

Include accessibility in onboarding materials so new team members learn it’s part of the definition of quality.

### Definition of done

Add **“Accessible”** to your checklist for shipping a feature. It can be as simple as: “Tested with VoiceOver and Dynamic Type.”

### Workshops and lunch‑and‑learns

Share improvements and lessons learned across the team. Short demos of real fixes are often the most effective. Watch *"Convenience for You is Independence for Me"* (WWDC 2017) as a team — Todd Stabelfeldt's story of living with quadriplegia and how apps changed his independence is consistently impactful.

### Code review and QA

Encourage reviewers to flag accessibility issues early. Check out branches and run the app — catching a bug at review time is much faster than after release. Add quick manual tests (VoiceOver, Dynamic Type) to QA smoke checks.

In product discussions and design reviews, keep asking: *"What does this look like at the largest accessibility font size? Are any key actions hidden behind gestures? Do we have copy for all interactive elements?"* A few repetitions and the team will start asking these themselves.

### Hiring and workplace

People with disabilities face barriers in hiring processes and workplace tools. Improving these creates a more inclusive team that builds more inclusive products. Consider adding accessibility knowledge as a requirement or a positive differentiator in iOS developer job descriptions — it raises awareness in the community at scale.

### Audit document

A practical way to get started is to put your headphones on, turn VoiceOver on, and navigate through the most important flows in your app. Document what you find in a shared document — organized by screen or feature. Keep completed fixes rather than deleting them (cross them out or add a ✅). This serves two purposes: it communicates the current state visually (a long list is a clear signal), and it gives teammates a direct starting point when they ask "how can I help?"

Once you have the list, share it in a sprint demo, all-hands, or internal meetup. Show a particularly bad experience, then show it fixed. Most fixes take just a few lines of code. Seeing that directly makes it difficult for anyone to argue it's too complex or expensive to do.

Demo with assistive technologies to raise awareness with the rest of the team on the diversity of ways users interact with products.

### Sustainability and burnout prevention

Accessibility work is long-term. Avoid hero culture and distribute ownership:

- Define realistic iteration goals instead of "fix everything now"
- Celebrate incremental wins
- Keep decision logs so knowledge persists when people move teams

## Checklist for Culture

- [ ] Accessibility considered in design phase
- [ ] Engineers trained on fundamentals
- [ ] Safe space for questions (Slack, office hours)
- [ ] Wins celebrated and contributions recognized
- [ ] Regular manual testing with assistive tech
- [ ] Shared components improved for accessibility
- [ ] Accessibility included in definition of done
- [ ] Accessibility improvements mentioned in release notes

## Sources

- [Fostering an Accessibility Culture (Smashing Magazine)](https://www.smashingmagazine.com/2025/04/fostering-accessibility-culture/)
- [Accessibility Up To 11 — #365DaysIOSAccessibility](https://accessibilityupto11.com/365-days-ios-accessibility/)
- [Accessibility Up To 11 — Resources](https://accessibilityupto11.com/resources/)
