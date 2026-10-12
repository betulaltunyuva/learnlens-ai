<p align="center">
  <img src="assets/banner.svg" alt="LearnLens AI — multimodal, source-grounded adaptive tutor" width="100%">
</p>

<p align="center">
  <b>A multimodal, source-grounded AI tutor that learns from your own course material — PDFs, slides and lecture videos — adapts its questions to you, and refuses to make things up.</b>
</p>

<p align="center">
  <img alt="Multimodal AI Hackathon 2026 — Track D" src="https://img.shields.io/badge/Multimodal%20AI%20Hackathon%202026-Track%20D-4F46E5?style=for-the-badge"><br>
  <i>Track D: Personalized Tutoring &amp; Adaptive Learning</i>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white">
  <img alt="Streamlit" src="https://img.shields.io/badge/Streamlit-UI-FF4B4B?logo=streamlit&logoColor=white">
  <img alt="Google Gemini" src="https://img.shields.io/badge/Google%20Gemini-API-8E75B2?logo=googlegemini&logoColor=white">
  <img alt="RAGAS" src="https://img.shields.io/badge/RAGAS-0.4.3-0F172A">
  <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-22C55E">
</p>

<p align="center">
  <a href="https://youtu.be/5sruuk1WHsA">
    <img src="https://img.youtube.com/vi/5sruuk1WHsA/maxresdefault.jpg" alt="Watch the LearnLens AI demo video on YouTube" width="720">
  </a><br>
  <b>▶ <a href="https://youtu.be/5sruuk1WHsA">Watch the 3-minute demo</a></b>
</p>

<p align="center">
  <a href="https://youtu.be/5sruuk1WHsA">Demo video</a> ·
  <a href="#screenshots">Screenshots</a> ·
  <a href="#track-d-alignment">Track D alignment</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#sample-run-quick-benchmark-10-oct-2026">Results</a> ·
  <a href="#installation">Installation</a> ·
  <a href="#demo-flow">Demo flow</a> ·
  <a href="TRACK_D_STATUS.md">Track D status</a> ·
  <a href="SUBMISSION.md">Submission summary</a>
</p>

---

## Problem

Students study from a mix of lecture slides, PDF notes and recorded lectures.
General-purpose chatbots answer from the open internet rather than from *this*
course, they hallucinate confidently, they do not cite where an answer came from,
and they do not know what a particular student already understands. Practice
questions are generic, repeat themselves, and do not get harder or easier as the
student improves.

## Solution

LearnLens AI turns a student's own material into one searchable multimodal
knowledge base and builds a tutor around it:

- **Grounded answers** with exact citations — `[file.pdf, Page 3]`,
  `[deck.pptx, Slide 7]`, `[lecture.mp4, 02:10 - 03:05]`.
- **Honest refusal** when the material does not contain the answer:
  *"Bu bilgi yüklenen ders materyallerinde bulunamadı."*
- **Adaptive, verified assessments** (MCQ, short answer, numerical) whose
  difficulty follows the student's performance.
- **A learner model** that tracks mastery per topic from quizzes, the diagnostic
  and tutoring conversations, and uses it to personalize explanations.
- **Built-in evaluation**: RAGAS metrics, a team-built benchmark, refusal accuracy,
  citation matching and question repetition rate.

## Why LearnLens AI is different

- **One knowledge base for all modalities.** PDF pages (including embedded
  figures), slides (text, tables, images) and timestamped video segments are
  retrieved together, so one answer can cite a slide and a video timestamp.
- **Grounding is enforced and measured.** Strict prompts, citation preservation,
  a fixed refusal sentence, and a benchmark that checks refusals and expected
  sources.
- **Questions are checked twice.** A second Gemini pass verifies every generated
  question against the source and rejects anything ambiguous, unsupported or not
  self-contained (including questions that would need a figure the student
  cannot see).
- **Transparent learner model.** A simple exponential moving average per topic,
  documented constants, no black box.
- **Quota-aware.** Embeddings and knowledge tags are cached, tagging is batched
  and need-based, and the benchmark stops cleanly on API quota errors.

## Screenshots

