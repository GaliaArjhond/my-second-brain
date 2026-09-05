# IntelliBreak Requirements Analysis

## Executive Summary
Based on careful analysis of both project documents (Intellibreak_Thesis (2).docx and Susundin_bato.docx), the IntelliBreak system is a web-based Machine Learning-Based Productivity and Work-Life Balance Recommendation System designed specifically for Virtual Assistants and Work-from-Home professionals.

The documents are **complementary with no direct conflicts**. Document 1 provides the conceptual framework, objectives, scope, and theoretical foundation, while Document 2 provides operational role details, specific capabilities, and development tasks.

## 1. Consolidated Requirements

### Present in BOTH Documents:
- User registration and login system
- Task management (checklist-based: create, organize, update, complete)
- Work-hour tracking
- Break time monitoring
- Lunch monitoring/scheduling (explicitly called out in both)
- Machine learning recommendation module
- Personalized recommendations for work-life balance
- Productivity dashboard showing work history, completed tasks, recommendations
- Web-based application (desktop/laptop only)
- Three distinct roles: Admin, Manager, Employee (VA/WFH)
- Manager approval required for employee registration/login
- System only monitors data recorded within the application
- Recommendations only (does not replace performance evaluations)

### Present in ONLY Document 1 (Thesis):
- Inactive period monitoring
- Analysis of overworking/underworking patterns
- ISO/IEC 25010 Software Quality Model evaluation
- Specific ML focus: working hours, completed tasks, break times, lunch schedules, inactive periods
- Work-Life Balance Theory and Self-Determination Theory as theoretical framework
- Explicit exclusions: payroll, attendance monitoring, client communication, video conferencing, project management

### Present in ONLY Document 2 (Susundin_bato):
- Manager can set deadlines and priorities (explicitly stated)
- Admin maintains database and system settings
- Admin views reports and analytics
- Manager views employees' productivity reports
- Manager views employees' work-life balance recommendations (privacy permitting)
- Specific development task list: login/registration, task checklist/dashboard, work-hour/break/lunch tracking, ML module, testing

## 2. Required User Roles and Capabilities

### Admin Role:
- Manage all user accounts
- Manage managers and employees
- Monitor system activities
- View reports and analytics
- Maintain database and system settings

### Manager Role:
- Verify and approve employee accounts
- Reject invalid registrations
- Create and assign tasks
- Set deadlines and priorities
- Monitor employee progress
- View employees' productivity reports
- View employees' work-life balance recommendations (if permitted by privacy policy)

### Employee Role (Virtual Assistant / WFH Professional):
- Register an account
- Log in after manager approval
- View assigned tasks
- Update task progress
- Mark tasks as completed
- Receive personalized recommendations
- View their productivity dashboard
- Monitor their work history

## 3. Required Features/Modules

### Core Features:
1. **User Authentication System**:
   - Registration
   - Login (after manager approval for employees)
   - Password handling (secure hashing implied)

2. **Task Management Module**:
   - Checklist interface
   - Create tasks
   - Organize tasks
   - Update task progress
   - Complete tasks
   - Manager-assigned tasks
   - Task deadlines
   - Task priorities

3. **Work Monitoring Module**:
   - Work-hour tracking (shift start/end times)
   - Break time monitoring
   - Lunch monitoring/scheduling
   - Inactive period monitoring

4. **Machine Learning Module**:
   - Work habit analysis
   - Pattern detection (overworking/underworking)
   - Personalized recommendation generation
   - Based on: working hours, completed tasks, break times, lunch schedules, inactive periods

5. **Dashboard and Reporting**:
   - Productivity dashboard
   - Work history viewing
   - Completed tasks tracking
   - Personalized recommendations display
   - Manager view of employee productivity reports
   - Admin view of system reports/analytics
   - Admin monitoring of system activities

6. **Role-Based Access Control**:
   - Three distinct roles (Admin/Manager/Employee)
   - Manager approval workflow for employee accounts
   - Role-specific UI/access restrictions

## 4. Required Data to Store

### User & Authentication Data:
- User credentials (hashed passwords, never plaintext)
- Email addresses (unique, case-insensitive)
- User roles (Admin/Manager/Employee)
- Account approval status (PENDING/APPROVED/REJECTED)
- User profile information (full name, etc.)
- Account timestamps (created/updated)

### Profile & Organizational Data:
- Job role/position
- Industry
- Years of experience
- Demographic information (age, gender, etc.)
- Target daily hours
- Preferred shift times
- Work days pattern
- Hire date
- Department
- Contact information

### Team & Hierarchy Data:
- Team names and descriptions
- Manager assignments to teams
- Employee-team membership history (with join/leave dates)

### Work Monitoring Data:
- Work sessions (shift start/end timestamps)
- Break periods (start/end timestamps, break type)
- Notes on breaks (optional)

### Task Data:
- Task titles and descriptions
- Task status (TODO/IN_PROGRESS/DONE/BLOCKED)
- Task priority (LOW/MEDIUM/HIGH)
- Task deadlines
- Task estimated vs actual time
- Task association with work sessions (nullable for pre/post-session tasks)
- Task timestamps (created/updated)

### Machine Learning Data:
- Generated recommendations (text content)
- Recommendation types/categories
- Recommendation generation timestamps
- Recommendation view status (viewed/unviewed)

### System Monitoring Data:
- Admin/system actions (for audit trail)
- Action types and details
- IP addresses and user agents
- Action timestamps

## 5. Required Database Entities/Tables

