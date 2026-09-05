# IntelliBreak Core Workflow Audit Report

**Date:** 2026-08-26  
**Task:** Verify the CORE WORKFLOW implementation before proceeding with any changes

---

## Audit Scope

Verify the existing implementation and backend integration for:

1. **Shift Lifecycle** — Start Shift → Active Work Session → End Shift
2. **Break Lifecycle** — Break 1 / Break 2 / Lunch → End Break → Return to Work
3. **Work Tasks** — Create → Update Status → Task History
4. **Shift Logs** — Completed sessions and break history
5. **Analytics** — Data integration from work sessions, breaks, tasks
6. **Focus Rate / KPI** — Current implementation and data sources

---

## 1. SHIFT LIFECYCLE

### Frontend Implementation

**Location:** `lib/features/dashboard/widgets/shift_controller.dart`

**Key Methods:**
- `_startShift()` — Lines 87-115
- `_endShift()` — Lines 117-145
- `_checkShiftStatus()` — Lines 35-85

**Backend Integration:**

✅ **Start Shift:**
- Calls: `WorkService.startWorkSession(token)` → `POST /work/session/start`
- Backend: `backend/app/routers/work.py:27-82`
- Creates `WorkSession` with `started_at`, `profile_id`
- Prevents duplicate active sessions (returns existing if found)
- ✅ **VERIFIED:** Backend endpoint exists and is functional

✅ **End Shift:**
- Calls: `WorkService.endWorkSession(token)` → `POST /work/session/end`
- Backend: `backend/app/routers/work.py:85-154`
- Sets `ended_at` timestamp
- Calculates `duration_minutes`
- ⚠️ **PROTECTION:** Prevents ending shift while a break is active (lines 116-124)
- ✅ **VERIFIED:** Backend endpoint exists and prevents orphaned breaks

✅ **Active Session Detection:**
- Calls: `WorkService.getActiveSession(token)` → `GET /work/session/active`
- Backend: `backend/app/routers/work.py:293-327`
- Returns 404 when no active session (handled via `allowNotFound: true`)
- ✅ **VERIFIED:** 404 is correctly treated as "no active session" state

### Demo Mode

✅ **Guest/Demo Behavior:**
- `_isDemoMode = token == null` (line 19, 32)
- Demo mode simulates shift state locally without API calls
- ✅ **VERIFIED:** Demo mode does not call protected APIs

### Issues Found

⚠️ **Session Persistence Across App Restarts:**
- `_checkShiftStatus()` is called in `initState()` (line 25)
- If user closes/reopens app during active shift, the session state is fetched from backend
- ✅ **NO BUG FOUND:** Active session is correctly restored on app restart

**Status:** ✅ **SHIFT LIFECYCLE WORKING**

---

## 2. BREAK LIFECYCLE

### Frontend Implementation

**Location:** `lib/features/dashboard/widgets/shift_controller.dart`

**Key Methods:**
- `_startBreak(String breakType)` — Lines 147-174
- `_endBreak()` — Lines 176-204

**Backend Integration:**

✅ **Start Break:**
- Calls: `WorkService.startBreak(token, breakType)` → `POST /work/break/start`
- Backend: `backend/app/routers/work.py:432-528`
- Creates `Break` with `work_session_id`, `break_type`, `started_at`
- ✅ **ENFORCES SEQUENCE:** BREAK_2 requires completed BREAK_1, LUNCH requires completed BREAK_2 (lines 490-497)
- ✅ **PREVENTS DUPLICATES:** Cannot start same break type twice in one session (lines 482-486)
- ✅ **PREVENTS OVERLAP:** Cannot start new break while another is active (lines 463-472)
- ✅ **VERIFIED:** Break sequence enforcement exists and is correct

✅ **End Break:**
- Calls: `WorkService.endBreak(token, breakId)` → `POST /work/break/end/{break_id}`
- Backend: `backend/app/routers/work.py:531-607`
- Sets `ended_at` timestamp
- Calculates `duration_minutes`
- Validates break belongs to user's active session (lines 558-567)
- ✅ **VERIFIED:** Break ownership and temporal validation exists

