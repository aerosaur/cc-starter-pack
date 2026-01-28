---
name: ui-design-system
description: UI design system toolkit for Senior UI Designer including design token generation, component documentation, responsive design calculations, and developer handoff tools. Use when creating design systems, defining design tokens (colors, spacing, typography), documenting components, building component libraries, ensuring visual consistency, or facilitating design-dev handoff. Triggers on mentions of Figma, Storybook, design specs, or component APIs.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# UI Design System

Professional toolkit for creating and maintaining scalable design systems.

## Core Capabilities
- Design token generation (colors, typography, spacing)
- Component system architecture
- Responsive design calculations
- Accessibility compliance
- Developer handoff documentation

## Key Scripts

### design_token_generator.py
Generates complete design system tokens from brand colors.

**Usage**: `python scripts/design_token_generator.py [brand_color] [style] [format]`
- Styles: modern, classic, playful
- Formats: json, css, scss

**Features**:
- Complete color palette generation
- Modular typography scale
- 8pt spacing grid system
- Shadow and animation tokens
- Responsive breakpoints
- Multiple export formats
