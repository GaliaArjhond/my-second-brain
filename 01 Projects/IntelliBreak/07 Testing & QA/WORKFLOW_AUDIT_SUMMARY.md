# IntelliBreak Core Workflow Audit — Executive Summary

**Date:** 2026-08-27  
**Status:** Audit Complete — Ready for Implementation Decisions

---

## 🎯 WHAT WAS AUDITED

The core employee workflow from start to finish:

```
START SHIFT
    ↓
ACTIVE WORK SESSION
    ↓
WORK TASKS
    ↓
BREAK 1 / BREAK 2 / LUNCH
    ↓
RETURN TO WORK
    ↓
END SHIFT
    ↓
SHIFT LOG / ANALYTICS
```

---

## ✅ WHAT ALREADY WORKS

### 1. Shift Lifecycle (WORKING)

**Frontend:** `lib/features/dashboard/widgets/shift_controller.dart`  
**Backend:** `backend/app/routers/work.py`

- ✅ Start Shift creates work session (`POST /work/session/start`)
- ✅ End Shift closes work session (`POST /work/session/end`)
- ✅ Active session detection (`GET /work/session/active`)
- ✅ Prevents duplicate active sessions
- ✅ Prevents ending shift while break is active
- ✅ Session persists across app restarts
- ✅ Guest/demo mode works offline

**Database:** `WorkSession` table with `started_at`, `ended_at`, `profile_id`

### 2. Break Lifecycle (WORKING)

**Frontend:** `lib/features/dashboard/widgets/shift_controller.dart`  
**Backend:** `backend/app/routers/work.py`

- ✅ Start Break 1/2/Lunch (`POST /work/break/start`)
- ✅ End Break (`POST /work/break/end/{break_id}`)
- ✅ Active break detection (`GET /work/break/active`)
- ✅ **Sequence enforcement:** BREAK_2 requires BREAK_1, LUNCH requires BREAK_2
- ✅ **Duplicate prevention:** Cannot start same break type twice
- ✅ **Overlap prevention:** Cannot start new break while one is active
- ✅ Session breaks history (`GET /work/session/{session_id}/breaks`)
- ✅ Break ownership validation

**Database:** `Break` table with `work_session_id`, `break_type`, `started_at`, `ended_at`

### 3. Work Tasks (MOSTLY WORKING)

**Frontend:** `lib/features/dashboard/widgets/work_tasks.dart`  
**Backend:** `backend/app/routers/work.py`

- ✅ Create Task (`POST /work/tasks`)
- ✅ Update Task Status (`PATCH /work/tasks/{task_id}`)
- ✅ Get Active Session Tasks (`GET /work/tasks/active`)
- ✅ **Requires active session** before creating task
- ✅ Tasks correctly associated with work session
- ✅ Task ownership validation
- ✅ Demo mode works

**Database:** `Task` table with `work_session_id`, `title`, `description`, `status`, `priority`, `estimated_minutes`, `actual_minutes`

**⚠️ ISSUE FOUND:** Frontend sends `category` field, but backend schema (`TaskCreateRequest`) does not accept it. Backend Task model has no `category` field.

---

## ❌ WHAT'S MISSING

### 1. Shift Logs (NOT IMPLEMENTED)

**Status:** Directory exists (`lib/features/shift_logs/`), but:
- ❌ No screen implementation
- ❌ No backend endpoint for shift history
- ❌ Sidebar navigation does nothing

**What's Needed:**
- Backend: `GET /work/sessions/history` endpoint
  - Return completed sessions (`ended_at IS NOT NULL`)
  - Include session duration
  - Include break summaries (count, total break time)
  - Include task count
- Frontend: `ShiftLogsScreen` widget to display history

### 2. Analytics Backend (NOT IMPLEMENTED)

**Status:** Frontend widgets exist and work in demo mode, but:
- ❌ No `backend/app/routers/analytics.py`
- ❌ No analytics endpoints

**What's Needed:**
- `POST /analytics/today/focus-ratio` → Returns `{activeFocus, nonActive, idle}`
- `GET /analytics/weekly/focus-idle` → Returns 7-day focus vs idle data
- `GET /analytics/weekly/work-hours` → Returns 7-day actual vs target hours
- `GET /analytics/focus-rate` → Returns `{rate: 0.0-1.0}`

**Frontend Ready:**
- ✅ `TodayFocusRatioChart` widget
- ✅ `DailyWorkHoursBarChart` widget
- ✅ `FocusIdleLineChart` widget
- ✅ `FocusRateProgress` widget
- ✅ `AnalyticsService` with API calls

### 3. Focus Rate Formula (NOT FINALIZED)

**Status:** 
- ✅ Frontend displays focus rate from backend
- ✅ No hardcoded formula (good — backend-only)
- ❌ Backend does not calculate focus rate yet
- ⚠️ **Adviser has NOT finalized the KPI formula or indicator weights**

**Available Data:**
- Work session duration
- Break compliance (BREAK_1 → BREAK_2 → LUNCH sequence)
- Task completion rate
- Time estimation accuracy (`actual_minutes` vs `estimated_minutes`)
- Idle time calculation

**Recommendation:** Prepare configurable KPI system where weights can be adjusted later.

### 4. Analytics Screen (NOT FOUND)

**Status:**
- ✅ Sidebar lists "Analytics" (index 4)
- ❌ No `AnalyticsScreen` widget found
- ❌ Navigation does not route to analytics screen

