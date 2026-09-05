---
type: project
project: IntelliBreak
status: active
tags:
  - intellibreak
  - thesis
  - flutter
---

# IntelliBreak

## Overview
IntelliBreak is a software project/thesis system focused on work-break management for people who work remotely, particularly Virtual Assistants (VAs) and Work-from-Home (WFH) workers. The project aims to help users manage work sessions and breaks while providing information that can support healthier and more productive work-break behavior.

## Current Status
Active development

## Goals
- Help users manage work sessions and breaks
- Provide information to support healthier and more productive work-break behavior
- Implement work/shift and break tracking functionality
- Implement user and administrative/manager functionality
- Conduct software evaluation with qualified target users
- Implement Machine Learning/Random Forest component when sufficient data and core functionality are established

## Target Users
- Virtual Assistants
- Work-from-Home workers

## Tech Stack
- Frontend/Application: Flutter (current technology being used)
- Backend: Firebase (being considered/used for backend services, including Firebase Authentication)
- Machine Learning: Random Forest component has been considered (exact implementation, features, training process, and integration timing to be determined)
- Note: The exact final backend architecture is still to be determined

## Core Features
- User authentication/login
- Session/shift management
- Work-session tracking
- Structured break logging
- Break records such as Break 1, Break 2, and Lunch
- Focus Rate
- Break-related recommendations or management
- User productivity/work information
- Manager view
- Team view
- Administrative functionality
- Software evaluation by qualified target users

## Architecture
Note: The project uses "shift" rather than "session" when referring to the user's work period where appropriate. "End shift" is preferred over "end session."
The system should support collecting relevant work/break data that can be used for evaluation and potentially for intelligent recommendations.
Exact database schema and backend architecture are to be determined.

## UI/UX
- IntelliBreak should feel like a calm, professional productivity application designed for long periods of use
- Priorities include visual comfort, accessibility, focus, low distraction, clear information hierarchy, consistency, and performance
- Dark mode should use a dark charcoal-based interface with controlled contrast rather than simply using pure black
- The UI should include appropriate dashboard, user, manager, team, and administrative views
- The design should be accessible and usable for extended work periods

## Research
- The project includes research into workplace productivity and work-break management
- Research is being conducted to support the design and evaluation of IntelliBreak
- A software evaluation questionnaire is being prepared for qualified Virtual Assistants / Work-from-Home workers
- The exact required number of respondents is still to be determined based on adviser/research requirements

## Decisions
- Terminology: Using "shift" rather than "session" for work periods; "End shift" preferred over "end session"
- Frontend technology: Flutter selected for application development
- Backend consideration: Firebase being considered/used for backend services
- Machine Learning: Random Forest component considered for future implementation
- Evaluation approach: Will be evaluated with qualified target users rather than requiring a specific company

## Known Problems
To be determined

## Related Technology Areas
- [[Flutter Folder Structure]]
- [[Authentication]]
- [[Break System]]
- [[Tech Stack]]
- [[UI Design]]

## Current Focus
- Build the core IntelliBreak application and UI first
- Establish the required work/shift and break tracking functionality
- Implement the required user and administrative/manager functionality
- Conduct software evaluation with qualified target users
- Implement the Random Forest/ML component when the required data, features, and core application functionality are sufficiently established

## Next Steps
To be determined

## Personal Tracking
- [[07 Personal/Finance/00 Dashboard.md]] - Thesis Fund tracking

## Related Notes
- [[Flutter Folder Structure]]
- [[Authentication]]
- [[Break System]]
- [[Tech Stack]]
- [[UI Design]]
- [[Current Thesis Requirements]]
- [[Focus Rate - Research Notes]]