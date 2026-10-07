# Vic - Conversational Orchestration Agent (Part 2)
Single chat front door (text primary, voice via mic button) built on LangGraph + Gemini function
calling. Routes to the 5 other GenAI features and the core as tools (parse_vacancy, search_players,
rank_candidates, draft_invite, translate_post, ask_rules, recommend_tournaments, send_invite...).
Approval rule: anything that sends a message or changes data needs a confirmation tap, and every
tool checks team permission. Vic can only use listed tools and never invents actions.