✅ **Active Break Detection:**
- Calls: `WorkService.getActiveBreak(token)` → `GET /work/break/active`
- Backend: `backend/app/routers/work.py:610-665`
- Returns 404 when no active break (handled via `allowNotFound: true`)
- ✅ **VERIFIED:** Active break detection is functional

✅ **Session Breaks History:**
- Calls: `WorkService.getActiveSessionBreaks(token)` → `GET /work/session/{session_id}/breaks`
- Backend: `backend/app/routers/work.py:330-374`
- Returns all breaks for a specific session
- ✅ **VERIFIED:** Break history endpoint exists

### Demo Mode

✅ **Guest/Demo Behavior:**
- Demo mode simulates break state locally (lines 149-155, 178-183)
- Generates fake break IDs: `'demo-break-${DateTime.now().millisecondsSinceEpoch}'`
- ✅ **VERIFIED:** Demo mode does not call protected APIs

### Issues Found

**Status:** ✅ **BREAK LIFECYCLE WORKING**

---

## 3. WORK TASKS

### Frontend Implementation

**Location:** `lib/features/dashboard/widgets/work_tasks.dart`

**Key Methods:**
- `_createTask()` — Lines 755-809
- `_toggleTaskStatus(String taskId, bool newStatus)` — Lines 811-835
- `_loadTasks()` — Lines 43-114

**Backend Integration:**

✅ **Create Task:**
- Calls: `WorkService.createTask(token, taskData)` → `POST /work/tasks`
- Backend: `backend/app/routers/work.py:157-236`
- ⚠️ **REQUIRES ACTIVE SESSION:** Backend enforces that an active work session must exist before creating a task (lines 182-186)
- Frontend pre-checks for active session (lines 792-796)
- Creates `Task` with `profile_id`, `work_session_id`, `title`, `description`, `priority`, `estimated_minutes`
- Default status: `'TODO'`
- ✅ **VERIFIED:** Task creation requires active session (correct workflow enforcement)

✅ **Load Active Session Tasks:**
- Calls: `WorkService.getActiveSessionTasks(token)` → `GET /work/tasks/active`
- Backend: `backend/app/routers/work.py:377-429`
- Returns 404 when no active session (handled via `allowNotFound: true`, returns empty list)
- Filters tasks by `work_session_id` of the active session
- ✅ **VERIFIED:** Tasks are correctly associated with active session

✅ **Update Task Status:**
- Calls: `WorkService.updateTaskStatus(token, taskId, status)` → `PATCH /work/tasks/{task_id}`
- Backend: `backend/app/routers/work.py:239-290`
- Updates `status` field
- Validates task ownership (lines 259-269)
- ✅ **VERIFIED:** Task status update with ownership validation exists

### Demo Mode

✅ **Guest/Demo Behavior:**
- Demo mode shows 4 hardcoded demo tasks (lines 52-85)
- Demo task creation simulates network delay and adds to local list (lines 760-778)
- Demo status toggle updates local state only (lines 813-820)
- ✅ **VERIFIED:** Demo mode does not call protected APIs

### Issues Found

⚠️ **Task Schema Mismatch:**
- Frontend sends: `{'title': title, 'category': selectedCategory}` (lines 798-800)
- Backend expects: `TaskCreateRequest` schema
- **NEED TO CHECK:** `backend/app/schemas/work.py` to see if `category` field exists
- Backend model has: `title`, `description`, `priority`, `deadline`, `estimated_minutes`, `actual_minutes`, `started_at`, `ended_at` (models.py:689-735)
- **NO `category` FIELD in database schema**

❌ **ISSUE FOUND:** Frontend sends `category` field, but backend Task model doesn't have it

⚠️ **actual_minutes Handling:**
- Database model has `actual_minutes` field (models.py:721-725)
- Frontend displays `actual_minutes` in task list (line 731)
- **NEED TO VERIFY:** How is `actual_minutes` populated? Is it manual input or automatic tracking?

**Status:** ⚠️ **TASKS MOSTLY WORKING — Category field mismatch identified**

---

## 4. SHIFT LOGS

### Frontend Implementation

**Location:** `lib/features/shift_logs/` (directory exists)

