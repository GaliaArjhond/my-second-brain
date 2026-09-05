---
type: feature
project: IntelliBreak
status: PROPOSED — NOT YET IMPLEMENTED
created: 2026-09-01
tags:
  - feature
  - break-reminders
  - wellness
  - deterministic-rules
  - future-ml
---

# Break Coach Feature Design

## Purpose

Break Coach is a **wellness assistant**, not a productivity maximization or employee-monitoring system.

**Core mission:**
- Remind employees when a break or lunch may be appropriate during a work session
- Encourage sustainable work patterns rather than maximizing task throughput
- Support healthy work-life balance and self-care

**What Break Coach is NOT:**
- Not mandatory break enforcement
- Not employee performance monitoring
- Not a diagnostic system claiming to measure fatigue, stress, or burnout
- Not designed to maximize task completion rates

**UX Language principle:**
Use language like "Consider taking a short break" not "You are fatigued" or "You need rest."

---

## Implementation Status

**Status: PROPOSED**

This design is approved and documented. Implementation has not begun.

No ML model is being built as part of the initial Break Coach launch.

The rule-based initial version will establish the notification patterns, UI component, and settings infrastructure. When ML becomes available (per [[ML Recommendation Methodology]]), it will plug into the decision layer without requiring UI changes.

---

## Architecture Overview

```
Current Work Behavior (Observable Data)
        ↓
Deterministic Rule Engine
        ↓
Break Coach Recommendation
        ↓
Modal Dialog / In-App Notification
        ↓
User Action: Take Break | Take Lunch | Snooze | Continue
        ↓
Existing Break APIs (POST /work/break/start)
```

**Future with ML:**

```
Current Work Behavior
        ↓
Deterministic Rules + ML Prediction (P(work_decline))
        ↓
Decision Layer: Combine signals
        ↓
Break Coach Recommendation (same UI)
```

---

## Rule-Based Trigger Logic

All triggers use **observable session and break data only**. No ML, no internal state assumptions.

### Trigger Rule 1: Long Continuous Work Period

**Condition:**
- `current_session_duration >= 90 minutes` AND
- `time_since_last_break >= 90 minutes`

**Action:**
- Show Break Coach with suggestion type: SHORT_BREAK
- Reason text: "You've been working continuously for [X] minutes. Consider taking a short break before continuing."
- Buttons: [Take Break 1] [Snooze 10 min] [Continue]

**Rationale:**
- 90 minutes aligns with ultradian rhythm research (common work/rest cycles)
- Observable from timestamps only
- No claim about fatigue or recovery need

---

### Trigger Rule 2: Extended Work Without Lunch

**Condition:**
- `current_session_duration >= 180 minutes` AND
- No `Break` record with `break_type = 'LUNCH'` in current session AND
- `current_hour >= 11 AND current_hour < 14` (optional: lunch time window)

**Action:**
- Show Break Coach with suggestion type: LUNCH
- Reason text: "You've been working for [X] minutes without a lunch break. A lunch break might be good now."
- Buttons: [Take Lunch] [Snooze 15 min] [Continue]

**Rationale:**
- Observes actual break behavior, not assumed need
- Suggests during typical lunch hours (optional, can be removed)
- Respects user's break schedule preferences

---

### Trigger Rule 3: After Multiple Short Breaks

**Condition:**
- `count(completed_BREAK_1_and_BREAK_2) >= 2` AND
- `time_since_last_completed_break >= 60 minutes`

**Action:**
- Show Break Coach with suggestion type: SHORT_BREAK
- Reason text: "You've taken [X] breaks recently. Consider another break if you need one."
- Buttons: [Take Break] [Snooze 10 min] [Continue]

**Rationale:**
- Acknowledges ongoing break pattern
- Gives gentle nudge without being prescriptive
- Respects user agency

---

### Trigger Rule 4: End-of-Day Continuity (Optional)

**Condition:**
- `current_hour >= 17 AND current_hour < 18` AND
- `current_session_duration >= 420 minutes` (7+ hours) AND
- No break in last 90 minutes

**Action:**
- Show Break Coach with suggestion type: SHORT_BREAK
- Reason text: "Your work session is approaching typical end-of-day time. Consider wrapping up or taking a final break."
- Buttons: [Take Break] [Continue]

**Rationale:**
- Gentle end-of-shift reminder
- Optional trigger (can be disabled via settings)

---

## Notification / Modal UI

### Visual Design

