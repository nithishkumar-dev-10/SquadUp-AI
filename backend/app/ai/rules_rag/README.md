# Rules and Umpire Assistant (Part 2, RAG)
Answers rule questions from indexed rulebooks + tournament conditions (source shown), and in
Umpire mode turns a user's story into a structured scenario and returns decision, rule reference,
explanation, confidence and escalate flag. Tournament conditions override the general rulebook;
low confidence defers to the on-field umpire. Stack: Gemini + embeddings, pgvector, LangChain/LangGraph, PDF extraction.