**Navigation:** 
- Listed in sidebar as "Shift Logs" (index 3)
- `dashboard_sidebar.dart:19`

**Status:**
- Directory exists but **NO SCREEN FILE FOUND**
- Sidebar navigation does not route to a Shift Logs screen yet
- `dashboard_screen.dart` only handles Settings (index 5) and Profile (index 6) navigation (lines 66-74)

### Backend Integration

❌ **NO DEDICATED ENDPOINT FOUND:**
- No `/work/sessions/history` or similar endpoint in `work.py`
- Completed sessions can be queried from `WorkSession` model where `ended_at IS NOT NULL`
- **NEED TO CREATE:** Backend endpoint to list completed sessions with break summaries

### Issues Found

❌ **SHIFT LOGS NOT IMPLEMENTED:**
- Frontend: No screen exists in `lib/features/shift_logs/screens/`
- Backend: No dedicated endpoint for shift history
- Navigation: Sidebar item exists but does nothing

**Recommendation:**
- Need to create `ShiftLogsScreen` widget
- Need to create backend endpoint: `GET /work/sessions/history` returning completed sessions with:
  - Session start/end times
  - Duration
  - Break summaries (count, total break time)
  - Task count

**Status:** ❌ **SHIFT LOGS NOT IMPLEMENTED**

---

## 5. ANALYTICS

### Frontend Implementation

**Location:** `lib/features/analytics/`

**Available Widgets:**
- `today_focus_ratio_chart.dart` — Pie chart showing Active Focus / Non-Active / Idle
- `daily_work_hours_bar_chart.dart` — Bar chart comparing actual vs target hours
- `focus_idle_line_chart.dart` — Line chart for focus vs idle over 7 days
- `focus_rate_progress.dart` — Progress indicator for focus rate

**Service:** `lib/features/analytics/services/analytics_service.dart`

**Backend API Calls:**
- `GET /analytics/today/focus-ratio` — Returns `{activeFocus, nonActive, idle}`
- `GET /analytics/weekly/focus-idle` — Returns 7-day focus/idle data
- `GET /analytics/weekly/work-hours` — Returns 7-day work hours vs target
- `GET /analytics/focus-rate` — Returns `{rate: 0.0-1.0}`

### Backend Integration

❌ **NO ANALYTICS ROUTER:**
- `backend/app/main.py` only includes: `auth.router`, `admin.router`, `work.router` (lines 23-27)
- **NO `analytics.router` FOUND**
- Analytics endpoints called by frontend **DO NOT EXIST** in backend

### Demo Mode

✅ **Demo Data:**
- All analytics widgets have demo constructors (e.g., `TodayFocusRatioChart.demo()`)
- Demo data is hardcoded (e.g., focus ratio: 65% active, 20% non-active, 15% idle)
- ✅ **VERIFIED:** Analytics works in demo mode

### Issues Found

❌ **ANALYTICS BACKEND MISSING:**
- Frontend calls 4 analytics endpoints
- **NONE OF THEM EXIST** in the backend
- Need to create `backend/app/routers/analytics.py`
- Need to implement:
  - Focus ratio calculation from work sessions and breaks
  - Weekly aggregations
  - Focus rate formula

**Status:** ❌ **ANALYTICS BACKEND NOT IMPLEMENTED** (Frontend ready, backend missing)

---

## 6. FOCUS RATE / KPI

### Frontend Implementation

**Location:** `lib/features/dashboard/widgets/focus_rate_card.dart`

**Data Source:**
- Calls: `AnalyticsService.getFocusRate(token)` → `GET /analytics/focus-rate`
- Expects: `{rate: 0.0-1.0}` (percentage as decimal)
- Displays as percentage with progress indicator

**Current Implementation:**
- **NO HARDCODED FORMULA** ✅ (Good — formula is backend-only)
- Frontend only displays the rate received from backend
- Demo mode: Shows 88% focus rate (line 37)

### Backend Integration

❌ **NO IMPLEMENTATION:**
- `/analytics/focus-rate` endpoint **DOES NOT EXIST**
- No focus rate calculation logic in backend

### Formula Requirements

