# PROJECT_SUMMARY.md - current state of the project (rewritten every step)

## What SquadUp is
SquadUp is a two-way marketplace for grassroots sports: captains post vacancies, players
mark themselves available, both sides find each other, an invite is accepted, phone numbers
are revealed and the vacancy count goes down. Part 1 builds the plain backend core
(volleyball, Coimbatore, English, no AI). Part 2 adds the AI layer (Tanglish parser, Vic,
matcher, drafting, translation, rules RAG, recommender) and the trust layer on top.

## Tech stack
Python + FastAPI, PostgreSQL (Docker), SQLAlchemy + Alembic, Pydantic, JWT auth,
fake OTP, pytest, Haversine distance. (Part 2 adds: Gemini, PyTorch/HF transformers,
LangGraph, pgvector, Redis, PostGIS.)

## Folder structure (one line per file/folder)
- AGENTS.md - auto-loaded session rules + the full Part 1 plan + compact Part 2 reference
- README.md - project overview
- .gitignore - hides .env, venv, __pycache__, ml/data, model weights
- docs/project-log/ - CHANGES.md, PROJECT_SUMMARY.md, DECISIONS.md (never in root)
- docs/reference/ - extracted project PDF text, API contract notes
- backend/app/main.py - FastAPI app factory (placeholder)
- backend/app/core/ - config.py (.env settings), db.py (session), security.py (JWT/OTP)
- backend/app/models/ - 9 SQLAlchemy tables (Step 1)
- backend/app/schemas/ - Pydantic request/response models per area (phone only in private)
- backend/app/routers/ - thin HTTP endpoints per area (Steps 2-10)
- backend/app/services/ - all business logic: permissions, matching, invites, notifications, dashboard
- backend/app/ai/ - PART 2: parser, matcher, drafting, translation, rules_rag, recommender, vic/{tools,graph,voice}
- backend/app/trust/ - PART 2: check-in, attendance, ratings, reliability, fraud, heat map
- backend/app/extras/ - PART 2: chat, tournaments, leaderboards, admin
- backend/alembic/ - migrations (Step 1)
- backend/tests/ - unit/, integration/, permissions/, privacy/
- backend/scripts/ - seed data (Step 11)
- backend/{docker-compose.yml, Dockerfile, .env.example, requirements.txt} - Step 0/11 placeholders
- ml/ - PART 2: data/{raw,synthetic,gold}, notebooks/, training/, evaluation/

## Data model
users (id, phone-PRIVATE, name, created_at); player_profiles (user_id, roles, level,
area, lat/lng, travel_radius_km, looking_for_team, looking_type, bio); availability
(user_id, day_of_week, slot); teams (id, name, sport, area, lat/lng, level, created_by);
team_members (team_id+user_id unique, member_role, joined_at); grounds (id, name, area,
lat/lng); vacancies (id, team_id, created_by, role_needed, count_needed, match_date, slot,
ground_id, vacancy_type, match_type, status, notes); invites (id, vacancy_id, from/to_user,
direction invite|request, status pending|accepted|declined|cancelled, message, timestamps);
notifications (id, user_id, type, ref_id, is_read, created_at).

## Endpoints built so far
None. The contract list is in docs/reference/API_CONTRACT.md.

## Request flow example
Not applicable yet; target pattern is router -> services/ function -> database -> Pydantic
response schema (routers stay thin, phone never in public responses).

## Current progress
Scaffold done. Steps done: none. Next: Step 0 (setup).

## How to run the project locally
Not runnable yet (Step 0 not started). Planned: docker compose up for Postgres, then
uvicorn app.main:app, GET /health.
