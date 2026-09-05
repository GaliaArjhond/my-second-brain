# IntelliBreak Dashboard Redesign Plan

## Current State Analysis

### Widget Hierarchy in DashboardScreen
1. GreetingSection
2. ShiftController
3. WorkTasks
4. FocusRateCard
5. WorkTimeCard
6. BreaksCard
7. BreaksHistoryCard
8. SessionHistoryCard
9. TodaysFocusSection
10. CurrentTaskSection
11. AnalyticsSection
12. WorkHourTrendsCard

### Current Issues
- Single column layout wastes horizontal space on desktop
- Most widgets use heavy card containers with excessive padding (24px)
- Significant vertical spacing between widgets (16-24px)
- Information density is low
- No clear visual hierarchy
- Widgets don't adapt well to horizontal space

## Proposed Solution

Following the user's specified information hierarchy:

### Desktop Layout (>=1100px width)
1. **Greeting/header** - Compact, top-aligned
2. **Full-width compact Shift Controller** - Horizontal button layout
3. **Compact metrics row** - FocusRate, WorkTime, and key metrics in horizontal row
4. **Two-column primary workspace**:
   - Left column: Today's Tasks (WorkTasks)
   - Right column: Current Task (CurrentTaskSection)
5. **Two-column recent activity**:
   - Left column: Recent Breaks (BreaksCard + compact history)
   - Right column: Session History (compact SessionHistoryCard)
6. **Bottom analytics area**:
   - Left: Work Hour Trends (WorkHourTrendsCard)
   - Right: Analytics summary (AnalyticsSection)

### Mobile Layout (<1100px width)
1. **Greeting/header** - Top priority
2. **Shift Controller** - Full width, prominent
3. **Current Task** - Immediate focus
4. **Today's Tasks** - Quick access
5. **Important metrics** - Focus rate and work time
6. **Break controls** - Easy access to break buttons
7. **History/analytics** - Stacked at bottom, less frequent access

## Widget-Specific Changes

### Widgets Needing Only Layout Changes
- **GreetingSection** - Reduce vertical padding, keep core functionality
- **ShiftController** - Transform vertical button layout to horizontal on desktop
- **WorkTasks** - Reorganize internal sections to use horizontal space better
- **TodaysFocusSection** - Convert vertical list to more compact horizontal or grid layout
- **CurrentTaskSection** - Reduce padding, optimize progress bar display
- **AnalyticsSection** - Reduce card-like container usage, optimize stat cards
- **WorkHourTrendsCard** - Reduce padding, optimize chart container

### Widgets Needing Internal UI Restructuring
- **FocusRateCard** - Transform from tall card to compact metric panel
- **WorkTimeCard** - Transform from tall card to compact metric panel
- **BreaksCard** - 
  - Desktop: Horizontal layout with break timers side-by-side
  - Keep circular progress indicators or simple text displays
  - Reduce container padding significantly
- **BreaksHistoryCard** - Transform from card-based history to compact list/table
- **SessionHistoryCard** - Transform from card-based history to compact list/table

## Specific Implementation Guidelines

### Spacing & Layout
- Use 8-12px border radii (down from 16px)
- Reduce internal widget padding from 24px to 8-16px
- Use Content-sized spacing rather than fixed SizedBox values
- Avoid nested containers that create visual card stacking
- Use dividers, lists, and panels instead of multiple nested cards

### Visual Design
- Establish hierarchy through typography (size, weight) not container size
- Green only for active/healthy/completed states
- Avoid shadows, gradients, glowing borders, decorative illustrations
- Use restrained color accents from existing theme
- Maintain accessibility patterns (icon + label + color)

### Responsive Behavior
- Desktop: Multi-column, information-dense, aggressive use of horizontal space
- Mobile: Priority stack - Shift Controller → Current Task → Today's Tasks → Metrics → Break controls → History/analytics
- Do not merely shrink desktop layout on mobile

### Preserved Functionality
- All existing widget functionality maintained
- No changes to backend APIs, database, authentication, TOTP
- WorkService behavior unchanged
- Focus Rate calculation preserved
- Team View functionality untouched
- All user interactions and workflows preserved

## Implementation Approach

1. Start with DashboardScreen layout restructuring
2. Modify each widget according to its restructuring needs
3. Implement responsive breakpoints (1100px for desktop)
4. Test desktop layout for information density and workflow preservation
5. Test mobile layout for priority access and usability
6. Verify no API/service logic was changed
7. Run Flutter analyzer to ensure code quality