# Reflection Questionnaire — PariShiksha Study Assistant
## Week 9 Mini-Project | PG Diploma AI-ML & Agentic AI Engineering

---

## Part A — Implementation Artifacts

### A1. Chunking Parameters

**Final parameters:**
- Chunk size: 300 tokens (approx 225 words)
- Overlap: 50 tokens
- Special handling: worked_examples kept whole, end_of_chapter split on question numbers, concept/activity use sliding window

**What pushed me to these values:**
I first tried pure fixed-size chunking at 500 tokens with no overlap. When testing retrieval for "What is Newton's second law?", the retrieved chunk contained both the second and third law on the same page, diluting the answer. Reducing to 300 tokens with 50 overlap kept individual concepts together while maintaining enough context. The most important decision was keeping worked_examples whole — Example 4.3 (bus acceleration) had its problem statement and solution on the same page. Splitting them would have caused the LLM to receive only the problem without the solution, producing a wrong answer.

---

### A2. A Retrieved Chunk That Was Wrong

**Query:** "What is displacement and how is it different from distance?"

**Wrong chunk retrieved (Rank 1):**
> "by drawing a velocity-time graph for its motion. 15. Two cars A and B start moving with a constant acceleration from rest in a straight line..."
> Section: 4.4 Motion in a Plane | Page 23

**Why the retriever returned it:**
The word "displacement" appears in end-of-chapter questions across multiple pages including page 23, which also contains distance-related numerical problems. BM25 matched on surface keyword frequency — "distance", "motion", "straight line" — without understanding that this chunk is an exercise page, not a definition page. The correct definition chunk (4.1.2) was ranked third because its text was shorter and had lower term frequency scores.

---

### A3. Grounding Prompt — v1 and v_final

**v1 (first attempt):**
You are a helpful science tutor. Answer the student's question using the provided context.
Only use information from the context below.
Context: {context}
Question: {question}

**v_final:**
You are a science study assistant for Class 9 students using NCERT textbooks.
You will be given CONTEXT extracted from NCERT Class 9 Science chapters.
Your job is to answer the student's question STRICTLY using only the provided context.
STRICT RULES:

If the answer is present in the context, answer clearly and simply for a Class 9 student.
If the answer is NOT in the context, respond with exactly: "I'm sorry, this topic is not covered..."
Do NOT use any outside knowledge, even if you know the answer.
Do NOT make up formulas, definitions, or examples not present in the context.
Keep answers concise — 3 to 5 sentences maximum.
If a formula is in the context, include it in your answer.

