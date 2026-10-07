# Tournament Recommender (Part 2)
Core filters tournaments by sport/district/dates; Gemini compares each event with the team
profile (level, roster, free dates) and returns a fit reason + deadline flag. Missing-player
needs link back to the AI Matcher. Stack: Gemini structured output, PostgreSQL, FastAPI, Celery.