The Break Coach notification appears as a **modal dialog overlay** above the dashboard.

```
┌─────────────────────────────────────────┐
│ 🧘 Break Coach                    [×]   │
│                                         │
│ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│                                         │
│ Time for a quick reset?                 │
│                                         │
│ You've been working continuously for    │
│ 92 minutes without a recorded break.    │
│ Consider taking a short break.          │
│                                         │
│ [Take Break] [Take Lunch] [Snooze ▼]   │
│ [Continue Working]                      │
│                                         │
│ ℹ️ Why am I seeing this?                │
│ (expandable section)                    │
│                                         │
└─────────────────────────────────────────┘
```

### Components

**Header:**
- Icon: `Icons.psychology_alt_outlined` (or similar wellness icon)
- Title: "Break Coach"
- Close button (X): Dismisses without recording action

**Body:**
- Recommendation title (dynamic): "Time for a quick reset?" / "Lunch break time?" / "Another break soon?"
- Reason text (dynamic): Explains observable trigger
- Call-to-action: Positive, optional language

**Action Buttons:**

1. **[Take Break]**
   - Calls existing API: `POST /work/break/start` with `break_type: 'BREAK_1'`
   - Closes modal
   - Automatically ends active break later
   - Future: Allow user to choose break type (BREAK_1, BREAK_2, SHORT_BREAK)

2. **[Take Lunch]**
   - Calls existing API: `POST /work/break/start` with `break_type: 'LUNCH'`
   - Closes modal
   - User manually ends lunch when done

3. **[Snooze]** (dropdown or submenu)
   - Options: 5 min, 10 min, 15 min, 30 min
   - Stores in local `SharedPreferences`: `break_coach_snoozed_until: DateTime`
   - Suppresses this recommendation until snooze expires
   - Does not suppress other triggers (e.g., if lunch rule would trigger, show it)
   - When session ends, snooze state is cleared

4. **[Continue Working]**
   - Closes modal without starting a break
   - Does NOT record any action
   - Does NOT punish user
   - Communicates recommendation is optional
   - After a certain timeout (e.g., 30 minutes), if conditions still met, may re-trigger

**Expandable Section: "Why am I seeing this?"**
- Title: "Why am I seeing this?"
- Body: Concrete reason based on observable data only
  - Examples:
    - "You've been working continuously for 92 minutes without a recorded break."
    - "Your work session is 185 minutes long and no lunch break is recorded."
    - "You've completed 2 short breaks and 90 minutes have passed since the last one."
- Never claims internal states: "You are fatigued," "You need rest," "Your performance is declining"

### Design System Integration

- Use existing `DesignSystem.premiumCard()` for container
- Use existing design tokens: spacing, radius, colors
- Follow [[Current Thesis Requirements]] accessibility guidelines
- Maintain high contrast support
- Support dark/light themes

### Placement & Lifecycle

**Where it appears:**
- Overlaid above dashboard content in a modal dialog
- Does not block shift controls or break buttons (user can still manually start breaks)
- Appears during active work sessions only
- Never interrupts an active break

**When it triggers:**
- Check rules every 5 minutes during active session
- Trigger when conditions first met
- Do not re-trigger same recommendation within 30 minutes (unless snoozed, then after snooze expires)
- Do not trigger if user has disabled Break Coach in settings

**When it closes:**
- User clicks any action button
- User clicks [×] close button
- Session ends
- Explicit user action: Continue, Take Break, Take Lunch, or Snooze

---

## Existing Backend & API Reuse

Break Coach reuses all existing break infrastructure. **No backend changes required.**

### Break APIs (Already Exist)

From [[Current Thesis Requirements]] work system:

**Start a break:**
```
POST /work/break/start
Body: { "break_type": "BREAK_1" | "BREAK_2" | "LUNCH" | "SHORT_BREAK" | "OTHER" }
Response: { "id": uuid, "break_type": str, "started_at": datetime, ... }
```

**End a break:**
```
POST /work/break/end/{break_id}
Body: {}
Response: { "id": uuid, "break_type": str, "started_at": datetime, "ended_at": datetime, "duration_minutes": int }
```

**Get active break:**
```
GET /work/break/active
Response: { "id": uuid, "break_type": str, "started_at": datetime, "ended_at": null, ... }
Status 404 if no active break
```

**Get breaks in session:**
```
GET /work/session/{session_id}/breaks
Response: [{ "id": uuid, "break_type": str, "started_at": datetime, "ended_at": datetime, "duration_minutes": int }, ...]
```

