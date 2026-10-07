# AI Matcher (Part 2)
Gemini reranks the core's eligible shortlist (10-20 players) by role fit, style, distance,
availability and reliability, and returns player_id + score + one-line reason (Pydantic-validated).
Works both directions (captain -> players, player -> teams). Gemini never invents players;
reliability numbers come from the core formula. Stack: Gemini structured output, PostGIS shortlist, FastAPI.
