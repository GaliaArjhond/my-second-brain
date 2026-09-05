# KPI Definitions and Data Mapping

## AUTHORITATIVE ADVISER INFORMATION

### Customer Satisfaction / Satisfaction Score
**Formula (Confirmed by Adviser):**
Good / Total number of surveys × 100

### TPH
**Formula (Confirmed by Adviser):**
Total number of tickets processed / Total number of productive hours

### AHT
**Definition (Confirmed by Adviser):**
Average time spent on a ticket, excluding Pending or On-hold status.

### FTR — First Touch Resolution
**Formula (Confirmed by Adviser):**
Total number of one-touch tickets / Total number of tickets handled

### FRT — First Response Time
**Definition (Confirmed by Adviser):**
Average first response time from Ticket Creation to First Response.

## Adviser-Provided Data

The adviser supplied the following data files:
- **Copy of ICONIC Scorecard 2025.xlsx**
- **Copy of ICONIC Scorecard 2025 – Roster 2025.pdf**
- **Copy of ICONIC Scorecard 2025 – Surveys Raw.pdf**

### Roster Data Fields
- Name
- EID
- Email Address
- SSTART (Shift Start)
- SSEND (Shift End)
- Task
- TL (Team Lead)
- Team
- Status
- Default Schedule

### Survey Data Fields
- Assignee name
- Bad (count of bad surveys)
- Good (count of good surveys)
- Grand Total (total surveys)
- Total # Good
- Total # Bad
- Combined Surveys
- # Disputed
- Satisfaction adjustment
- Team
- Site
- Month

## KPI Data Mapping

| KPI | Required Data | Available | Missing | Status |
|-----|---------------|-----------|---------|--------|
| Customer Satisfaction | Good surveys, Total surveys | Good, Grand Total | None | **CAN BE CALCULATED**<br>Formula: Good / Grand Total × 100 |
| TPH | Tickets processed, Productive hours | Ticket solved‑week (tickets processed information) | Actual productive hours | **BLOCKED** |
| AHT | Ticket handling time, Ticket status/history | None | Ticket‑level handling time, Pending/on‑hold periods | **BLOCKED** |
| FTR | One‑touch ticket count, Total tickets handled | None | Number of interactions/touches per ticket, Total handled tickets | **BLOCKED** |
| FRT | Ticket creation timestamp, First‑response timestamp | None | Ticket creation time, First agent response time | **BLOCKED** |

## Data Gap / Open Questions

### Needed for TPH
- Actual productive hours per employee
- Preferably based on actual clock‑in/out and break deductions

### Needed for AHT
- Ticket ID
- Ticket creation/start time
- Ticket handling/resolution time
- Status history
- Pending/on‑hold timestamps

### Needed for FTR
- Ticket ID
- Number of agent touches/replies
- Resolution information
- Total handled tickets

### Needed for FRT
- Ticket ID
- Ticket creation timestamp
- First agent response timestamp

## Important Thesis Decision

### KPI Implementation Decision
The IntelliBreak system will use the adviser‑confirmed KPI definitions.

Customer Satisfaction can be implemented using the currently supplied survey data.

TPH, AHT, FTR, and FRT must remain pending until the required ticket‑level/productivity data is obtained.

The system must **NOT** generate synthetic or assumed KPI values.

Do not invent KPI weights or alternative formulas.

## Adviser Follow-up

**Status:** Pending additional data

We need to ask the adviser whether ticket‑level data is available for:
- TPH
- AHT
- FTR
- FRT

The adviser has already confirmed the KPI definitions/formulas.

## Related Notes
[[Project Hub]]
[[Current Thesis Requirements]]
[[Analytics]]
[[Team View]]
[[Requirements]]
[[Decisions]]
[[Advisor Feedback]]
[[UI UX/Dashboard]]