<table>
  <tr>
    <td width="50%"><img src="assets/screenshots/learner-profile.png" width="410" alt="Learner Profile tab with level, goal, explanation style and diagnostic status"></td>
    <td width="50%"><img src="assets/screenshots/mastery-dashboard.png" width="410" alt="Mastery Dashboard with overall accuracy, strongest topic, recommended focus and topic performance"></td>
  </tr>
  <tr>
    <td align="center"><b>🧭 Learner Profile</b> — level, goal, explanation style and the cold-start diagnostic</td>
    <td align="center"><b>📊 Mastery Dashboard</b> — accuracy, strongest topic, recommended focus</td>
  </tr>
  <tr>
    <td width="50%"><img src="assets/screenshots/learner-mastery-misconceptions.png" width="410" alt="Per-topic EMA mastery table and Likely Misconceptions card with a cited correct answer"></td>
    <td width="50%"><img src="assets/screenshots/evaluation.png" width="410" alt="Evaluation tab with question repetition rate and the synthetic personalization benchmark"></td>
  </tr>
  <tr>
    <td align="center"><b>🧩 EMA learner model & 🧠 Likely Misconceptions</b> — built only from the student's own wrong answers, with source citation</td>
    <td align="center"><b>🧪 Evaluation</b> — question repetition rate and the clearly labelled synthetic personalization benchmark</td>
  </tr>
</table>

## Track D alignment

| Track D requirement | LearnLens AI |
|---|---|
| Multimodal content understanding | PDF text + figures, PPTX text/tables/images, video segments, image Q&A |
| Knowledge structure per content unit | Topic, subtopic, key concepts, prerequisites per unit (batched Gemini tagging, cached) |
| Source-grounded tutoring | Retrieval-augmented answers with page / slide / timestamp citations and refusal |
| Adaptive assessment | MCQ / short answer / numerical, Easy ↔ Medium ↔ Hard adaptation, semantic grading |
| Question quality | Second-pass verification, self-containment, duplicate prevention, repetition rate |
| Learner model updated after quizzes **and** conversations | EMA mastery per topic + topic accuracy + conversation counts |
| Weak topics and misconceptions | Recommended focus + Likely Misconceptions from the student's own wrong answers |
| Cold start | 3-question diagnostic (Easy → Medium → Hard) |
| Personalization | Level, goal, explanation style and weak topics injected into the tutor |
| Persistence | `learner_state.json` across sessions |
| Evaluation | RAGAS (4 metrics), batch benchmark, refusal accuracy, source match, repetition rate, **synthetic** multi-session profile simulation |

A detailed requirement-by-requirement status is in
[TRACK_D_STATUS.md](TRACK_D_STATUS.md).

## Main features

| Tab | What it does |
|---|---|
| 🧭 Learner Profile | Level, learning goal, explanation style; 3-question starting-level diagnostic |
| 📄 PDF RAG | Upload PDFs; page-level retrieval with figures; cited answers |
| 📽️ Slide Understanding | Upload PPTX; slide text, tables and embedded images; cited answers |
| 🎞️ Video Understanding | Upload a lecture video; Gemini builds timestamped knowledge segments |
| 🖼️ Image Understanding | Ask questions about a diagram, photo or screenshot |
| 🎓 Unified Tutor | Personalized, grounded answers across all uploaded material; Knowledge Structure view |
| 🧠 Adaptive Assessment | Verified MCQ / short / numerical questions with adaptive difficulty |
| 📊 Mastery Dashboard | Accuracy, EMA topic mastery, strongest / weakest topic, likely misconceptions |
| 🧪 Evaluation | RAGAS single question, benchmark test set, repetition rate, synthetic personalization benchmark |

## Multimodal capabilities

| Input | Processing | Citation |
|---|---|---|
| PDF | PyMuPDF text, chunked per page (1200 chars, 200 overlap); up to 2 embedded figures per page, sent to Gemini when the page is retrieved | `[file.pdf, Page N]` |
| PowerPoint | python-pptx text frames, tables, embedded pictures (up to 2 per retrieved slide sent to Gemini) | `[file.pptx, Slide N]` |
| Video | Gemini Files API; Gemini segments the lecture into timestamped topic segments that are embedded like text | `[video.mp4, mm:ss - mm:ss]` |
| Image | Direct visual question answering with Gemini | — |

## Architecture