1. **users** - Authentication and core user information
2. **profiles** - Extended profile information for all users
3. **teams** - Organizational groupings supporting manager-employee hierarchy
4. **team_memberships** - Many-to-many relationship between employees and teams with historical tracking
5. **work_sessions** - Track work shifts/periods for calculating work hours
6. **breaks** - Monitor break times and lunch periods
7. **tasks** - Task management system with support for manager assignment and tracking
8. **recommendations** - Store ML-generated personalized recommendations
9. **system_activity_logs** - Admin system monitoring and audit trail

## 6. Relationships Between Tables

1. **users 1:1 profiles** - Each user has exactly one profile
2. **users 1:many teams** - A user (as manager) can manage multiple teams
3. **teams 1:many team_memberships** - A team has many membership records over time
4. **profiles 1:many team_memberships** - A profile (employee) can belong to multiple teams over time
5. **profiles 1:many work_sessions** - A user has many work sessions over time
6. **work_sessions 1:many breaks** - A work session contains many break periods
7. **profiles 1:many tasks** - A user owns many tasks over time
8. **work_sessions 0:many tasks** - Tasks may be associated with a work session (nullable)
9. **profiles 1:many recommendations** - A user receives many ML-generated recommendations
10. **users 0:many system_activity_logs** - Users perform many system actions (logged)

## 7. Required Fields and Constraints

### Key Constraints:
- **UUID primary keys** using `gen_random_uuid()` for security
- **TIMESTAMPTZ** for all timestamp fields to handle time zones
- **Case-insensitive email uniqueness** via index on lower(email)
- **Role validation**: CHECK constraint for role IN ('ADMIN', 'MANAGER', 'EMPLOYEE')
- **Approval status validation**: CHECK constraint for approval_status IN ('PENDING', 'APPROVED', 'REJECTED')
- **Break type validation**: CHECK constraint for break_type IN ('SHORT_BREAK', 'LUNCH', 'OTHER')
- **Task status validation**: CHECK constraint for status IN ('TODO', 'IN_PROGRESS', 'DONE', 'BLOCKED')
- **Task priority validation**: CHECK constraint for priority IN ('LOW', 'MEDIUM', 'HIGH')
- **Temporal logic**: CHECK constraints ensuring end times > start times when not null
- **Active session prevention**: Partial unique index on work_sessions (profile_id) WHERE ended_at IS NULL
- **Active break prevention**: Partial unique index on breaks (work_session_id) WHERE ended_at IS NULL
- **Soft delete flags**: is_active BOOLEAN on users and teams tables
- **Historical tracking**: left_at TIMESTAMP NULLABLE in team_memberships with validation
- **Required foreign keys** with appropriate CASCADE/SET NULL behaviors

## 8. Data to be Calculated Dynamically

### Should NOT be stored (calculate on demand):
- **Focus Rate** - Calculate from raw timing data (work_sessions, breaks, tasks)
- **Productivity percentages** - Derive from completed tasks vs total tasks
- **Work-life balance scores** - Compute from work hours, break patterns, etc.
- **Streaks/consecutive days** - Calculate from work session dates
- **Average task completion time** - Derive from task timing data
- **Break frequency ratios** - Calculate from break counts vs work time

### Should be stored (raw data):
- Work session start/end timestamps
- Break start/end timestamps and types
- Task creation/update/completion timestamps
- Task estimated and actual time spent
- ML-generated recommendation content and metadata
- All user/profile/team/organizational data
- System activity logs

## 9. Missing from Previous Schema

Based on the documents, the previous schema was largely correct but needed these additions/clarifications:

### Missing Elements:
1. **Explicit lunch break type** - Already had 'LUNCH' in break_type CHECK constraint ✓
2. **Manager ability to set deadlines and priorities** - Already had priority and deadline fields in tasks table ✓
3. **Admin system monitoring capability** - Added system_activity_logs table
4. **Explicit inactive period monitoring** - Already captured via break monitoring and work session gaps
5. **Overworking/underworking pattern analysis** - Supported by work session duration tracking
6. **Work history viewing** - Supported by querying work_sessions, breaks, tasks per user
7. **Completed tasks tracking** - Already in tasks table with status='DONE'
8. **Manager view of employee productivity reports** - Supported by role-based queries
9. **Admin view of system reports/analytics** - Supported by system_activity_logs and aggregated queries

## 10. To Remove from Previous Schema

Based on document analysis, nothing needs removal - all previous schema elements are supported:

### All Previous Elements Are Valid:
- users table with role and approval_status ✓
- profiles table with professional information ✓
- teams and team_memberships for hierarchy ✓
- work_sessions for shift tracking ✓
- breaks with break_type including LUNCH ✓
- tasks with nullable work_session_id, priority, deadline ✓
- recommendations for ML output ✓
- system_activity_logs for admin monitoring ✓

## 11. Conflicts Between Documents

✅ **NO CONFLICTS IDENTIFIED**. The documents are fully complementary:
- Document 1: Provides vision, objectives, scope, theoretical framework, exclusions
- Document 2: Provides operational details, role capabilities, specific features, development tasks
- Both agree on target users (VAs/WFH), core purpose (ML-based recommendations), and system type (web-only)

## Approval Status

The requirements analysis shows that the previously proposed database schema is **consistent with both documents** and requires no fundamental changes. All schema elements can be justified by explicit requirements from the source documents or reasonable technical inferences necessary for implementation.

**Recommended Action**: Proceed to SQL/database implementation using the previously proposed schema, as it correctly implements all verified requirements from both project documents.