# AGENTS.md - SquadUp Part 1 Build Rules (auto-loaded every session)

## 1. Project

SquadUp Part 1 is a backend-only marketplace app: a captain creates a team and posts
vacancies ("need 2 spikers Sunday morning at Peelamedu"), a player fills a profile
(role, area, availability) and marks "looking for a team". Each side filters and sees
the other, one side sends an invite/request, the other accepts, phone numbers are
revealed to both, and the vacancy count goes down. A dashboard shows what needs
attention. Part 2 (Tanglish parser, AI matcher, Vic) sits on top of this.

**Scope: volleyball only, Coimbatore only, English UI, backend only, no AI.**

## 2. NOT in Part 1 (never build these)

- Chat (phone reveal after accept; they call or WhatsApp)
- GPS check-in, ratings, reliability score, fraud rules, heat map
- Counter-offers on invites (accept, decline, cancel only)
- Tournament board, translation, Tamil UI
- Push notifications, SMS, WhatsApp API (in-app notification list only)
- Real SMS OTP (use a fake OTP printed in the console)
- Cricket, other districts, admin moderation panel

## 3. Tech stack

| Piece | Choice |
|---|---|
| Language / framework | Python + FastAPI |
| Database | PostgreSQL (Docker) |
| ORM + migrations | SQLAlchemy + Alembic |
| Validation | Pydantic |
| Auth | JWT access token |
| OTP | Fake OTP printed to console |
| Tests | pytest |
| Distance | Haversine on lat/lng (PostGIS later) |

## 4. Folder structure

```
SquadUp-AI/
  AGENTS.md            # this file (auto-loaded every session)
  README.md
  .gitignore
  docs/
    project-log/       # CHANGES.md, PROJECT_SUMMARY.md, DECISIONS.md (NEVER in root)
    reference/         # extracted project PDF text, API_CONTRACT.md
  backend/
    app/
      main.py          # creates the FastAPI app, includes routers
      core/
        config.py      # settings from .env
        db.py          # database session
        security.py    # JWT create/verify, OTP helpers
      models/          # SQLAlchemy tables (users, player_profiles, availability,
                       #   teams, team_members, grounds, vacancies, invites, notifications)
      schemas/         # Pydantic request/response models
      routers/         # thin endpoints (auth, profile, players, teams, grounds,
                       #   vacancies, invites, notifications, dashboard)
      services/        # real logic: permissions, matching, invites, notifications, dashboard
      ai/              # PART 2 placeholders: parser/, matcher/, drafting/, translation/,
                       #   rules_rag/, recommender/, vic/{tools,graph,voice}/
      trust/           # PART 2 placeholders: check-in, ratings, reliability, fraud, heat map
      extras/          # PART 2 placeholders: chat, tournaments, leaderboards, admin
    alembic/           # migration files (created properly in Step 1)
    tests/             # unit/, integration/, permissions/, privacy/
    scripts/           # seed data (Step 11)
    docker-compose.yml, Dockerfile, .env.example, requirements.txt
  ml/                  # PART 2 placeholders: data/{raw,synthetic,gold}, notebooks/,
                       #   training/, evaluation/
```

## 5. Architecture rules

- **Routers are thin.** Receive the request, call a function in `services/`, return
  the result. All business logic (permissions, matching, invites) lives in
  `services/` so Vic can call the same functions later.
- Use Pydantic schemas for all request/response shapes.
- **Never return phone numbers in public responses.** Separate public vs private
  response models; do not "remember to hide it".
- Permission checks go in the backend on every endpoint, never only in the frontend.
- Keep a `sport` column from day one instead of hard-coding "volleyball".
- One step at a time; never stack steps.

## 6. Data model (all tables)

### users
| Column | Notes |
|---|---|
| id | primary key |
| phone | unique. **Private.** Never returned in public responses |
| name | display name |
| created_at | timestamp |