```mermaid
flowchart TB
    subgraph S1["① Multimodal ingestion"]
        direction TB
        PDF["📄 PDF<br/>PyMuPDF text + figures"]
        PPTX["📽️ PPTX<br/>text · tables · images"]
        VID["🎞️ Video<br/>Gemini Files API →<br/>timestamped segments"]
    end
    subgraph S2["② Unified knowledge base"]
        direction LR
        EMB["Gemini embeddings<br/>gemini-embedding-2 · 768-d"] --> KB[("Units + citations<br/>+ knowledge tags")]
    end
    subgraph S3["③ Grounded tutoring & assessment"]
        direction LR
        RET["Cosine retrieval<br/>top-k · scope filter"] --> GEN["Gemini generation<br/>gemini-3.5-flash-lite"]
        GEN --> ANS["✅ Cited answer<br/>or explicit refusal"]
        GEN --> Q["🧠 Verified adaptive<br/>question + grading"]
    end
    subgraph S4["④ Learner model"]
        direction LR
        LM[("EMA mastery · topic accuracy<br/>conversation counts")] --> ST[("learner_state.json")]
    end
    S1 --> S2 --> S3 --> S4
    S4 -. "level · goal · style · weak topics" .-> S3
```

Images are answered directly with Gemini vision in the 🖼️ Image Understanding tab.

Everything runs in one Streamlit app (`app.py`). Gemini is the only AI provider.

## How source-grounded RAG works

1. Each PDF chunk, slide and video segment becomes a **knowledge unit** with a
   fixed citation string.
2. The question is embedded and the top units are retrieved by cosine similarity
   (optionally limited to PDF / Slides / Video).
3. Gemini receives only those units (plus up to 2 images per retrieved page or
   slide) and strict rules: answer only from the material, keep the exact
   citation for every important claim, and otherwise reply
   *"Bu bilgi yüklenen ders materyallerinde bulunamadı."*
4. The UI shows the answer, the sources used (with similarity scores and topic),
   and marks refusals explicitly. Refusal detection is deterministic and ignores
   case, Turkish character variants, Markdown and punctuation.

## Adaptive assessment approach

- **Generation:** source units are sampled (or retrieved for a chosen topic); Gemini
  writes one question with a required exact citation, varying the "question focus"
  each attempt. PDF figures of those pages help the model understand the material.
- **Checks before acceptance:** valid JSON, exact allowed citation, 4 options with
  the key among them (MCQ), not similar to the last 15 questions (normalized
  `SequenceMatcher` ≥ 0.82), no references to hidden visuals ("the figure",
  "şekildeki", "slaytta" …).
- **Verification:** a second Gemini pass (with the same images) must confirm the
  question is answerable from the material alone, correct, unambiguous and fully
  self-contained. Otherwise another question is generated (up to 8 attempts).
- **Adaptation:** 2 correct answers in a row → harder; 1 wrong answer → easier.
- **Grading:** MCQ by exact option; short and numerical answers semantically by
  Gemini (equivalent numerical forms accepted).

## Learner model (EMA)

For every topic, LearnLens keeps a mastery estimate between 0 and 1:

```
new_mastery = old_mastery × (1 − α) + evidence × α        (new topic starts at 0.50)
```

| Evidence source | evidence | α |
|---|---|---|
| Assessment or diagnostic answer — correct / wrong | 1.0 / 0.0 | 0.30 |
| Tutor conversation (heuristic) | 0.55 | 0.05 |
| Tutor conversation with an explicit difficulty signal ("anlamadım", "zorlanıyorum", "I'm confused" …) | 0.35 | 0.08 |

Conversation evidence is deliberately weak: asking about a topic is not proof of
learning. Off-material (refused) questions do not change the model.

**Canonical topics:** a question's topic is the knowledge tag of the unit it cites,
and tutor conversations use the tag of the top retrieved unit, so quizzes,
mastery, conversation counts and the dashboard refer to the same topic. Labels
that differ only in case or whitespace are unified; different topics are never
merged.

## Personalization

The tutor prompt includes the learner's **level** (Beginner / Intermediate /
Advanced), **learning goal**, **explanation style** (Concise / Balanced /
Detailed) and up to three **weak topics** (topic accuracy < 70 % or EMA mastery
< 0.50). Each level and style maps to explicit writing instructions. Grounding
rules always take precedence: personalization may change *how* something is
explained, never *what* the material says.

