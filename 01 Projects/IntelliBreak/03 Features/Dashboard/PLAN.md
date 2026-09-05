# IntelliBreak Dashboard Regression Fix Plan

## Problem Analysis
The current dashboard implementation has regressed from the approved visual structure:
- Missing persistent sidebar on desktop (widgets/dashboard_sidebar.dart exists but not used)
- DashboardScreen uses CustomScrollView with SliverToBoxAdapter creating vertical stack instead of grid layout
- Header exists but layout doesn't match approved structure
- Several structural components missing or misplaced according to approved visual spec

## Required Changes
Modify `dashboard_screen.dart` to implement the approved layout while preserving:
- Guest demo data functionality
- Authentication detection
- All existing widget behaviors and states
- Theme and dark mode support

## Implementation Strategy
1. Replace CustomScrollView with a responsive layout using LayoutBuilder
2. For desktop (≥1100px):
   - Show DashboardSidebar permanently in a Row alongside main content
   - Main content structured in approved grid format
3. For mobile (<1100px):
   - Show DashboardSidebar in drawer
   - Main content full width with appropriate mobile stacking
4. Implement the exact grid structure from approved spec:
   - Top row: AI Wellness Coach (left) | Today's Metrics (right, 2×2 grid)
   - Middle: Shift Controller (left, aligned with AI Wellness) | Work Tasks & Focus Ratio row (right)
   - Bottom: Analytics Section full width
5. Use appropriate flex factors to maintain proportions:
   - Work Tasks ≈ 2/3, Focus Ratio ≈ 1/3
   - AI Wellness Coach and Shift Controller aligned left with same width
6. Preserve all existing widget implementations and their demo/data logic

## Files to Modify
- `lib/features/dashboard/screens/dashboard_screen.dart` (primary)

## Verification Checklist
After implementation, verify at breakpoint widths:
- 1440px: Sidebar visible, header correct, 2×2 metrics, proper alignments
- 1280px: Same as above
- 1024px: Sidebar still visible (desktop breakpoint)
- 390px: Hamburger/drawer, compact header, proper mobile stacking