### player_profiles (one per user)
| Column | Notes |
|---|---|
| user_id | unique, links to users |
| roles | list: spiker, setter, libero, blocker, all-rounder |
| level | beginner / intermediate / advanced |
| area | text, e.g. Peelamedu |
| lat, lng | for distance maths |
| travel_radius_km | how far this player will travel |
| looking_for_team | true/false. If true, appears in Players Available directory |
| looking_type | permanent / guest / tournament_only |
| bio | optional |

### availability (many rows per user)
| Column | Notes |
|---|---|
| user_id | links to users |
| day_of_week | 0-6 |
| slot | morning / afternoon / evening |

"Free Sunday morning" = one row (6, morning).

### teams
| Column | Notes |
|---|---|
| id, name | |
| sport | "volleyball" now, but keep the column (cricket later, no schema change) |
| area, lat, lng | home location |
| level | village / town / district |
| created_by | the owner |

### team_members
| Column | Notes |
|---|---|
| team_id, user_id | together unique, nobody joins twice |
| member_role | owner / manager / player |
| joined_at | |

Permissions are built on this table: "can this user edit this team?" = "row here
with role owner or manager?".

### grounds
| Column | Notes |
|---|---|
| id, name, area, lat, lng | seeded manually for Coimbatore; later GPS check-in |

### vacancies
| Column | Notes |
|---|---|
| id | |
| team_id, created_by | |
| role_needed | spiker, setter... |
| count_needed | how many still needed. Goes down as people accept |
| match_date | |
| slot | morning / afternoon / evening |
| ground_id | |
| vacancy_type | permanent / guest / tournament |
| match_type | friendly / tournament |
| status | open / filled / closed |
| notes | optional |

### invites (one table for both directions)
| Column | Notes |
|---|---|
| id | |
| vacancy_id | which vacancy this is about |
| from_user_id, to_user_id | |
| direction | `invite` (team -> player) or `request` (player -> team) |
| status | pending / accepted / declined / cancelled |
| message | optional |
| created_at, responded_at | |

One table for both directions is deliberate: identical flow, only who may accept
changes.

### notifications
| Column | Notes |
|---|---|
| id, user_id | who gets it |
| type | invite_received, request_received, invite_accepted... |
| ref_id | the invite or vacancy it points to |
| is_read | |
| created_at | |

## 7. Build order (one step at a time)

### Step 0 - Setup
**Build:** repo, virtual env, FastAPI "hello" endpoint, Postgres in Docker, `.env`,
first commit, folder structure from section 4.
**Done when:** `GET /health` returns OK and the app connects to Postgres.

### Step 1 - Data model and API contract
**Build:** all tables from section 6 as SQLAlchemy models, first Alembic migration,
full endpoint list (section 8).
**Rules:** user writes table design / ER diagram by hand first; AI turns the design
into models, generates the migration, seeds fake data, generates the OpenAPI spec.
**Hand-off:** give the endpoint list + OpenAPI spec to the frontend team now.
**Done when:** `alembic upgrade head` creates all tables and they are visible in a
DB viewer.

### Step 2 - Auth (OTP + JWT)
**Build:** `POST /auth/send-otp`, `POST /auth/verify-otp`, `get_current_user`
dependency.
**How:** server makes a 6-digit code with an expiry (printed to console for v1);
on correct code, create user if new and return a JWT; every later request carries
the JWT which `get_current_user` decodes.
**Done when:** login works with a fake OTP and a protected endpoint returns 401
without a token.

### Step 3 - Player profile and availability
**Build:** `GET/PUT /me/profile`, `PUT /me/availability`,
`GET /players/available` (no filters yet).
**Rules:** privacy rule — directory response schema must NOT contain phone.
Separate Pydantic models: `PlayerPublic` (no phone) vs `PlayerPrivate` (own
profile).
**Done when:** the directory lists players and the phone never appears in any
response.

### Step 4 - Teams, roster and permissions
**Build:** create team, view team, edit team, list members, add/remove a member,
promote a member to manager.
**Rules:** user writes `require_team_role(user, team, allowed_roles)` by hand
first; every team endpoint calls it.