**Get active session:**
```
GET /work/session/active
Response: { "id": uuid, "started_at": datetime, "ended_at": null, ... }
Status 404 if no active session
```

### Data Already Available

Break Coach needs only:
- `WorkSession.started_at` → calculate session duration
- `WorkSession.id` → query breaks for this session
- `Break.started_at`, `Break.ended_at`, `Break.break_type` → determine last break time and type
- `datetime.now()` → calculate elapsed time

No new database fields required.

---

## Existing Frontend Infrastructure

### Notification Settings (Already Exist)

From `lib/features/dashboard/screens/settings_screen.dart`:

**Settings class:**
```dart
class NotificationSettings {
  static const String enableNotifications = '...enable_notifications';
  static const String breakReminders = '...break_reminders';
  static const String wellnessReminders = '...wellness_reminders';
  static const String breakIntervalKey = '...break_interval'; // default 30 min
  static const String breakDurationKey = '...break_duration'; // default 10 min
}
```

**Break Coach will extend this with:**
- `break_coach_enabled` (toggle)
- `break_coach_snooze_duration` (5, 10, 15, 30 min options)
- (Optional) `break_coach_rule_continuous_work_threshold` (default 90 min)
- (Optional) `break_coach_rule_lunch_threshold` (default 180 min)

### Settings UI (Already Exists)

Settings screen has "Breaks & Wellness" section. Break Coach settings will extend this section.

### Work Service (Already Exists)

From `lib/features/dashboard/services/work_service.dart`:

```dart
class WorkService {
  Future<Map<String, dynamic>> startBreak(String token, String breakType)
  Future<Map<String, dynamic>> endBreak(String token, String breakId)
  Future<Map<String, dynamic>?> getActiveBreak(String token)
  Future<List<dynamic>> getActiveSessionBreaks(String token)
  Future<Map<String, dynamic>?> getActiveSession(String token)
}
```

Break Coach will call these directly. No new service methods needed.

### Theme & Design System

From `lib/core/design_system.dart` — all components available:
- `DesignSystem.premiumCard()`
- `DesignSystem.sectionHeader()`
- `DesignSystem.primaryButton()`
- `DesignSystem.secondaryButton()`
- Color tokens, spacing constants, typography

---

## Frontend Components to Create (Not Yet Implemented)

These components will be built **in a future implementation task**. Design is documented here for reference.

### 1. `break_coach_models.dart`

**Defines data structures:**

```dart
class BreakCoachRecommendation {
  String type; // SHORT_BREAK, LUNCH, etc.
  String title; // "Time for a quick reset?"
  String reason; // "You've been working 92 minutes..."
  String suggestedBreakType; // BREAK_1, LUNCH, etc.
  DateTime generatedAt;
  BreakCoachTrigger trigger; // Rule-based or ML-based
}

enum BreakCoachTrigger {
  ruleContinuousWork,
  ruleLunchTime,
  ruleMultipleBreaks,
  ruleEndOfDay,
  mlPrediction, // Future
}

class BreakCoachSnoozeState {
  DateTime? snoozedUntil;
  BreakCoachRecommendation? originalRecommendation;
}
```

### 2. `break_coach_service.dart`

**Implements rule engine:**

```dart
class BreakCoachService {
  final WorkService _workService;
  
  // Main method: Check if recommendation should trigger
  Future<BreakCoachRecommendation?> getRecommendation(String token)
  
  // Individual rule checks
  Future<bool> _checkContinuousWorkRule(int sessionMinutes, int lastBreakMinutes)
  Future<bool> _checkLunchRule(int sessionMinutes, List<Break> breaks)
  Future<bool> _checkMultipleBreaksRule(List<Break> breaks)
  
  // Future extension point for ML
  Future<BreakCoachRecommendation?> _checkMLPrediction(String token)
}
```

### 3. `break_coach_notification.dart`

**Displays the modal:**

```dart
class BreakCoachNotification extends StatefulWidget {
  final BreakCoachRecommendation recommendation;
  final VoidCallback onTakeBreak;
  final VoidCallback onTakeLunch;
  final ValueChanged<int> onSnooze; // Snooze duration in minutes
  final VoidCallback onContinue;
  final VoidCallback onClose;
}
```

### 4. Modify `shift_controller.dart`

**Add Break Coach check loop:**
- Timer every 5 minutes to poll `BreakCoachService.getRecommendation()`
- Show `BreakCoachNotification` modal when triggered
- Hide modal on user action
- Respect snooze state