**What caused the revision:**
The v1 prompt used "only use information from the context" which is permissive — the model interpreted it as "prefer the context but supplement if needed." When testing Q18 (Newton's law of gravitation), v1 produced a full answer even though the gravitation chapter was not in our corpus. Changing to an explicit REFUSAL instruction with an exact response string fixed this — all 5 out-of-scope questions were correctly refused with v_final.

---

## Part B — Numbers from Evaluation

### B1. Evaluation Scores

- Total questions: 18
- Correct: 11/13 answered questions
- Partial: 2/13 (Q01 wrong retrieval, Q02 incomplete equations)
- Grounded: 13/13 — every answered question was supported by retrieved chunks
- Refusals appropriate: 5/5

**Number that bothered me most: 2 partial scores on direct questions.**
Q01 (displacement) and Q02 (kinematic equations) are the most basic questions in the corpus — they should have been easiest. Both failed due to retrieval, not generation. The LLM answered correctly from whatever context it received, but the retriever pulled the wrong chunks. This means a student asking the most fundamental question gets a worse answer than one asking a complex paraphrased question. That is backwards and indicates the retriever needs significant improvement.

### B2. Chunk Size Experiment
Not completed in this submission due to time constraints. Planned: compare 250 vs 500 token chunks on the same 18-question eval set. Expected outcome: 500 token chunks would hurt precision (multiple concepts per chunk) while 250 token chunks might lose context on worked examples.

### B3. Model Family Comparison
Not completed in this submission. Used Llama-3.1-8b-instant (decoder-only) via Groq as primary model. Planned comparison with flan-t5-small (encoder-decoder) noted for future work. Expected: flan-t5-small would struggle on multi-step reasoning questions due to its 77M parameter capacity limit.

---

## Part C — Debugging Moments

### C1. Most Frustrating Bug

**Bug:** Gemini API kept returning truncated answers mid-sentence for Newton's law questions. The answer would stop at "the object" with no punctuation.

**Time to fix:** ~45 minutes

**What I tried first:** Rewrote the grounding prompt thinking the instruction length was causing issues. Made no difference.

**Actual fix:** The `max_output_tokens` was set to 512 which was too low for detailed physics answers. Increasing to 1024 fixed it. The `finish_reason` was MAX_TOKENS not STOP — adding a warning check revealed this immediately.

**Fastest path for someone hitting the same bug:** Always check `response.candidates[0].finish_reason.name` after generation. If it says MAX_TOKENS instead of STOP, double your token limit before touching anything else.

### C2. What Still Bothers Me

The retriever consistently fails on basic definitional queries. "What is displacement?" retrieves page 23 (exercise page) instead of page 3 (definition page). The word "displacement" appears 47 times across the corpus so BM25 cannot discriminate. This bothers me because a Class 9 student's first question is almost always a definition question — exactly the query type our system handles worst. Fixing this requires dense semantic retrieval (sentence-transformers) which is planned for the Advanced stage.

---

## Part D — Architecture and Reasoning

### D1. Why Not Just ChatGPT?

When we tested Q18 — "Explain Newton's law of gravitation" — our system correctly refused because gravitation is not in our corpus. A raw ChatGPT call would have answered confidently from its training data. That sounds better, but it is not. PariShiksha's non-negotiable requirement is that answers stay grounded in NCERT content. A student who asks about gravitation should be told to refer to that chapter, not receive an answer that may use different notation or explanation level than their textbook. ChatGPT has no such constraint and would freely mix knowledge from multiple sources, making it impossible to verify whether the answer matches what the student's teacher will say.

### D2. The GANs Reflection

GANs are the wrong tool for this problem because they are designed for generation without grounding. A GAN generator learns to produce plausible-looking outputs that fool a discriminator — it has no mechanism to constrain generation to a specific source document. For PariShiksha, the core requirement is the opposite: we need a system that refuses to generate anything not supported by NCERT text. GANs optimize for plausibility, not faithfulness. The deeper principle is that generative architecture choice must match the fidelity requirement of the problem. When the problem demands "generate only from this specific source and refuse otherwise," you need retrieval-augmented generation with explicit grounding constraints — not adversarial generation.

### D3. Honest Pilot Readiness

**Honest answer: No, not Monday.**

Three things I would fix first:

1. **Retrieval quality** — 2 out of 13 answered questions failed due to retrieval pulling wrong chunks. A 15% retrieval failure rate means roughly 1 in 7 students gets a wrong or incomplete answer on basic questions. Dense retrieval using sentence-transformers must replace BM25 before any pilot.

2. **Evaluation coverage** — My eval set has 18 questions, all written by me. Real students ask questions with spelling errors, Hindi-English code-switching, and half-remembered terms. At least 50 real student questions from the contract teacher are needed before launching.

3. **No guardrails for adversarial input** — I tested clean out-of-scope questions but not prompt injection attempts or malformed inputs. A pilot with 100 students will surface these within the first day.

---

## Part E — Effort and Self-Assessment

### E1. Effort Rating

**8/10**

I am genuinely proud of the grounding prompt iteration — going from a permissive v1 that hallucinated on out-of-scope questions to a v_final that correctly refused all 5 out-of-scope queries. That required actually testing failure cases and tracing the root cause rather than assuming the prompt was fine.

### E2. Gap Between Me and a Stronger Student

A stronger student would have completed the dense retrieval comparison (Advanced stage). I planned to use sentence-transformers/all-MiniLM-L6-v2 alongside BM25 and compare scores on the same eval set. I did not complete it due to API rate limit issues consuming time that should have been spent on Advanced tasks. The rate limit problem was partly my fault — I did not read the free tier quotas before starting the evaluation loop.

### E3. What Two More Days Would Change

**First thing:** Replace BM25 with dense retrieval using sentence-transformers. This is the single highest-impact change — it would fix the retrieval failures on Q01 and Q02 and likely push correctness from 11/13 to 13/13.

**Last thing:** Add teacher mode (Optional) — citations with page and section references on every answer. This is only valuable after the retrieval foundation is solid. Polishing output format before fixing retrieval would be the wrong priority order.
