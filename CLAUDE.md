# My Second Brain

This repository is my personal knowledge base.

## Purpose

Use this vault to store long-term knowledge, project context,
research, decisions, technical notes, and ideas.

## Folder Structure

- `00 Inbox` — temporary notes and unprocessed information
- `01 Projects` — active projects
- `02 Areas` — ongoing areas of responsibility
- `03 Research` — research, papers, and literature notes
- `04 Knowledge` — technical and general knowledge
- `05 Decisions` — important decisions and their reasoning
- `06 Resources` — useful references and resources
- `99 Archive` — inactive or obsolete information

## Rules

1. Treat existing notes as the source of truth.
2. Do not invent information.
3. Clearly distinguish facts, assumptions, and suggestions.
4. Do not delete notes unless explicitly instructed.
5. Do not overwrite important information without permission.
6. Prefer linking existing notes instead of duplicating information.
7. Keep notes focused on one main topic.
8. Preserve important project decisions.
9. Identify contradictions between notes.
10. Flag information that may be outdated.
11. Do not create unnecessary notes.
12. Before making large structural changes, explain what should change first.

## When Processing Notes

When asked to organize notes:

1. Read the relevant existing notes.
2. Identify duplicates.
3. Identify related notes.
4. Identify contradictions.
5. Suggest where information belongs.
6. Only modify files when explicitly requested.

## Note Types

### project

- Purpose: Track active projects with goals, tasks, and outcomes.
- Where it normally belongs: 01 Projects
- Required frontmatter: type, status, startDate
- When NOT to create: For ongoing responsibilities that aren't time-bound, or for reference material.

### knowledge

- Purpose: Capture stable, reusable information such as concepts, facts, or how-to guides.
- Where it normally belongs: 04 Knowledge
- Required frontmatter: type, tags
- When NOT to create: For time-specific notes, raw research, or personal notes that aren't generally applicable.

### research

- Purpose: Record literature reviews, experiment notes, or investigation findings.
- Where it normally belongs: 03 Research
- Required frontmatter: type, source, date
- When NOT to create: For finalized knowledge that belongs in Knowledge, or for project-specific tasks.

### decision

- Purpose: Document important choices, their rationale, and alternatives.
- Where it normally belongs: 05 Decisions
- Required frontmatter: type, date, outcome
- When NOT to create: For minor choices, temporary preferences, or notes that don't have lasting impact.

### Obsidian Documentation Rules

When making thesis-related changes:

1. Read relevant Obsidian notes before changing requirements or decisions.
2. Use existing notes whenever possible.
3. Add [[wikilinks]] only when they represent meaningful semantic relationships.
4. Do not add links merely to increase Graph View connectivity.
5. Prefer these relationships:
   - Research → Requirements
   - Adviser Feedback → Decisions
   - Requirements → Decisions
   - Decisions → Implementation
   - Learning → Implementation
   - Problems/Fixes → Technologies or features
6. Keep hub notes as navigation points rather than dumping all information into them.
7. Do not modify unrelated notes.
8. Do not invent facts, requirements, decisions, or relationships.
9. When a significant relationship is discovered, update the appropriate note with a [[wikilink]].
10. Before finishing a documentation task, check whether the new information should connect to an existing note.

## Important

The user remains the final decision maker.
Do not treat AI-generated information as automatically correct.
