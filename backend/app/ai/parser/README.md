# Tanglish Vacancy Parser (Part 2)
Fine-tuned transformer (MuRIL/IndicBERT) with intent head + slot tagger that turns a messy
Tanglish line into a structured vacancy form: "sunday morning peelamedu 2 spikers 1 setter venum"
-> intent=post_vacancy, roles+counts, date, time, place, urgency.
Served as POST /parse/post with a confidence score; low confidence pre-fills the form for manual fixing.