### 5. Modify `settings_screen.dart`

**Add Break Coach settings under "Breaks & Wellness":**
- Toggle: "Enable Break Coach"
- Dropdown: "Snooze duration" (5, 10, 15, 30 min)
- Info text: "Break Coach provides wellness reminders during work sessions"

---

## Future ML Integration Point

**Current implementation:** Deterministic rules only.

**ML integration (future, when [[ML Recommendation Methodology]] is implemented):**

The `BreakCoachService` decision layer will accept ML signals:

```dart
// Future extension (not implemented now)
Future<BreakCoachRecommendation?> _checkMLPrediction(String token) async {
  // 1. Extract features from session/break data
  // 2. Call ML model → P(work_pattern_decline) ∈ [0.0, 1.0]
  // 3. If P > 0.7, generate recommendation with trigger = mlPrediction
  // 4. Return recommendation or null
}
```

**In decision layer:**

```dart
Future<BreakCoachRecommendation?> getRecommendation(String token) async {
  // Step 1: Apply deterministic rules
  final ruleRec = await _checkDeterministicRules();
  
  // Step 2: Get ML signal (null for now, active when ML is ready)
  final mlRec = await _checkMLPrediction(token);
  
  // Step 3: Combine (ML enhances, does not replace rules)
  // If ML confidence high, prioritize ML recommendation
  if (mlRec != null && mlRec.trigger == BreakCoachTrigger.mlPrediction) {
    return mlRec;
  }
  
  // Otherwise, use deterministic rule
  return ruleRec;
}
```

**Why this design:**
- UI does not change when ML is added
- Rules are always active (baseline)
- ML is additive (higher confidence, prioritized)
- Maintains observability and explainability
- Aligns with [[ML Recommendation Methodology]] (work pattern decline, not internal state)

---

## Notification System Implementation

### In-App Notification (Primary)

**Approach:**
- Modal dialog overlay (described above)
- No external dependencies
- Works in all browsers, all permission states
- Triggered by polling `BreakCoachService` every 5 minutes

**Why modal:**
- Captures attention without being aggressive
- Allows user to ignore or snooze
- Fits with existing dashboard design
- Does not require browser permissions

### Browser Web Notification API (Future Enhancement)

**Deferred to Phase 2:**
- Not part of initial Break Coach
- Flutter web has no built-in browser notification support
- Would require custom JavaScript interop (`dart:js` / `js_interop` package)
- Optional: If user enables browser notifications in settings, use `Notification.requestPermission()` + `new Notification()`
- Fallback to in-app modal if browser notification fails or permission denied

**For now:** In-app modal only. No browser API complexity.

---

## UX Language Guidelines

### Recommended Language

✅ Use:
- "Consider taking a short break."
- "A break might help you maintain focus."
- "Your work pattern is changing."
- "You've been working [X] minutes without a recorded break."
- "Your current work pattern suggests you may benefit from a break."

### Forbidden Language

❌ Avoid:
- "You are fatigued."
- "You need rest."
- "Your performance is declining."
- "You are stressed."
- "Your wellbeing is at risk."
- "You medically need a break."

**Rationale:** Break Coach is advisory and wellness-focused, not diagnostic or mandatory.

---

## Expected User Workflows

### Workflow 1: User Takes Recommended Break

```
1. User working (92 minutes, no break recorded)
2. Break Coach modal appears: "Time for a quick reset?"
3. User clicks [Take Break]
4. Modal closes
5. Backend: POST /work/break/start → BREAK_1 created
6. User takes 10-minute break
7. User clicks [End Break] (existing UI)
8. Backend: POST /work/break/end → BREAK_1 closed, duration recorded
9. WorkSession continues
10. Break Coach does not trigger again for 30 minutes (or until next rule condition)
```

### Workflow 2: User Snoozes

```
1. Break Coach modal appears
2. User clicks [Snooze] → selects "10 minutes"
3. Modal closes
4. LocalStorage: break_coach_snoozed_until = now + 10 min
5. Break Coach polling checks: `snoozed_until > now` → skip recommendation
6. After 10 minutes, polling checks again: snooze expired, can trigger again
```

### Workflow 3: User Continues Working

```
1. Break Coach modal appears
2. User clicks [Continue Working]
3. Modal closes
4. No break started
5. No action recorded
6. User continues uninterrupted
7. Break Coach respects user's choice (does not immediately re-trigger)
```

