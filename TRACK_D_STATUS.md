# LearnLens AI — Track D Status

**Multimodal AI Hackathon 2026 · Track D: Personalized Tutoring & Adaptive Learning**

## Project overview

LearnLens AI is a Streamlit tutor that works from a student's own course material
(PDFs, PowerPoint slides, lecture videos, images). It answers questions only from
that material with citations, generates verified adaptive assessments, maintains a
per-topic learner model, and personalizes explanations to the learner's level,
goal, preferred style and weak topics.

## Architecture

```
Uploads (PDF / PPTX / Video / Image)
   │
   ├─ PDF  → PyMuPDF text (page chunks, 1200/200) + embedded figures (≤2/page)
   ├─ PPTX → python-pptx text, tables, embedded pictures (per slide)
   └─ Video→ Gemini Files API → timestamped segments (topic + text)
   │
   ▼
Gemini embeddings (gemini-embedding-2, 768-d)
   ▼
Unified knowledge base (items + vectors, citation per item,
  topic / subtopic / key_concepts / prerequisites tags)
   ▼
Cosine retrieval (top-k, source scope filter)
   ▼
Gemini generation (gemini-3.5-flash-lite) with learner profile + mastery
   ▼
Learner model (topic_performance, learner_mastery EMA, conversation_counts)
   ▼
learner_state.json  ·  evaluation_results.json/.csv  ·  knowledge_tags.json
```

Single-file app: `app.py`. Tabs (demo order): Learner Profile, PDF RAG,
Slide Understanding, Video Understanding, Image Understanding, Unified Tutor,
Adaptive Assessment, Mastery Dashboard, Evaluation. A sidebar shows session
status (knowledge units, tags, learner level, next difficulty).

## Multimodal ingestion

| Modality | Extraction | Unit / citation |
|---|---|---|
| PDF | Page text split into chunks; up to 2 embedded figures per page (PyMuPDF `get_images` / `extract_image`, tiny images skipped, unsupported formats converted to PNG). Figure-only pages are kept as retrievable units. | `[file.pdf, Page N]` |
| Slides | Text frames, tables, embedded pictures per slide | `[file.pptx, Slide N]` |
| Video | Gemini segments the video into timestamped topic segments (JSON mode; empty segments dropped; processing wait capped at 10 min) | `[video.mp4, mm:ss - mm:ss]` |
| Image | Direct Gemini visual Q&A (Image tab) | — |

When a retrieved PDF page or slide has images, up to 2 images per page/slide are
attached to the Gemini request (each page sent once, labelled with its citation).

A failed upload (embedding error, quota, unreadable file) shows an error with a
**Retry** button in that tab only; it is not retried automatically on every
rerun, and the other tabs keep working. Removing a file from an uploader removes
its units from the knowledge base.

## Source grounding

- Answers use only retrieved material; every important claim carries the exact
  citation supplied in context (page / slide / timestamp).
- Unsupported questions are refused with:
  **"Bu bilgi yüklenen ders materyallerinde bulunamadı."**
- Refusal detection (tutor UI, learner model, benchmark) is deterministic and
  tolerant to formatting: Turkish ı/İ and diacritics, case, Markdown,
  punctuation and line breaks are normalized before matching a small set of
  Turkish/English refusal phrases. No extra LLM call.
- Personalization instructions explicitly may not add unsupported facts.

## Unified knowledge base and knowledge tags

PDF chunks, slides and video segments are merged into one vector index
(`build_unified_knowledge_base`) with a source-scope filter (All / PDF / Slides / Video).
Each unit carries `topic`, `subtopic`, `key_concepts`, `prerequisites`.

- Tags come from Gemini in batches of 8 units per call (JSON mode), cached in
  `knowledge_tags.json` by a hash of citation + text. Cached units are never
  re-tagged, also across restarts.
- **Automatic, need-based tagging:** when the Unified Tutor answers (after the
  answer is shown) or an assessment/diagnostic question is accepted, only the few
  untagged units actually used are tagged — at most one batched call. Units that
  failed once are not retried automatically in that session. A toggle in
  *🧩 Knowledge Structure* turns this off to save quota.
