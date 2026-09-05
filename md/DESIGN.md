---
name: IntelliBreak
description: A productivity app for managing work sessions, breaks, and focus tracking
colors:
  primaryBackground: "#F8F9FA"  # lightPrimaryBackground
  secondaryBackground: "#FFFFFF"  # lightSecondaryBackground
  primaryText: "#171923"  # lightPrimaryText
  secondaryText: "#6B7280"  # lightSecondaryText
  borders: "#E0E0E0"  # lightBorders
  accent: "#6C63FF"  # lightAccent/darkAccent
typography:
  display:
    fontSize: "24px"
    fontWeight: 700
  headline:
    fontSize: "18px"
    fontWeight: 700
  title:
    fontSize: "16px"
    fontWeight: 600
  body:
    fontSize: "14px"
    fontWeight: 500
  label:
    fontSize: "12px"
    fontWeight: 400
rounded:
  xs: "4px"
  sm: "6px"
  md: "8px"
  lg: "12px"
  xl: "16px"
spacing:
  xs: "4px"
  sm: "8px"
  md: "12px"
  lg: "16px"
  xl: "20px"
  xxl: "24px"
components:
  dashboard-card:
    backgroundColor: "{colors.secondaryBackground}"
    rounded: "{rounded.md}"
    padding: "{spacing.md}"
    border: "1px solid {colors.borders}"
  elevated-card:
    backgroundColor: "{colors.secondaryBackground}"
    rounded: "{rounded.md}"
    padding: "{spacing.md}"
    border: "1px solid {colors.borders}"
    boxShadow: "0 2px 4px rgba(0,0,0,0.05)"
  section-header:
    paddingBottom: "{spacing.sm}"
  stat-card-compact:
    backgroundColor: "{colors.secondaryBackground}"
    rounded: "{rounded.md}"
    padding: "{spacing.md}"
    border: "1px solid {colors.borders}"
  metric-indicator-compact:
    dotSize: "4px"
    dotSpacing: "2px"
---
# Design System: IntelliBreak

## Overview

**Creative North Star: "The Focused Professional"**

IntelliBreak's design system is built around the concept of productive focus and intentional breaks. The visual language emphasizes clarity, accessibility, and purposeful interaction, supporting users in managing their work rhythms effectively. The system uses a clean, modern aesthetic with sufficient spacing and typographic hierarchy to reduce cognitive load while maintaining professional appeal.

**Key Characteristics:**
- Clean and minimal interface with purposeful elements
- Strong visual hierarchy guiding attention to primary actions
- Adaptive spacing and layout for different screen sizes
- Accessible color contrast and touch targets
- Consistent component styling for predictability

## Colors

The color system supports light, dark, and high contrast themes, with the light theme serving as the baseline. Colors are designed to provide sufficient contrast for readability while maintaining a professional, calm aesthetic.

### Primary Background
- **#F8F9FA**: Used for the main app background in light theme, providing a clean, neutral canvas that reduces visual noise

### Secondary Background
- **#FFFFFF**: Used for cards, containers, and elevation elements, creating a clear layering system

### Primary Text
- **#171923**: Used for primary content and headings, offering high contrast against light backgrounds

### Secondary Text
- **#6B7280**: Used for secondary content, labels, and supporting information, providing medium emphasis

### Borders
- **#E0E0E0**: Used for subtle dividers, card borders, and input outlines, creating definition without visual heaviness

### Accent
- **#6C63FF**: Used for interactive elements, primary buttons, and focus states, providing a vibrant yet professional accent color

## Typography

The typography system uses a clear hierarchy to guide reading and comprehension. Font weights and sizes are carefully calibrated to create visual distinction while maintaining readability across different screen sizes.

**Display Font:** System font (platform-appropriate)
**Body Font:** System font (platform-appropriate)
**Label/Mono Font:** System font (platform-appropriate)

**Character:** Clean, readable, and professional with clear hierarchy that supports both quick scanning and focused reading

