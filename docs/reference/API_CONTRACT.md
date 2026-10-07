# Part 1 API contract (endpoint list)

Source of truth for the frontend team. Final OpenAPI spec is generated after Step 1;
this list is fixed by the build guide.

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

Rules that shape every response: phone numbers never appear in public responses
(separate public/private Pydantic models); GET /users/{id}/contact returns a phone
only when an accepted invite exists between the two users.