**What's Needed:**
- Create `lib/features/analytics/screens/analytics_screen.dart`
- Compose the existing analytics widgets into a full screen
- Add navigation in `dashboard_screen.dart`

---

## ⚠️ ISSUES TO FIX

### Issue 1: Task `category` Field Mismatch

**Location:** `lib/features/dashboard/widgets/work_tasks.dart:798-800`

**Problem:**
- Frontend sends: `{'title': title, 'category': selectedCategory}`
- Backend schema (`TaskCreateRequest`): Does NOT accept `category`
- Backend model (`Task`): Has NO `category` field

**Solutions:**
1. **Option A:** Add `category` field to backend
   - Add to `Task` model in `models.py`
   - Add to `TaskCreateRequest` in `schemas/work.py`
   - Create Alembic migration
2. **Option B:** Remove `category` from frontend
   - Remove dropdown from UI
   - Remove from API payload

**Recommendation:** Choose Option A or B based on whether task categorization is a requirement.

### Issue 2: `actual_minutes` Tracking Unclear

**Location:** Database `Task` model has `actual_minutes` field

**Questions:**
- How is `actual_minutes` populated?
- Is it manual input by the user?
- Is it automatic tracking during task execution?
- Should it auto-update when status changes to `DONE`?

**Current State:**
- Frontend displays `actual_minutes` (line 731 in `work_tasks.dart`)
- Backend accepts `actual_minutes` in updates
- **No automatic tracking logic found**

**Recommendation:** Clarify tracking approach before implementing Shift Logs or Analytics.

---

## 📋 IMPLEMENTATION PRIORITIES

### Priority 1: Fix Existing Issues

1. **Resolve Task `category` Mismatch**
   - Decision needed: Add to backend or remove from frontend?
2. **Clarify `actual_minutes` Tracking**
   - Manual input or automatic tracking?

### Priority 2: Complete Core Workflow

3. **Implement Shift Logs**
   - Backend: `GET /work/sessions/history` endpoint
   - Frontend: `ShiftLogsScreen` widget
   - This allows employees to see their completed shifts

### Priority 3: Enable Analytics

4. **Implement Analytics Backend**
   - Create `backend/app/routers/analytics.py`
   - Implement 4 analytics endpoints
   - Define data aggregation logic from work sessions

5. **Create Analytics Screen**
   - Compose existing widgets
   - Add navigation

### Priority 4: KPI System (After Adviser Input)

6. **Wait for Focus Rate Formula**
   - Adviser must finalize KPI indicators and weights
7. **Implement Configurable KPI Calculation**
   - Allow weight adjustments without code changes

---

## 🧪 VERIFICATION STEPS COMPLETED

✅ **Code Inspection:**
- Read `WorkService` implementation
- Read `ShiftController` implementation  
- Read `work.py` backend routes
- Read `models.py` database schema
- Read `schemas/work.py` API schemas
- Verified `allowNotFound` handling for 404 responses

✅ **Static Analysis:**
- Ran `flutter analyze` — 25 warnings (unused variables, style issues), no blockers

---

## 🚫 WHAT TO AVOID

**DO NOT:**
1. Redesign the dashboard (mobile UI is complete)
2. Invent backend endpoints (use what exists)
3. Invent the final Focus Rate formula (wait for adviser)
4. Change authentication unless required by a bug
5. Modify database schema without migration

---

## ✅ NEXT STEPS

**Before implementing fixes, clarify:**

1. **Task `category`:** Add to backend or remove from frontend?
2. **`actual_minutes`:** How should it be tracked?
3. **Shift Logs priority:** Should this be implemented before Analytics?
4. **Analytics priority:** When should backend analytics be implemented?
5. **Focus Rate formula:** When will adviser finalize the KPI weights?

**After clarification, implement in this order:**
1. Fix task schema issue
2. Implement Shift Logs (if priority 1)
3. Implement Analytics backend (if priority 2)
4. Wait for KPI formula finalization
5. Implement configurable Focus Rate calculation

---

## 📊 COMPLETION STATUS

| Component | Status | Notes |
|-----------|--------|-------|
| Shift Lifecycle | ✅ Working | Start, End, Active Detection |
| Break Lifecycle | ✅ Working | Sequence enforcement, duplicate prevention |
| Work Tasks | ⚠️ Mostly Working | Category field mismatch |
| Shift Logs | ❌ Not Implemented | Backend + frontend missing |
| Analytics Backend | ❌ Not Implemented | 4 endpoints needed |
| Analytics Screen | ❌ Not Implemented | Screen widget missing |
| Focus Rate Formula | ⚠️ Not Finalized | Waiting on adviser |
| Demo Mode | ✅ Working | All features work offline |

---

## 📁 REFERENCE FILES

**Audit Report:** `CORE_WORKFLOW_AUDIT.md` (detailed findings)  
**This Summary:** `WORKFLOW_AUDIT_SUMMARY.md`

**Key Source Files:**
- Frontend: `lib/features/dashboard/widgets/shift_controller.dart`
- Frontend: `lib/features/dashboard/widgets/work_tasks.dart`
- Frontend: `lib/features/dashboard/services/work_service.dart`
- Backend: `backend/app/routers/work.py`
- Backend: `backend/app/models.py`
- Backend: `backend/app/schemas/work.py`

---

**Audit Complete. Ready for implementation decisions.**