### Workflow 4: User Disables Break Coach

```
1. User goes to Settings → Breaks & Wellness
2. Toggles OFF: "Enable Break Coach"
3. Break Coach polling stops
4. No modals appear
5. User can re-enable anytime
```

---

## Settings & Preferences

### New Settings to Add to `NotificationSettings`

```dart
static const String breakCoachEnabled = 'break_coach_enabled'; // default: true
static const String breakCoachSnoozeDefault = 'break_coach_snooze_default'; // default: 10
static const String breakCoachRuleContinuousWorkMinutes = 'break_coach_rule_continuous_work'; // default: 90
static const String breakCoachRuleLunchMinutes = 'break_coach_rule_lunch'; // default: 180
```

### Settings UI Section

**Location:** Settings → Breaks & Wellness

**Fields:**
1. Toggle: "Enable Break Coach"
   - When OFF: no Break Coach modals
   - When ON: polling and recommendations active

2. Dropdown: "Snooze duration"
   - Options: 5 min, 10 min (default), 15 min, 30 min
   - Used when user clicks [Snooze]

3. (Optional) Advanced settings:
   - "Continuous work threshold" (default 90 min)
   - "Lunch reminder threshold" (default 180 min)
   - "End-of-day reminder" (toggle, default ON)

---

## Database & Schema

**NO schema changes required.**

Break Coach uses existing fields:
- `WorkSession.started_at` (timestamp)
- `WorkSession.ended_at` (timestamp, nullable)
- `Break.started_at` (timestamp)
- `Break.ended_at` (timestamp, nullable)
- `Break.break_type` (enum: BREAK_1, BREAK_2, LUNCH, etc.)

**Optional future fields** (not part of initial launch):
- `WorkSession.break_coach_last_triggered_at` (audit trail, informational only)
- `BreakCoachSnoozeLog` (if persistent snooze history is desired)

**For now:** Use `SharedPreferences` for session-scoped snooze state (cleared when session ends).

---

## Success Criteria

Break Coach is successful if:

1. ✅ Recommendations appear only when observable rules trigger
2. ✅ Users can easily take suggested breaks via existing API
3. ✅ Users can snooze without interruption
4. ✅ Users can disable Break Coach entirely
5. ✅ No claims about internal states (fatigue, stress, etc.)
6. ✅ Language is optional, advisory, supportive
7. ✅ Works without ML (rules-only launch)
8. ✅ Extensible for future ML integration
9. ✅ Does not interfere with existing shift/break/task controls

---

## Implementation Roadmap

### Phase 1: Rule-Based Deterministic Version (Proposed)

- [ ] Create `break_coach_models.dart`
- [ ] Create `break_coach_service.dart` (rule engine)
- [ ] Create `break_coach_notification.dart` (modal UI)
- [ ] Modify `shift_controller.dart` (polling loop)
- [ ] Modify `settings_screen.dart` (Break Coach settings)
- [ ] Extend `NotificationSettings` for preferences
- [ ] Test all rule triggers
- [ ] Verify existing break APIs work end-to-end
- [ ] Manual QA: snooze, continue, take break workflows

### Phase 2: Browser Notifications (Future)

- Add `dart:js` interop for browser notification API
- Add settings: "Enable browser notifications"
- Add permission request flow
- Fallback to in-app modal if permission denied

### Phase 3: ML Integration (Future, pending [[ML Recommendation Methodology]])

- Add ML prediction service
- Integrate P(work_decline) into decision layer
- Add ML trigger type to modal reason text
- No UI changes required

---

## Related Notes

- [[Current Thesis Requirements]] — wellness recommendation system requirement
- [[ML Recommendation Methodology]] — future ML integration, Work Pattern Decline Prediction
- [[Break System]] — existing break APIs and data model
- [[Recommendations]] — recommendation engine design
- [[Work Sessions]] — session lifecycle and data
- [[Breaks]] — break lifecycle and data

---

## Status & Next Steps

**Status:** PROPOSED — NOT YET IMPLEMENTED

**Design approval:** ✅ Complete

**Next steps:**
1. User review and feedback on this design
2. Finalize rule thresholds and language
3. Create implementation task for Phase 1
4. Begin frontend component development

---

## Document History

- **2026-09-01** — Initial design document created during audit phase
  - Documented purpose, rules, UI, existing infrastructure
  - Defined components to be created
  - Established ML integration extension point
  - Marked as PROPOSED