## Diagnostic / cold start

New learners take a 3-question diagnostic (Easy → Medium → Hard MCQs generated and
verified from their own material, focused on their stated goal):

| Score | Estimated level | Starting difficulty |
|---|---|---|
| 0–1 | Beginner | Easy |
| 2 | Intermediate | Medium |
| 3 | Advanced | Hard |

Each diagnostic answer also updates EMA mastery for its topic.

## Evaluation methodology

All evaluation lives in the **🧪 Evaluation** tab and runs against the material
you upload:

- **Single-question RAGAS evaluation** — your question + reference answer.
- **Benchmark test set** — [`evaluation_testset.json`](evaluation_testset.json),
  team-built for a clustering lecture deck (*Kümeleme.pptx*): 5 in-material
  questions with reference answers and expected slide citations, plus 1
  off-material question that must be refused.
  - *Quick Benchmark* (default, 3 cases, quota-safe) or *Full Benchmark* (6 cases).
  - Per case: RAGAS metrics, refusal correctness, expected-source match
    (citation found in the retrieved sources or in the answer).
  - Summary: average of each RAGAS metric, **Refusal Accuracy**,
    **Citation / Expected Source Match**; exported to `evaluation_results.json`
    and `evaluation_results.csv`.
  - API quota errors stop the run cleanly; with no successful case, no scores
    are shown.
- **Question Repetition Rate** — share of accepted assessment questions that are
  similar to an earlier one (same rule as duplicate prevention).
- **Personalization Benchmark (synthetic)** — three *simulated* learner profiles
  replayed through the real EMA update over two sessions. Deterministic, no API
  calls, and clearly labelled as simulation — **not** real-student evidence.

### Sample run (Quick Benchmark, 10 Oct 2026)

One complete Quick Benchmark run on *Kümeleme.pptx* (plus one PDF in the same
session) with `gemini-3.5-flash-lite`. Raw output:
[`results/quick_benchmark_2026-10-10.json`](results/quick_benchmark_2026-10-10.json)
· [`.csv`](results/quick_benchmark_2026-10-10.csv).

| Metric | Result | Cases |
|---|---|---|
| Faithfulness (avg) | **0.71** | 2 in-material |
| Answer Relevancy (avg) | **0.94** | 2 in-material |
| Context Precision (avg) | **0.75** | 2 in-material |
| Context Recall (avg) | **1.00** | 2 in-material |
| Refusal Accuracy (refuse vs. answer decided correctly) | **3 / 3** | 1 off-material + 2 in-material |
| Expected Source Match | **2 / 2** | 2 in-material |

> ⚠️ This is a **single, small run** (3 cases), shown as evidence that the
> pipeline works end to end, not as a general accuracy claim. Scores depend on
> the uploaded material, the model and API quota, and RAGAS scores vary between
> runs. Run the *Full Benchmark* to evaluate all 6 cases.

**Example from this run** — the question *"Minkowski uzaklığı nedir?"*
("What is the Minkowski distance?") was answered with citations on every claim
(excerpt):

