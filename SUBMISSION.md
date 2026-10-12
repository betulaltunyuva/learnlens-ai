# LearnLens AI — Submission Summary

**Multimodal AI Hackathon 2026 · Track D: Personalized Tutoring & Adaptive Learning**

## Project name

LearnLens AI

## Links

- Demo video: https://youtu.be/5sruuk1WHsA
- Code: https://github.com/betulaltunyuva/learnlens-ai
- Devpost: https://devpost.com/software/learnlens-ai-aolqb2

## Elevator pitch

A multimodal tutor that learns from your own PDFs, slides and lecture videos,
answers only with cited evidence from them, adapts verified practice questions to
your level, and tracks what you have mastered.

## Inspiration / problem

Course material is spread across slides, PDF notes and recorded lectures. Generic
chatbots answer from the open web instead of the course, hallucinate, give no
sources, and do not know what an individual student already understands.
Practice questions are generic, repetitive and not adapted to the learner.

## What it does

- Builds one knowledge base from **PDFs (text + figures), PowerPoint (text,
  tables, images) and lecture videos (timestamped segments)**; also answers
  questions about single images.
- **Unified Tutor:** personalized answers grounded only in that material, with
  page / slide / timestamp citations, and an explicit refusal
  (*"Bu bilgi yüklenen ders materyallerinde bulunamadı."*) for off-material
  questions.
- **Adaptive Assessment:** MCQ, short-answer and numerical questions, each verified
  by a second model pass, de-duplicated, self-contained, and adapting
  Easy ↔ Medium ↔ Hard to performance.
- **Learner model:** per-topic EMA mastery updated after every quiz answer,
  diagnostic answer and tutoring conversation; topic accuracy; weak topics and
  likely misconceptions from the student's own wrong answers.
- **Cold start:** a 3-question diagnostic sets the starting level and difficulty.
- **Evaluation built in:** RAGAS (faithfulness, answer relevancy, context
  precision, context recall), a team-built benchmark with refusal accuracy and
  expected-source matching, question repetition rate, and a clearly labelled
  **synthetic** multi-session personalization simulation.

## How it works

1. Uploads are split into knowledge units, each with a fixed citation, and
   embedded with `gemini-embedding-2` (768-d).
2. Gemini tags units with topic, subtopic, key concepts and prerequisites in
   batches of 8, cached locally; only units actually in use are tagged
   automatically.
3. A question retrieves the top units by cosine similarity; Gemini
   (`gemini-3.5-flash-lite`) answers with only those units, their images and the
   learner's profile and weak topics, preserving citations.
4. Assessment questions cite one source unit, pass deterministic checks
   (citation, duplicates, no hidden-figure references) and a verifier pass, and
   take the knowledge tag of the cited unit as canonical topic.
5. Every answer updates the learner model:
   `mastery = old × (1 − α) + evidence × α` (quiz α = 0.30; tutoring is weak
   heuristic evidence, α = 0.05–0.08).
6. Learner state persists in a local JSON file; evaluation results export to JSON
   and CSV.

## Multimodal workflow

| Modality | How LearnLens uses it |
|---|---|
| PDF | Page text chunks + up to 2 embedded figures per page, sent with the page when it is retrieved and when questions are generated/verified |
| Slides | Text, tables and embedded images per slide; images go to the tutor and the question verifier |
| Video | Gemini Files API → timestamped topic segments, retrievable and citable like text |
| Image | Direct visual Q&A |

## Personalization approach

- Profile: level (Beginner / Intermediate / Advanced), learning goal, explanation
  style (Concise / Balanced / Detailed) — each mapped to explicit instructions.
- Weak topics (accuracy < 70 % or EMA mastery < 0.50) get extra explanatory
  support in tutor answers.
- Diagnostic sets the starting level and difficulty; assessment difficulty then
  adapts after every answer.
- Grounding always wins: personalization changes *how* content is explained, never
  *what* the material says.

## What makes it innovative

- A single retrieval space across PDF figures, slides and video timestamps, with
  modality-specific citations in one answer.
- Grounding treated as a measurable feature: refusal accuracy and expected-source
  matching in the benchmark, not just a prompt instruction.
- Two-stage question generation with deterministic and model-based checks, so
  practice questions are answerable, self-contained and not repeated.
- A transparent learner model that combines quiz, diagnostic and conversation
  evidence with explicit, documented weights.

## Challenges faced

- **RAGAS with Gemini:** embedding dimension mismatches broke RAGAS until we wrote
  a fixed 768-d Gemini embedding wrapper and used Gemini's OpenAI-compatible
  endpoint as judge.
- **Free-tier API quota:** RAGAS makes many calls per case and hit HTTP 429. We
  added a quota-safe Quick Benchmark, pauses between cases, clean stopping on
  quota errors, and never show scores from a run with no successful case.
- **Streamlit's rerun model:** state written during rendering can silently
  overwrite later changes (our diagnostic result was resetting adaptive
  difficulty), and failed uploads could re-call APIs on every rerun. We made these
  steps apply once and retry only on request.
- **Self-contained questions:** models like to write "according to the figure
  above". We combined prompt rules, a deterministic phrase filter and a verifier
  that sees the same images as the generator.
- **Consistent topics:** free-text quiz topics fragmented the learner model; topics
  now come from the cited unit's knowledge tag.

## Accomplishments

- End-to-end Track D loop working in one app: multimodal ingestion → grounded
  tutoring → verified adaptive assessment → learner model → personalization →
  evaluation.
- Citations for PDF pages, slides and video timestamps in a single knowledge base.
- Built-in, exportable evaluation instead of hand-picked examples. A complete
  Quick Benchmark run (3 cases, small sample) reached faithfulness 0.71, answer
  relevancy 0.94, context precision 0.75, context recall 1.00, refusal accuracy
  3/3 and expected-source match 2/2 (`results/quick_benchmark_2026-10-10.json`).
- Robust state handling: corrupt or partial learner files repair themselves, and
  failures in one component do not take down the rest of the app.

## What we learned

- Grounding needs enforcement *and* measurement; a refusal sentence is only useful
  if refusals are tested.
- Cheap deterministic checks (citations, duplicates, phrase filters) save many
  model calls and catch errors a model verifier can miss.
- Evaluation frameworks are sensitive to provider details (embedding dimensions,
  rate limits); quota-aware design matters as much as model quality.
- In Streamlit, side effects must be explicit and one-shot.

## Future work

- A pilot study with real students (the current multi-session personalization
  result is a **synthetic** simulation only).
- Show figures inside assessment questions so visual questions can be asked safely.
- Spaced-repetition scheduling from mastery estimates.
- A larger, multi-course benchmark and an independent judge model.
- Multi-user accounts and server-side storage.

## Built with

Python · Streamlit · Google Gemini API (`gemini-3.5-flash-lite`,
`gemini-embedding-2`, Files API, OpenAI-compatible endpoint) · google-genai ·
PyMuPDF · python-pptx · NumPy · RAGAS · openai (Python client for Gemini's
compatible endpoint) · python-dotenv

## AI development disclosure

AI coding assistants including ChatGPT and Claude Code were used during
development for code review, debugging and implementation assistance. All
generated code was reviewed, integrated and tested by the project author.
The application itself uses Google Gemini at runtime.