| Action | Owner | Manager | Player | Outsider |
|---|---|---|---|---|
| View team | yes | yes | yes | yes |
| Edit team details | yes | yes | no | no |
| Add / remove roster members | yes | yes | no | no |
| Promote to manager | yes | no | no | no |
| Delete team | yes | no | no | no |
| Post vacancy for team | yes | yes | no | no |

**Done when:** pytest tests prove each "no" in the table returns 403.

### Step 5 - Grounds
**Build:** `GET /grounds`. Seed 10-20 real Coimbatore grounds by hand (name, area,
lat, lng).
**Done when:** ground list returns real places.

### Step 6 - Vacancy board
**Build:** create vacancy, view, list (open only by default), close, edit.
**Rules:** only owner or manager can create/edit/close; `count_needed` >= 1;
`match_date` not in the past; only `open` vacancies show on the public board.
**Done when:** a captain can post, and a stranger can see but not edit.

### Step 7 - Filters and matching (heart of the app)
**Build two searches:**
1. Player searching vacancies: `GET /vacancies?role=&area=&date=&slot=&max_distance_km=`
2. Captain searching players for a vacancy: `GET /vacancies/{id}/candidates`

**Candidate filter (user writes by hand).** A player is a candidate if ALL true:
1. `looking_for_team` is true
2. their roles include the vacancy's `role_needed`
3. they have an availability row for the weekday of `match_date` and the vacancy `slot`
4. distance from player to ground is within the player's `travel_radius_km`
5. they are not already on this team
6. they have no pending invite for this vacancy

Then sort by distance (closest first). Distance = Haversine formula between two
lat/lng points.
**Done when:** seeded data shows the expected players and hides the wrong ones.

### Step 8 - Invites, requests and the accept flow
**Build:**
- `POST /vacancies/{id}/invite` (captain -> player)
- `POST /vacancies/{id}/request` (player -> team)
- `POST /invites/{id}/accept`
- `POST /invites/{id}/decline`
- `POST /invites/{id}/cancel` (the sender withdraws it)
- `GET /me/invites` (received and sent)

**Status machine (user writes by hand):**
```
pending --accept--> accepted
pending --decline-> declined
pending --cancel--> cancelled
(accepted, declined, cancelled are final. Nothing moves out of them.)
```

**Who may do what:**

| Action | Allowed user |
|---|---|
| Accept / decline an `invite` | the invited player (to_user) |
| Accept / decline a `request` | owner or manager of the team |
| Cancel | the user who sent it |

**What happens on accept (all in one database transaction):**
1. Check the invite is still `pending`.
2. Check the vacancy is still `open` and has `count_needed` above 0.
3. Set invite to `accepted`.
4. Reduce `count_needed` by 1. If it reaches 0, set vacancy to `filled`.
5. If `vacancy_type` is `permanent`, add the player to `team_members` as `player`.
6. Create notifications for the other side.

**Two bugs to design against:**
- **Race condition:** two players accept the last slot at the same moment. Lock the
  vacancy row (`SELECT ... FOR UPDATE`) inside the transaction so only one wins.
- **Double accept:** accepting an already accepted invite must fail cleanly, not
  reduce the count twice.

**Phone reveal rule:** `GET /users/{id}/contact` returns the phone only if an
`accepted` invite exists between the two users. Otherwise return 403.
**Done when:** tests cover every status move, both wrong-user cases, a full vacancy,
and the phone reveal.

### Step 9 - Notifications (in-app only)
**Build:** create a notification inside the same functions that create or change
invites. `GET /me/notifications`, `POST /notifications/{id}/read`.
**Rules:** user decides which event notifies whom and writes it as a table in
`docs/project-log/DECISIONS.md`.
**Done when:** every invite event produces a notification for the right person.

### Step 10 - Dashboard
**Build:** one endpoint `GET /me/dashboard` returning everything the home screen
needs. Suggested content (user decides):