- **Manual:** *Generate tags for all N untagged unit(s)* tags the whole material.
- Batches are told which topic names already exist and reuse that spelling.
- Existing video segment topics are preserved. If tagging fails, RAG works unchanged.

## Canonical topics

Quiz results, diagnostic answers, EMA mastery, conversation counts and the
dashboard share one topic name per source concept:

1. A question's topic = topic tag of the knowledge unit it cites
   (the question's own free-text topic is kept as `generated_topic` and used
   only when no tag exists).
2. Tutor conversations use the topic tag of the top retrieved unit.
3. `canonical_topic_name` maps labels that differ only in whitespace/case to the
   spelling already in use. It never merges different topics (no fuzzy matching).

## Adaptive assessment

- Question types: MCQ, Short Answer, Numerical.
- Difficulty Easy → Medium → Hard: two consecutive correct answers step up,
  one wrong answer steps down.
- Optional topic scope uses semantic retrieval to pick source units.
- Every question carries one of the exact citations of its source units.
- Open answers are graded semantically by Gemini (numerical equivalence accepted).
- Generation, verification and grading use Gemini JSON mode; a malformed
  response only skips that attempt. API/quota errors stop immediately with a
  short message instead of retrying.

## Question verification and visual content

Each generated question goes through a second Gemini pass that checks it is
answerable from the material only, the key is correct, exactly one MCQ option is
correct, numerical answers are derivable, the citation is supported, and the
question is fully self-contained.

PDF figures of the selected pages (max 2 per page) are attached to **both** the
generation and the verification request; slide images are attached to
verification. The assessment UI shows **text only**, so:

- the generator is told the student will not see any image,
- the verifier rejects any question that needs a figure, chart, equation image
  or table not fully described in the text,
- a deterministic filter rejects questions that point at hidden visuals
  ("şekildeki", "grafiğe göre", "slaytta", "the figure", "shown above" …) before
  the verifier call.

Rejected questions are regenerated (up to 8 attempts).

## Duplicate prevention

New questions are compared against the last 15 accepted questions plus the current
one using normalized text + `SequenceMatcher` ratio ≥ 0.82 (`is_similar_question`);
similar questions are discarded. Each attempt also uses a different "question focus".

**Question Repetition Rate** (Evaluation tab) re-checks every accepted question in
`quiz_history` against all earlier ones with the same rule:
`repeated / total — rate%`.

## Learner model

| Signal | Store | Rule |
|---|---|---|
| Quiz answer | `topic_performance` | correct / total per topic |
| Quiz answer | `learner_mastery` | EMA, evidence 1.0 / 0.0, α = 0.30 |
| Diagnostic answer | `learner_mastery` | EMA, evidence 1.0 / 0.0, α = 0.30 (not added to `topic_performance`) |
| Tutor conversation | `learner_mastery` | EMA, evidence 0.55, α = 0.05 (conversation-based heuristic) |
| Tutor conversation with difficulty signal ("anlamadım", "zorlanıyorum", "karıştırıyorum", "bilmiyorum", "I'm confused" …) | `learner_mastery` | EMA, evidence 0.35, α = 0.08 |
| Tutor conversation | `conversation_counts` | +1 per topic |

`new = old × (1 − α) + evidence × α`, new topics start at 0.50, one shared
function (`ema_update`). Refused (off-material) questions do not update the model.

