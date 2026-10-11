# Accessibility Overview

## I. What Is Web Accessibility?

Web accessibility (a11y) ensures that **people with disabilities can perceive, understand, navigate, and interact** with web applications. It's not optional — it's a legal requirement (ADA, EU Accessibility Act, Section 508) and a fundamental quality attribute of any professional frontend application.

**The acronym "a11y":** "accessibility" has 11 letters between "a" and "y" — hence "a11y."

## II. Types of Disabilities

| Type | Examples | How It Affects Web Use |
|------|----------|----------------------|
| **Visual** | Blindness, low vision, color blindness | Screen readers, magnification, contrast sensitivity |
| **Motor** | paralysis, tremor, repetitive strain | Keyboard navigation, switch devices, voice control |
| **Cognitive** | Dyslexia, ADHD, autism | Clear layout, consistent navigation, plain language |
| **Auditory** | Deafness, hard of hearing | Captions, transcripts, visual alerts |
| **Temporary** | Broken arm, eye infection, bright sunlight | One-handed use, readability, audio alternatives |

## III. The Four Principles (POUR)

WCAG (Web Content Accessibility Guidelines) defines four foundational principles:

| Principle | Definition | Example |
|-----------|-----------|---------|
| **Perceivable** | Users can perceive the content | Alt text for images, captions for video, sufficient contrast |
| **Operable** | Users can operate the interface | Keyboard navigation, no seizure-inducing content, adequate time |
| **Understandable** | Users can understand the content | Clear language, predictable behavior, error prevention |
| **Robust** | Works with assistive technologies | Valid HTML, ARIA attributes, semantic markup |

## IV. WCAG Levels

| Level | Requirement | Who It Applies To |
|-------|------------|-------------------|
| **A** | Minimum accessibility | Legal minimum in many jurisdictions |
| **AA** | Standard accessibility | Industry standard; required by most laws (ADA, EU) |
| **AAA** | Enhanced accessibility | Ideal but rarely fully achievable |

**Target: WCAG 2.1 AA** — this is the industry standard and the level most legal requirements reference.

## V. Common Accessibility Barriers

| Barrier | Impact | Solution |
|---------|--------|----------|
| Images without alt text | Screen readers can't describe images | Add meaningful `alt` attributes |
| No keyboard navigation | Motor-impaired users can't operate UI | Ensure all interactive elements are focusable |
| Low color contrast | Low-vision users can't read text | Meet 4.5:1 contrast ratio for normal text |
| No form labels | Screen readers can't identify inputs | Use `<label>` elements linked to inputs |
| Missing ARIA | Assistive tech can't understand dynamic UI | Use correct ARIA roles and attributes |
| No skip navigation | Keyboard users must tab through entire nav | Add skip-to-content link |
| Auto-playing media | Deaf/blind users miss content | No autoplay; provide controls |
| Time-limited sessions | Cognitive/motor users lose data | Extendable time limits |

## VI. Accessibility in the Development Lifecycle

```
Design → Develop → Test → Deploy → Monitor
   │         │        │       │        │
   ▼         ▼        ▼       ▼        ▼
Color     Semantic  axe     CI a11y   User
contrast  HTML,     DevTools checks  feedback
mockups   keyboard  manual           analytics
          support   testing
```

**Shift-left a11y:** Catch issues early — design for accessibility, not retrofit.

## VII. Legal Requirements

| Law | Jurisdiction | Standard |
|-----|-------------|----------|
| **ADA** (Americans with Disabilities Act) | United States | WCAG 2.0/2.1 AA (via case law) |
| **Section 508** | US Federal | WCAG 2.0 AA |
| **European Accessibility Act** | EU | EN 301 549 (references WCAG 2.1 AA) |
| **AODA** | Ontario, Canada | WCAG 2.0 AA |

**Key point:** Accessibility lawsuits have increased dramatically. Companies like Domino's, Apple, and Beyoncé's website have been sued for accessibility violations.

## VIII. Key Takeaways

1. **Accessibility is not optional** — it's a legal requirement and quality attribute
2. **Target WCAG 2.1 AA** — the industry standard for compliance
3. **POUR** — Perceivable, Operable, Understandable, Robust
4. **Shift-left** — design for accessibility, don't retrofit
5. **Automated tools catch ~30%** of issues; manual testing is essential for the rest

## References

- [WCAG 2.1 Guidelines](https://www.w3.org/TR/WCAG21/)
- [WebAIM: Web Accessibility in Mind](https://webaim.org/)
- [A11Y Project](https://www.a11yproject.com/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