| Section | What it shows |
|---|---|
| My teams | teams I belong to and my role in each |
| Needs my action | pending invites sent to me, pending requests for teams I manage |
| My open vacancies | vacancies from my teams with count still needed and pending requests |
| Upcoming matches | accepted invites with a future date |
| Suggested for me | open vacancies matching my profile (reuse Step 7 filter in reverse) |
| Unread notifications | count only |

**Done when:** a fresh user and a busy captain both get a sensible dashboard.

### Step 11 - Tests, seed data, deploy
**Build:** seed script (20 players, 5 teams, 15 vacancies), full test run,
Dockerfile, one cheap deployment (Render or Railway).
**Done when:** a teammate can open the live API docs and complete the flow from
login to accepted invite.

## 8. Endpoint list (share with frontend team)

| Area | Endpoint |
|---|---|
| Auth | POST /auth/send-otp, POST /auth/verify-otp |
| Profile | GET /me, GET/PUT /me/profile, PUT /me/availability |
| Directory | GET /players/available, GET /players/{id} |
| Teams | POST /teams, GET /teams/{id}, PUT /teams/{id}, DELETE /teams/{id} |
| Roster | GET /teams/{id}/members, POST /teams/{id}/members, DELETE /teams/{id}/members/{user_id}, PUT /teams/{id}/members/{user_id}/role |
| Grounds | GET /grounds |
| Vacancies | POST /teams/{id}/vacancies, GET /vacancies, GET /vacancies/{id}, PUT /vacancies/{id}, POST /vacancies/{id}/close, GET /vacancies/{id}/candidates |
| Invites | POST /vacancies/{id}/invite, POST /vacancies/{id}/request, POST /invites/{id}/accept, /decline, /cancel, GET /me/invites |
| Contact | GET /users/{id}/contact |
| Notifications | GET /me/notifications, POST /notifications/{id}/read |
| Dashboard | GET /me/dashboard |

## 9. Hand-written by user vs AI-built

**User writes the first version (AI does NOT write unless explicitly asked):**
- Table design / ER diagram (Step 1)
- `require_team_role` permission function (Step 4)
- Candidate-matching filter (Step 7)
- Invite status machine and accept transaction (Step 8)
- Also hand-written by user: privacy rule (Step 3), dashboard decisions (Step 10),
  the pytest tests for permissions and privacy

**AI builds, user reviews:**
- Project setup, Docker, config
- OTP and JWT boilerplate
- CRUD endpoints for profile, team, grounds
- Pydantic schemas
- Migrations and seed data
- Notification endpoints
- Dashboard SQL queries, Dockerfile, deploy config

## 10. Workflow rules (strict)

1. Do NOT build anything until the user says **"implement Step N"**.
2. Work on ONE step at a time.
3. Before coding any step, give a plan ONLY (files to create/change, approach in
   plain language, edge cases) and wait for **"plan approved"**.
4. Keep routers thin. Put all logic in `services/`. Use Pydantic schemas. Never
   return phone numbers in public responses.
5. Do not touch files outside the current step.
6. Do not write the Step 1, 4, 7, 8 core logic unless the user explicitly asks.
7. Read every diff; explain the request flow; run tests before finishing a step.
8. Do not add anything from the "NOT in Part 1" list.

## 11. Documentation rule (mandatory)

Maintain in **`docs/project-log/`**, updated at the END of every step:

- **`docs/project-log/CHANGES.md`** — step-by-step log, newest at the bottom:
  `## Step N - <name>  (<date>)` with sections: What was built, Files created,
  Files changed, Tables / endpoints added, How to test it, Tests added,
  Known gaps / TODO.
- **`docs/project-log/PROJECT_SUMMARY.md`** — rewritten after every step so it always
  reflects the CURRENT state: what SquadUp is, tech stack, folder structure (one line
  per file), data model, endpoints built so far (grouped), a request flow example
  (router -> service -> database -> response), current progress (done/next), how to
  run locally.
