# CHANGES.md - step-by-step log (newest at the bottom)

## Scaffold - project structure  (2026-10-07)
- What was built: full folder tree for Part 1 core + Part 2 placeholders; no logic,
  no installs, no docker, no Step 0 work.
- Files created: AGENTS.md updates (paths + Part 2 reference), README.md, .gitignore,
  docs/reference/API_CONTRACT.md, docs/reference/SquadUp_AI_Scoped_extracted.txt,
  docs/project-log/{CHANGES,PROJECT_SUMMARY,DECISIONS}.md,
  backend/app/ (main.py, core/, models/, schemas/, routers/, services/ — one-line
  docstrings only), Part 2 placeholders (ai/*, trust/, extras/ with README + __init__),
  backend/{alembic,tests,scripts}, placeholder infra files
  (docker-compose.yml, Dockerfile, .env.example, requirements.txt), ml/* placeholder tree.
- Files changed: none outside the scaffold (SquadUp_Part1_Build_Guide.md was already
  deleted from root by the user before this task).
- Tables / endpoints added: none.
- How to test it: `ls -R` matches the tree in PROJECT_SUMMARY.md; nothing runs yet.
- Tests added: none (tests/ folders are empty placeholders).
- Known gaps / TODO: Step 0 not started; alembic/, tests/, scripts/ are empty;
  Part 2 folders are placeholders by design.
