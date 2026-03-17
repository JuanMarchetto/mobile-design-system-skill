# Mobile Design System Skill

A comprehensive Claude Code skill for mobile UI/UX design covering React Native components, touch interactions, gesture patterns, accessibility, app navigation, and visual design principles.

## Overview

This skill consolidates mobile design knowledge into a single, structured system. It covers everything from touch target sizing to emotional design principles, with implementation-ready React Native patterns.

## Features

- **Core Design Principles**: Touch targets, thumb zones, spacing grids, color systems, typography constraints
- **React Native Patterns**: Components, animations (Animated + Reanimated), platform-specific code, accessibility
- **Touch Interactions**: Tap, swipe, pinch-to-zoom, long press, drag-and-drop
- **Navigation**: Tab bars, drawer navigation, bottom sheets, stack navigation, modals
- **Visual Design Process**: 5-step framework from context to polish, 60/30/10 color rule, 8-point grid
- **Industry Conventions**: Design languages for AI, crypto, finance, health, education, fitness, and more
- **Emotional Design**: Peak-End Rule, emotional feedback loops, strategic principles from Spotify
- **Accessibility**: WCAG compliance, VoiceOver/TalkBack, Dynamic Type, contrast requirements
- **Performance**: Image optimization, skeleton screens, Core Web Vitals, PWA patterns

## When to Use

This skill activates when you:
- Design or build mobile app screens
- Implement React Native UI components
- Review mobile UX patterns
- Work with iOS or Android platform conventions
- Optimize mobile performance or accessibility
- Create mobile-first responsive web apps

## File Structure

```
mobile-design-system-skill/
├── SKILL.md                           # Core principles, when to use, quick reference
├── references/
│   ├── react-native-patterns.md       # React Native components, animations, accessibility
│   ├── ux-patterns.md                 # Touch gestures, navigation, mobile UI components, performance
│   ├── design-process.md              # 5-step design framework, visual rules, smart patterns
│   └── industry-conventions.md        # Industry-specific design languages, emotional design
├── .claude-plugin/plugin.json
├── README.md
├── LICENSE
└── .gitignore
```

## Install

```
/plugin marketplace add JuanMarchetto/agent-skills
/plugin install mobile-design-system@agent-skills
```

Or via [skills.sh](https://skills.sh):
```bash
npx skills add JuanMarchetto/mobile-design-system-skill
```

Or manually:
```bash
git clone https://github.com/JuanMarchetto/mobile-design-system-skill.git
cp -r mobile-design-system-skill ~/.claude/skills/mobile-design-system
```

## Design Principles at a Glance

| Principle | Rule |
|-----------|------|
| Touch Targets | 44pt minimum (iOS), 48dp minimum (Android) |
| Spacing | 4px/8px grid: 4, 8, 12, 16, 24, 32, 48 |
| Color | 60% neutral, 30% complementary, 10% accent |
| Typography | 1 font family, max 4 sizes, max 2 weights |
| Contrast | 4.5:1 for normal text (WCAG AA) |
| Animations | 200-300ms, native driver, 60fps |
| Font Size | 16px minimum on web (prevents iOS zoom) |

## License

MIT License