> Minkowski uzaklığı, *p* sayıda değişken göz önüne alınarak gözlem değerleri
> arasındaki mesafeyi hesaplamak için kullanılan genel bir uzaklık bağıntısıdır
> **[Kümeleme.pptx, Slide 7]**. Kümeleme çözümlemelerinde Öklit ve Manhattan
> uzaklıkları ile birlikte tercih edilen bu ölçüt **[Kümeleme.pptx, Slide 5]** …
>
> *(EN: "Minkowski distance is a general formula for the distance between
> observations over p variables [Slide 7]. Used in cluster analysis together with
> the Euclidean and Manhattan distances [Slide 5] …")*

The off-material question *"Yapay sinir ağlarında backpropagation algoritması
nedir?"* ("What is backpropagation in neural networks?") was refused with the
fixed sentence *"Bu bilgi yüklenen ders materyallerinde bulunamadı."*
("This information was not found in the uploaded course material.")

### Try it with the included sample material

The Turkish lecture deck used for the run above is course material and is
**not included** in this repository. Instead,
[`sample_materials/`](sample_materials/) contains an English sample lecture on
clustering, written for this project and used in the demo video:

| File | Modality | Content |
|---|---|---|
| `Clustering_Lecture.pptx` | Slides (8, with 3 figures and a table) | Distance measures, distance matrix, hierarchical clustering, linkage methods, k-means, choosing K |
| `kmeans_notes.pdf` | PDF (2 pages, with figures) | k-means objective, algorithm, worked example, elbow method |
| `hierarchical_clustering_lecture.mp4` | Video (1:48, narrated) | Reading a dendrogram, single vs. complete linkage, hierarchical clustering vs. k-means |
| `evaluation_testset_sample.json` | Benchmark test set | 5 in-material questions with expected slide / page citations + 1 off-material question |

To benchmark it, copy the sample test set over the default one, upload the
`.pptx` and `.pdf`, and click **Run Benchmark** in the **🧪 Evaluation** tab:

```bash
# Windows
copy sample_materials\evaluation_testset_sample.json evaluation_testset.json
# macOS / Linux
cp sample_materials/evaluation_testset_sample.json evaluation_testset.json
```

`git checkout evaluation_testset.json` restores the original test set. In our
check on 12 Oct 2026, a Quick Benchmark on this sample set completed 3 / 3
cases with refusal accuracy 3 / 3 and expected-source match 2 / 2 (again a
small run, not a general accuracy claim).

### Running the benchmark on your own material

To benchmark any material of your own, edit
[`evaluation_testset.json`](evaluation_testset.json):

```json
{
  "id": "my_case_01",
  "question": "What is k-means trying to minimize?",
  "reference_answer": "The total within-cluster squared error.",
  "expected_source": "my_deck.pptx, Slide 12",
  "should_refuse": false
}
```

Add at least one `"should_refuse": true` question that your material does not
cover, upload the files, and run the benchmark in the **🧪 Evaluation** tab.

### Language

LearnLens is not tied to Turkish: tutor answers are written in the language of
the student's question (also when the material is in another language), and
generated questions follow the language of the material. The benchmark deck
above is Turkish; the sample material and the demo video are English. The fixed
refusal sentence is Turkish (*"This information was not found in the uploaded
course material."*); English refusal phrasings are recognised as well.

## RAGAS metrics

| Metric | Question it answers |
|---|---|
| **Faithfulness** | Are the answer's claims supported by the retrieved context? |
| **Answer Relevancy** | Does the answer address the question? |
| **Context Precision** | Are the retrieved units relevant (and ranked well) for the reference answer? |
| **Context Recall** | Does the retrieved context contain what the reference answer needs? |

RAGAS 0.4.3 uses Gemini as judge through Gemini's OpenAI-compatible endpoint
(the `openai` Python client, no OpenAI account), and a fixed 768-dimensional Gemini
embedding wrapper so that dimensions always match.

## Known limitations

- The learner model is a transparent heuristic (EMA), not a validated
  psychometric model; conversation evidence in particular is a heuristic.
- No study with real students has been run; the personalization benchmark is
  synthetic.
- Canonical topics rely on knowledge tags; without tags the generator's
  free-text topic is used. Tag topics are broad sections.
- Gemini free-tier quotas are small. RAGAS makes many calls per case, so the Full
  Benchmark can hit HTTP 429; Quick Benchmark is the default.
- The benchmark is small (6 cases, one lecture deck); expected-source matching
  assumes the deck is uploaded as `Kümeleme.pptx`, which is not redistributed
  here. The English sample set in `sample_materials/` can be run instead (see
  *Try it with the included sample material*).
- RAGAS uses the same model family as the tutor as its judge (possible
  self-evaluation bias); scores vary between runs.
- Refusal detection is phrase-based (deterministic, no extra LLM call).
- Assessment questions are text-only: questions that would need a figure are
  rejected instead of displayed with the figure.
- Uploaded material lives in the browser session and must be re-uploaded after a
  refresh; learner progress persists in `learner_state.json`.
- Single-user, local JSON storage; no authentication.

## Tech stack

- **Python 3.10+**, **Streamlit** (UI)
- **Google Gemini** via the `google-genai` SDK
  - `gemini-3.5-flash-lite` — answers, question generation and verification,
    grading, video segmentation, knowledge tagging, RAGAS judge
  - `gemini-embedding-2` (768-d) — retrieval embeddings
  - Gemini Files API — video upload and understanding
  - Gemini OpenAI-compatible endpoint — RAGAS judge via the `openai` client
- **PyMuPDF** (PDF text and figures), **python-pptx** (slides), **NumPy**
  (vector search)
- **RAGAS 0.4.3** (evaluation), **python-dotenv**

## Installation

Requirements: Python 3.10 or newer and a Gemini API key
(free at [Google AI Studio](https://aistudio.google.com/apikey)).

```bash
git clone https://github.com/betulaltunyuva/learnlens-ai.git LearnLens-AI
cd LearnLens-AI

python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

## Environment setup

```bash
# Windows
copy .env.example .env
# macOS / Linux
cp .env.example .env
```

Edit `.env` and set your key:

```
GEMINI_API_KEY=your_api_key_here
```

`.env` is git-ignored. Never commit a real key.

## Run

```bash
streamlit run app.py
```

The app opens at http://localhost:8501. Local files created while running
(all git-ignored): `learner_state.json`, `knowledge_tags.json`,
`evaluation_results.json`, `evaluation_results.csv`.

## Demo flow

1. **🧭 Learner Profile** — choose a level, enter a goal, pick a style, *Save*.
2. **📽️ Slide / 📄 PDF / 🎞️ Video** — upload the files from
   [`sample_materials/`](sample_materials/) (or your own lecture material).
3. **🧭 Learner Profile → Start Diagnostic** — answer 3 questions; the level and
   starting difficulty update (see the sidebar).
4. **🎓 Unified Tutor** — ask *"When should I use hierarchical clustering
   instead of k-means?"* → one answer citing slides, a PDF page and a video
   timestamp, plus a learner-model update; then ask *"What is backpropagation
   in neural networks?"* → refusal. Open *🧩 Knowledge Structure*.
5. **🧠 Adaptive Assessment** — answer a few questions; watch *Next Difficulty*.
6. **📊 Mastery Dashboard** — topic accuracy, EMA mastery, recommended focus,
   Likely Misconceptions.
7. **🧪 Evaluation** — single RAGAS question, then *Run Benchmark* (Quick); show
   the repetition rate and the synthetic personalization table.

## Repository structure

```
LearnLens-AI/
├── app.py                   # Streamlit application (all features)
├── .streamlit/config.toml   # UI theme (colors, font)
├── requirements.txt         # Python dependencies
├── .env.example             # Template for GEMINI_API_KEY
├── .gitignore
├── evaluation_testset.json  # Team-built benchmark test set
├── results/                 # Snapshot of a complete Quick Benchmark run
├── sample_materials/        # English sample lecture (PPTX, PDF, video) + sample test set
├── assets/
│   ├── banner.svg           # README banner
│   └── screenshots/         # README screenshots
├── LICENSE                  # MIT
├── README.md
├── SUBMISSION.md            # Devpost-ready project summary
└── TRACK_D_STATUS.md        # Requirement-by-requirement implementation status
```

## Responsible AI and grounding

- LearnLens answers only from material the learner uploaded and cites the exact
  page, slide or timestamp so every claim can be checked.
- When the material does not contain the answer, it says so instead of guessing.
- Assessment questions are verified against the source before they are shown.
- Learner-model rules are simple and documented; misconceptions are shown only
  from the student's own wrong answers, never invented.
- Simulated results are labelled as simulated; no real-student outcomes are
  claimed.
- Learner data stays local in a JSON file. Uploaded material is sent to the
  Gemini API for processing, subject to Google's API terms. Do not upload
  material you are not allowed to share with that service.
- LearnLens supports learning; it does not replace teachers or official course
  information.

## AI-assisted development disclosure

AI coding assistants including ChatGPT and Claude Code were used during
development for code review, debugging and implementation assistance. All
generated code was reviewed, integrated and tested by the project author.

At runtime, the application itself uses Google Gemini models as described in the
tech stack.

## License

Released under the [MIT License](LICENSE).

## Author

**Betül Altunyuva** — [GitHub](https://github.com/betulaltunyuva) ·
[Devpost](https://devpost.com/betulaltunyuva)