### Hierarchy
- **Display** (700, 24px): Used for major section headers and screen titles
- **Headline** (700, 18px): Used for important section headings and key metrics
- **Title** (600, 16px): Used for card titles and prominent labels
- **Body** (500, 14px): Used for primary content and paragraph text
- **Label** (400, 12px): Used for captions, form labels, and secondary text

## Layout

IntelliBreak employs a responsive layout system that adapts to different screen sizes while maintaining consistent information density and usability. The system uses a 4px-based spacing scale for consistency and rhythm.

The dashboard features a persistent sidebar on desktop (≥1100px width) that collapses to a drawer on mobile. Content is organized into logical sections with clear visual hierarchy, using the spacing system to create breathing room between elements. Grid-based alignment ensures consistency across different layouts.

## Elevation & Depth

The design system uses subtle elevation to create visual layering and hierarchy. Surfaces are generally flat at rest, with elevation applied as a response to state (hover, focus, pressed) or to distinguish container elements from the background.

### Shadow Vocabulary
- **Ambient-low** (`box-shadow: 0 2px 4px rgba(0,0,0,0.05)`): Used for cards and containers to provide subtle lift from the background
- **Focus-ring** (`box-shadow: 0 0 0 2px {colors.accent}`): Applied to interactive elements when focused or hovered

## Shapes

The shape language emphasizes rounded corners to create a friendly, approachable aesthetic while maintaining professionalism. Corner radii follow the spacing scale for consistency.

**Corner/Radius Strategy:** All interactive containers and cards use rounded corners from the spacing system, with smaller radii (4px-6px) for tight spaces and larger radii (12px-16px) for prominent elements.

**Border Treatment:** Borders are subtle and consistent, using the borders color at 1px weight for definition without visual heaviness.

## Components

### Buttons
- **Shape:** Rounded with 8px radius (md)
- **Primary:** Background {colors.accent}, text {colors.secondaryBackground}, padding 16px 48px
- **Hover / Focus:** Background opacity reduced to 90%, with focus ring
- **Secondary / Ghost:** Background transparent, text {colors.accent}, border 1px solid {colors.accent}

### Cards / Containers
- **Corner Style:** 8px radius (md) for standard cards, 12px radius (lg) for elevated cards
- **Background:** {colors.secondaryBackground}
- **Shadow Strategy:** Refer to Elevation & Depth section
- **Border:** 1px solid {colors.borders} for dashboard cards
- **Internal Padding:** 16px (md) standard, adjustable based on content density

### Navigation
- **Style:** Icon-only sidebar with tooltips for description, text labels available on hover
- **Typography:** Uses label style (12px, 400 weight)
- **Default/Hover/Active States:** Icon color shifts from {colors.secondaryText} to {colors.accent} on active/hover
- **Mobile Treatment:** Collapses to bottom navigation drawer with icon + label

### Metric Indicators
- **Shape:** Circular progress dots with 4px size and 2px spacing
- **Color:** Filled dots use {colors.accent}, empty dots use {colors.borders} at 30% opacity
- **Label:** Uses title style (16px, 600 weight)
- **Value:** Uses headline style (18px, 700 weight)

## Do's and Don'ts

### Do:
- **Do** use the 4px spacing system (xs: 4px, sm: 8px, md: 12px, lg: 16px, xl: 20px, xxl: 24px) for consistent rhythm
- **Do** apply rounded corners consistently using the radius system (xs: 4px, sm: 6px, md: 8px, lg: 12px, xl: 16px)
- **Do** use {colors.accent} for primary actions and interactive elements
- **Do** maintain sufficient touch targets (minimum 48x48px) for interactive elements
- **Do** use elevation sparingly to create visual hierarchy without clutter

### Don't:
- **Don't** use hardcoded color values outside of the defined color system
- **Don't** mix different radius values within the same component family
- **Don't** reduce spacing below 4px as it compromises accessibility and readability
- **Don't** use the accent color for large background areas as it overwhelms the interface
- **Don't** create custom shadow values outside of the defined shadow vocabulary