- **`docs/project-log/DECISIONS.md`** — one line per design decision and why.

**Never create these three files in the root. They live in docs/project-log/.**

**A step is not "done" until CHANGES.md and PROJECT_SUMMARY.md are updated.**

## 12. Part 2 reference (from project PDF)

Compact extract of SquadUp-AI-Scoped.pdf (PDF deleted; full text at
`docs/reference/SquadUp_AI_Scoped_extracted.txt`). **Part 2 starts only after the
Part 1 checklist is complete.**

**Core design rule:** the core decides what is true and what is allowed; AI only
understands, ranks, explains and drafts; the app works if AI is down. Gemini never
invents data — it only works on records returned by the core.

**AI features (1 transformer + 6 GenAI, 0 classic ML):**

| Feature | What it does | Stack | Inputs / outputs / key rules |
|---|---|---|---|
| Tanglish Vacancy Parser (DL, flagship) | Turns one messy Tanglish line into a filled vacancy form | PyTorch, HF Transformers, MuRIL/IndicBERT, FastAPI, Pydantic, Gemini (data only) | In: raw post. Out: intent (post_vacancy / looking_for_team / other) + slots (role, count, date, time, place, urgency) + confidence; low confidence pre-fills form for manual fix. Served as POST /parse/post |
| Vic (orchestration agent, text + voice) | One chat front door reaching every feature as tools | LangGraph, Gemini function calling + audio STT/TTS, FastAPI, Pydantic, Redis (state) | Tools: parse_vacancy, search_players/teams, rank_candidates, draft_invite, translate_post, ask_rules/give_ruling, recommend_tournaments, send_invite, get_profile, update_profile. **Approval rule:** anything that sends or changes data needs a confirmation tap; every tool checks team permission; Vic cannot invent actions |
| AI Matcher | Reranks the core's shortlist and explains each rank in one line | Gemini structured output, PostgreSQL + PostGIS, FastAPI, Pydantic | Core filters 10-20 eligible (Gemini never invents players) -> Gemini returns fixed JSON {player_id, score, reason} validated by Pydantic -> captain approves -> core notifies. Works both directions; reliability figures come from the core formula |
| Auto-drafted Invites | Writes invite/request messages and follow-ups in Tamil/English/Tanglish | Gemini, FastAPI, Celery (follow-ups), push/SMS/WhatsApp | Core passes vacancy details, player name, language, tone; captain edits/approves before core sends |
| Translation | Makes every post readable in any of the 3 languages | Gemini, FastAPI, Redis (cache), i18n | Post stored once in original language; translated once per language and cached; sports terms and place names kept as-is |
| Rules and Umpire RAG | Answers rule questions from real docs; umpire mode gives decisions | Gemini + embeddings, pgvector, LangChain/LangGraph, PDF extraction, Pydantic | Scenario structuring -> ask one question if a fact is missing -> retrieve chunks (tournament conditions override rulebook) -> fixed JSON: decision, rule reference, explanation, confidence, escalate flag. Low confidence / no rule = refer to umpire, never guess |
| Tournament Recommender | Suggests tournaments that fit a team's level, location, calendar | Gemini structured output, PostgreSQL, FastAPI, Celery | Core filters by sport/district/dates -> Gemini returns fit reason + deadline flag; short teams link to AI Matcher |

**Pipeline:** text post -> parser -> structured vacancy -> core filters shortlist ->
Gemini ranks/explains -> captain approves -> invites sent -> match played -> captain
confirms attendance -> heat map and reliability updated.

**Trust layer (Part 2):** GPS check-in near a known ground, captain-confirmed
attendance, two-way ratings, formula-based reliability score, rule-based fraud checks,
calendar heat map.

**Scope note (deferred, do NOT build):** photo-based check-in verification (Vision
Transformer), Team Composition Assistant, Backup Player Suggester.

## 13. Current progress

**Project structure scaffold done. Steps done: none. Next: Step 0.**