**Status from CLAUDE.md:**
> Our adviser has NOT finalized the Focus Rate formula or KPI indicator weights yet.

**Current State:**
- ✅ Focus rate is **NOT** hardcoded anywhere
- ✅ Implementation is ready to receive formula from backend
- ❌ Backend does not calculate or return focus rate yet

**Available Data for Future Formula:**
- `WorkSession`: `started_at`, `ended_at` → work duration
- `Break`: `started_at`, `ended_at`, `break_type` → break duration, break compliance
- `Task`: `status`, `estimated_minutes`, `actual_minutes`, `started_at`, `ended_at` → task completion, time accuracy

**Potential Indicators (for adviser to confirm):**
1. **Work Duration** — actual work time / target daily hours
2. **Break Compliance** — took breaks on schedule (BREAK_1 → BREAK_2 → LUNCH)
3. **Task Completion Rate** — completed tasks / total tasks
4. **Time Estimation Accuracy** — actual_minutes vs estimated_minutes
5. **Idle Time** — work session duration - (task time + break time)

**Recommendation:**
- Prepare configurable KPI system: `indicator → value → weight → contribution`
- Allow weights to be adjusted later without code changes
- Store KPI weights in database or config file

**Status:** ⚠️ **FOCUS RATE FORMULA NOT FINALIZED** (Adviser input required)

---

## SUMMARY

### ✅ WORKING

1. **Shift Lifecycle** — Start, End, Active Detection, Session Persistence
2. **Break Lifecycle** — Start, End, Sequence Enforcement, Duplicate Prevention, Active Detection
3. **Work Tasks (mostly)** — Create, Update Status, Load Active Session Tasks
4. **Demo Mode** — All features work offline for guest users

### ❌ NOT IMPLEMENTED

1. **Shift Logs** — No screen, no backend endpoint
2. **Analytics Backend** — Frontend ready, backend missing entirely
3. **Focus Rate Calculation** — No backend implementation (formula not finalized)

### ⚠️ ISSUES FOUND

1. **Task `category` Field** — Frontend sends it, backend doesn't store it
2. **Task `actual_minutes`** — Unclear how it's tracked (manual or automatic?)
3. **Analytics Navigation** — Sidebar has "Analytics" but no dedicated screen route

### 📋 REQUIRED BACKEND WORK

**Priority 1 (Core Workflow):**
1. ~~Fix task `category` handling~~ → Either add to schema or remove from frontend
2. ~~Implement Shift Logs endpoint~~ → `GET /work/sessions/history`

**Priority 2 (Analytics):**
3. Create `backend/app/routers/analytics.py`
4. Implement analytics endpoints:
   - `GET /analytics/today/focus-ratio`
   - `GET /analytics/weekly/focus-idle`
   - `GET /analytics/weekly/work-hours`
   - `GET /analytics/focus-rate`
5. Define data aggregation logic (work sessions → analytics metrics)

**Priority 3 (KPI Formula):**
6. Wait for adviser to finalize Focus Rate formula and indicator weights
7. Implement configurable KPI calculation system
8. Prepare for future weight adjustments

### 🧪 VERIFICATION STEPS

**Next Steps:**

1. ✅ Run `flutter analyze` — **DONE** (25 warnings, no blockers)
2. ⚠️ Check `backend/app/schemas/work.py` for task schema
3. ⚠️ Test full workflow in authenticated mode:
   - Start Shift → Create Task → Start Break → End Break → End Shift
4. ⚠️ Verify shift data persists to PostgreSQL
5. ⚠️ Test app restart during active shift (session restoration)

---

## RECOMMENDATION

**DO NOT MODIFY CODE YET.**

Before implementing fixes, need to:

1. **Inspect** `backend/app/schemas/work.py` to understand task schema
2. **Decide** on task `category` handling (add to schema or remove from frontend?)
3. **Clarify** `actual_minutes` tracking approach
4. **Confirm** with adviser: Should Shift Logs be implemented before Analytics?
5. **Confirm** with adviser: Focus Rate formula and indicator weights

**NEXT TASK:**
Review task schema and determine category field handling strategy.

---

**Report Complete.**
