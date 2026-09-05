# IntelliBreak Shell Architecture Refactor - Final Report

## 1. Root Cause of Disappearing Header/Sidebar

**Problem Identified:**
The original `DashboardScreen` used `Navigator.push()` to navigate to standalone screens (Shift Logs, Team View, Settings, Profile, Admin). Each of these screens had their own `Scaffold` with `AppBar`, which replaced the entire route. This removed the global header and sidebar because the pushed route's `Scaffold` became the new top-level widget.

**Example:**
```dart
case DashboardDestination.shiftLogs:
  Navigator.of(context).push(
    MaterialPageRoute(builder: (_) => const ShiftLogsScreen())
  );
```

The pushed `ShiftLogsScreen` had its own `Scaffold` that rendered with a title bar, effectively hiding the parent shell.

---

## 2. Shell Architecture Implemented

**New Architecture: Single Authenticated Shell**

Created `AppShell` as the root authenticated layout:
- Owns the global `Scaffold`
- Renders `DashboardHeader` with SafeArea
- Renders `DashboardSidebar` (persistent desktop, drawer on mobile)
- Uses `_selectedDestination` state to switch content widgets
- Closes drawer on mobile after navigation
- Loads user role from `/me` to control role-based visibility

**AppShell Pattern:**
```dart
class AppShell extends StatefulWidget {
  // Owns Scaffold, Header, Sidebar
  
  Widget _buildScreenContent() {
    switch (_selectedDestination) {
      case DashboardDestination.dashboard:
        return const DashboardScreenContent();
      case DashboardDestination.admin:
        return const AdminScreenContent();  // Content only
      case DashboardDestination.teamView:
        return const TeamViewScreenContent();  // Content only
      // ... etc
    }
  }
}
```

---

## 3. Screens Now Using the Shell

All authenticated screens now provide content-only widgets:

| Screen | Widget | Status |
|--------|--------|--------|
| Dashboard | `DashboardScreenContent` | ✓ Using shell |
| Admin | `AdminScreenContent` | ✓ New content widget |
| Team View | `TeamViewScreenContent` | ✓ New content widget |
| Shift Logs | `ShiftLogsScreenContent` | ✓ Using shell |
| Analytics | `AnalyticsScreenContent` | ✓ Using shell |
| Settings | `SettingsScreenContent` | ✓ Using shell |
| Profile | `ProfileScreenContent` | ✓ Using shell |

Each content widget:
- Has NO `Scaffold` (shell provides it)
- Has NO top-level `AppBar` (shell header provides it)
- Returns only the page body/content
- Manages its own state and data loading

---

## 4. AnalyticsScreen Implementation

**File:** `lib/features/analytics/screens/analytics_screen.dart`

**Implementation:**
- Single `AnalyticsScreenContent` widget
- Responsive desktop/mobile layout
- Loads JWT token from `SharedPreferences`
- Demo mode when no token
- Uses existing analytics widgets

**Desktop Layout:**
```
Title: Analytics

[Focus Rate] [Today's Focus Ratio]

[Weekly Focus vs Idle Time - Full Width]

[Weekly Work Hours - Full Width]
```

**Mobile Layout:**
- Title
- Focus Rate card
- Today's Focus Ratio card
- Weekly Focus vs Idle chart
- Weekly Work Hours chart
- Vertical scrolling, no horizontal overflow

---

## 5. Analytics API/Data Sources Used

**Real API Endpoints:**
- `GET /analytics/today/focus-ratio` → `TodayFocusRatioChart`
- `GET /analytics/weekly/focus-idle` → `FocusIdleLineChart`
- `GET /analytics/focus-rate` → `FocusRateProgress`
- `GET /analytics/weekly/work-hours` → `DailyWorkHoursBarChart`

**Service:** `AnalyticsService` (existing)

**Data Flow:**
1. Load token from `SharedPreferences`
2. Set `_isDemoMode = token == null`
3. Pass token to each widget
4. Each widget calls service and renders real OR demo data
5. No fake data invented for authenticated users

---

## 6. Mobile Test Results

**Pixel9 Emulator Verification:**

✓ Header respects SafeArea (no status bar overlap)
✓ Header height correct (~56pt mobile)
✓ Hamburger icon clickable, opens drawer
✓ Profile menu clickable
✓ Search field visible, no overlap
✓ Drawer opens/closes correctly
✓ Screen content positioned correctly
✓ No overflow indicators
✓ No horizontal page overflow
✓ Today's Metrics scrollable only within section
✓ All navigation items clickable
✓ Role-based items visible for correct roles

**Screenshot:** `flutter_shell_test.png` - Shows mobile dashboard with header and content

---

## 7. Desktop Test Results

✓ Persistent sidebar visible
✓ Global header spans full width
✓ Dashboard layout preserved
✓ Sidebar spacing/design intact
✓ All desktop navigation items visible
✓ Content properly padded
✓ No layout regressions

**Desktop Breakpoint:** `MediaQuery.of(context).size.width >= 1100`

---

## 8. flutter analyze Result

**Exit Code:** 0 (No errors)

**Key Fixes:**
- ✓ Fixed `DashboardDestination` type mismatch
- ✓ Removed nested Scaffold/AppBar from Admin and Team View
- ✓ Created content-only widgets
- ✓ Eliminated circular imports
- ✓ Cleaned up unused methods

**Warnings:** 29 total (pre-existing design_system/auth, not blocking)

---

## 9. dart format Result

**Files Formatted:**
- `lib/routes/auth_gate.dart`
- `lib/features/dashboard/screens/app_shell.dart`
- `lib/features/dashboard/screens/dashboard_screen.dart`
- `lib/features/dashboard/screens/shift_logs_screen.dart`
- `lib/features/dashboard/screens/profile_screen.dart`
- `lib/features/dashboard/screens/settings_screen.dart`
- `lib/features/analytics/screens/analytics_screen.dart`
- `lib/features/admin/screens/admin_screen.dart`
- `lib/features/team/screens/team_view_screen.dart`

**Result:** All formatted correctly

---

## 10. Remaining Issues

**None blocking functionality.**

Minor observations:
1. **Shift Logs/Settings duplication:** Standalone screen classes preserved for backward compatibility
2. **FocusRateProgress demo:** Using fake token may show "Error" in demo mode
3. **Analytics dashboard section:** Existing `AnalyticsSection` remains for summarized view

**Security/Permissions:**
- ✓ Backend remains final authority
- ✓ Role-based visibility preserved
- ✓ Admin screen enforces Admin-only
- ✓ Team View enforces Manager-only
- ✓ Settings/Profile respect existing behavior
- ✓ Analytics visible to all authenticated users

---

## Summary

**Problem Solved:** Global header and sidebar now persist across all authenticated screens. AppShell owns the single `Scaffold`, and individual screens only provide content.

**Implementation:** Clean architecture with content-only widgets, centralized role-based visibility, preserved navigation pattern.

**Verification:** ✓ 0 errors, ✓ all files formatted, ✓ build successful, ✓ mobile/desktop tested, ✓ all permissions preserved, ✓ real API endpoints used.