Mastery Dashboard shows: overall accuracy, topic accuracy with
Mastered / Developing / Needs Review, strongest topic, recommended focus,
EMA mastery per topic with quiz and conversation counts, and
**🧠 Likely Misconceptions** — the last 3 incorrect answers with topic, question,
student answer, correct answer, explanation and source citation (nothing that did
not come from the student's own answers).

## Diagnostic / cold start

Three MCQs (Easy → Medium → Hard) generated and verified from the uploaded material,
focused on the learner's stated goal. Score 0–1 → Beginner/Easy, 2 →
Intermediate/Medium, 3 → Advanced/Hard; sets the learner level and the starting
assessment difficulty. The result is applied **once** per diagnostic run, so the
assessment can adapt difficulty afterwards. Each diagnostic answer also updates
`learner_mastery`. The Start button is disabled until material is uploaded.

## Personalization

The tutor prompt includes learner level (Beginner / Intermediate / Advanced),
learning goal, explanation style (Concise / Balanced / Detailed) and weak topics:
topic accuracy < 70 % **or** EMA mastery < 0.50 (so diagnostic and conversation
evidence also count). Level and style map to explicit instructions; grounding
rules take precedence.

## Persistence

`learner_state.json` (atomic write via temp file + `os.replace`): learner profile,
diagnostic result, quiz score/total/difficulty/streak, topic performance, quiz
history, `learner_mastery`, `conversation_counts`. Only JSON-native types are
stored (the in-memory `seen_questions` set is not persisted).

- A missing, corrupt or partially invalid file never crashes the app: values with
  the wrong type are replaced by defaults (`ensure_core_state`), unknown profile
  values fall back to Intermediate / Balanced.
- If the file cannot be written (e.g. locked by a sync client) a warning is shown
  and progress stays in the session.
- **Reset Assessment** clears only the current assessment session (score, total,
  streak, current question); learner history — topic performance, quiz history,
  mastery, conversation counts — is kept, and the diagnostic-based difficulty is
  preserved when a diagnostic has been completed.

## RAGAS evaluation

Single-question evaluation (Evaluation tab): retrieve → generate → score with
RAGAS **Faithfulness, Answer Relevancy, Context Precision, Context Recall**.
Judge LLM: Gemini via its OpenAI-compatible endpoint (`AsyncOpenAI` +
`llm_factory`); embeddings via `FixedGeminiRagasEmbeddings` (fixed 768-d to avoid
dimension mismatches). Unavailable (NaN) scores are shown as "—".

## Batch benchmark

**📚 Benchmark Test Set** runs `evaluation_testset.json` (team-built, 5 in-material
questions on *Kümeleme.pptx* + 1 off-material question) through the same
pipeline. Per case: four RAGAS metrics (in-material), refusal correctness, and
whether the expected citation appears in the retrieved sources or the answer.

- **Quick Benchmark** (default, quota-safe): the off-material case first (cheapest,
  no RAGAS calls), then the first 2 in-material cases. **Full Benchmark**: all 6.
- 5 s pause between cases. On a Gemini 429 / `RESOURCE_EXHAUSTED` the case is
  marked `quota_error`, remaining cases are `skipped` (no further calls).
- Scores are computed only from fully successful cases. With 0 successful cases
  no metric cards are shown ("Benchmark stopped — Gemini API quota reached.");
  partial runs are labelled partial.
- A warning appears if the test set's expected file (e.g. `Kümeleme.pptx`) is not
  loaded; empty retrieval is recorded as a case error.
- Results: `evaluation_results.json` and `evaluation_results.csv` with
  `run_status`, `successful_cases`, `failed_cases`, `quota_exhausted`.

## Simulated personalization benchmark

**🧑‍🎓 Personalization Benchmark** (Evaluation tab) is explicitly **synthetic**.
Three profiles replay fixed evidence through the real `ema_update` rule, two
sessions each; deterministic and API-free. These are not real-student results.

| Profile | Initial | After S1 | After S2 | Gain |
|---|---|---|---|---|
| Beginner (✗ ✗ ✓ / ✓ ✓ ✓) | 0.30 | 0.403 | 0.795 | +0.495 |
| Intermediate (✗ ✓ ✓ / ✓ ✓ ✓) | 0.50 | 0.681 | 0.891 | +0.391 |
| Advanced (✓ ✓ ✓ / ✓ ✓ ✓) | 0.70 | 0.897 | 0.965 | +0.265 |

The real question repetition rate is shown next to it.

## Track D checklist

| Requirement | Implementation | Status |
|---|---|---|
| Multimodal ingestion (text, slides, video, images) | PDF text + figures, PPTX text/tables/images, Gemini video segments, image Q&A | ✅ Done |
| Source-grounded answers with citations | Page / slide / timestamp citations, strict grounding prompt | ✅ Done |
| Refusal of unsupported questions | Fixed refusal sentence, normalized detection; measured by Refusal Accuracy | ✅ Done |
| Unified knowledge base | PDF + slides + video in one index with scope filter | ✅ Done |
| Content units tagged (topic, subtopic, key concepts, prerequisites) | Batched Gemini tagging, cached; automatic for used units + manual for all; Knowledge Structure view | ✅ Done |
| Adaptive assessment (MCQ / short / numerical) | Easy/Medium/Hard adaptation, semantic grading | ✅ Done |
| Question verification | Second-pass verifier incl. self-containment and hidden-visual rejection; PDF figures in generation + verification | ✅ Done |
| No repeated questions + repetition rate | Similarity filter + Question Repetition Rate metric | ✅ Done |
| Learner model updated after every quiz | `topic_performance` + EMA `learner_mastery`, canonical topics | ✅ Done |
| Learner model updated after every conversation | Conversation-based heuristic EMA + `conversation_counts` | ✅ Done (heuristic) |
| Weak topics + likely misconceptions | Dashboard recommended focus + Likely Misconceptions from wrong answers | ✅ Done |
| Cold-start diagnostic | 3-question Easy→Medium→Hard diagnostic | ✅ Done |
| Personalization (level, goal, style, weak topics) | Personalized tutor prompt using accuracy and EMA mastery | ✅ Done |
| Persistence across sessions | `learner_state.json`, self-repairing on load | ✅ Done |
| RAGAS evaluation | Single question + quota-safe batch benchmark, JSON/CSV export | ✅ Done |
| Simulated student profiles across multiple sessions | Synthetic 3-profile × 2-session benchmark | ✅ Done (synthetic) |

### Optional enhancements

Not implemented / intentionally out of current scope: flashcards / spaced
repetition, study schedule, audio tutor, course map visualization, login /
multi-user accounts.

## Known limitations

- Conversation-based mastery is a transparent heuristic, not proof of learning.
- Canonical topics depend on knowledge tags. Without tags (tagging off or failed)
  the generator's free-text topic is used, so one concept can still appear under
  two names. Tag topics are broad sections, so mastery is tracked per section.
- Gemini free-tier quotas are small: a Full Benchmark (≈ 10+ calls per in-material
  case through RAGAS) often hits 429; use Quick Benchmark and retry after reset.
- The benchmark is small (6 cases, one deck). Expected-source matching assumes the
  deck is uploaded as `Kümeleme.pptx` with the listed slide numbers.
- Refusal detection is phrase-based: a refusal worded completely differently
  would not be recognized.
- RAGAS uses the same Gemini model family as judge (possible self-evaluation bias);
  RAGAS scores can vary between runs.
- The personalization benchmark is simulated; there is no real-student study.
- Assessment questions are text-only; questions that would need a figure are
  rejected rather than shown with the figure.
- Uploaded material lives in the Streamlit session; it must be re-uploaded after
  a browser refresh (learner progress persists).
- Single-user local JSON storage; no authentication.

## How to run

```bash
python -m venv venv
venv\Scripts\activate          # Windows  (macOS/Linux: source venv/bin/activate)
pip install -r requirements.txt
copy .env.example .env         # macOS/Linux: cp .env.example .env  → set GEMINI_API_KEY
streamlit run app.py
```

Generated locally (git-ignored): `learner_state.json`, `knowledge_tags.json`,
`evaluation_results.json`, `evaluation_results.csv`. `evaluation_testset.json` is
committed and recreated with the default set if missing.

## Demo flow

1. **Learner Profile** — set level, goal, style.
2. **Slide Understanding** — upload `Kümeleme.pptx`; optionally a PDF and a video.
3. **Learner Profile → Start Diagnostic** (3 questions) — level and starting difficulty.
4. **Unified Tutor** — ask "Minkowski uzaklığı nedir?" (cited answer + learner-model
   update), then an off-material question (refusal). Open *🧩 Knowledge Structure*.
5. **Adaptive Assessment** — answer a few questions; show difficulty adapting.
6. **Mastery Dashboard** — accuracy, EMA mastery, recommended focus,
   Likely Misconceptions.
7. **Evaluation** — single RAGAS question, then *Quick Benchmark*; show averages,
   refusal accuracy, source match, repetition rate, synthetic personalization table.
