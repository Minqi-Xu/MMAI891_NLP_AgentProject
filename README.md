# Adaptive Study Agent

Adaptive Study Agent is a Streamlit application that turns user-provided learning materials into a guided study workflow:
- summarize content
- extract key concepts
- generate a 5-question adaptive quiz
- evaluate responses and confidence
- provide targeted explanations
- track improvement by topic over time

The app supports `.txt`, `.pdf`, `.docx`, and `.pptx` uploads.

## Core Features

- Multi-file ingestion with long-document chunking
- Grounded generation (quiz content constrained to uploaded material)
- Comprehensive concept extraction (not limited to 5 concepts)
- Post-summary concept validation to filter weak or ungrounded concepts
- Fixed 5-question quizzes using weighted concept sampling
  - random concept selection across extracted concepts
  - higher weight for historically weak concepts
- Post-quiz grounding validation with strict verify/repair loop:
  - verify each question is answerable from uploaded materials
  - keep valid questions unchanged
  - regenerate invalid questions using the same quality rules as initial generation and validate again
  - notify the user when conservative fallback questions are used after repeated failed validation
- Confidence-based difficulty routing
  - confidence 1-2: foundational
  - confidence 3: standard
  - confidence 4-5: advanced
- Persistent local topic memory (`data/user_memory.json`)
- Quiz History page with:
  - first vs latest improvement deltas
  - trend chart (accuracy + confidence)
  - concept-level error rates

## Project Structure

```text
.
├── Main_Page.py
├── README.md
├── requirements.txt
├── install.sh
├── install_transcript.txt
├── pages/
│   └── Quiz_History.py
├── evaluation/
│   └── user_progress_report.py
├── data/
│   ├── synthetic_biology_notes.txt
│   └── user_memory.json
└── secrets/
    ├── openai_api_key.txt.example
    └── openai_api_key.txt
```

## Quick Start

Run from project root:

```bash
./install.sh
```

`install.sh` will:
- create `.venv`
- install dependencies
- verify required imports
- start the Streamlit app

Windows note:
- Run `./install.sh` from Git Bash.
- The script supports both `.venv/bin` (macOS/Linux) and `.venv/Scripts` (Windows).

## Manual Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
streamlit run Main_Page.py
```

## OpenAI API Configuration

Choose one option:

1. Local key file (recommended for this project)
```bash
cp secrets/openai_api_key.txt.example secrets/openai_api_key.txt
# Replace placeholder with your real key
```

2. Environment variable
```bash
export OPENAI_API_KEY="sk-..."
```

If no key is available, the app runs in fallback mode.

## Using the App

1. Enter a topic.
2. Upload one or more files.
3. Click `Generate Summary + Initial Quiz`.
4. Answer quiz questions and set confidence for each response.
5. Submit quiz to see:
   - performance report
   - targeted explanations
   - explanation blocks mapped to specific missed questions (Q# + question text)
   - recommended study actions
   - next adaptive quiz preview
6. Open `Quiz History` page to review progress by topic.

## Data and Privacy

- Uploaded files are parsed via temporary files and removed immediately after extraction.
- Raw uploaded files are not permanently stored by default.
- Persistent data stored locally:
  - `data/user_memory.json` (topic stats, quiz outcomes, concept error rates)
- You can clear runtime output and/or saved memory from in-app controls.

## Progress Evaluation (CLI)

Generate a console report from saved history:

```bash
python evaluation/user_progress_report.py
```

The report includes:
- per-topic first vs latest score/accuracy/confidence
- per-topic deltas
- aggregate deltas for topics with at least two attempts

## Troubleshooting

- `No such file or directory: .venv/bin/python` on Windows:
  - use Git Bash and run `./install.sh` (already handles Windows venv paths).
- `No API key detected`:
  - add key to `secrets/openai_api_key.txt` or set `OPENAI_API_KEY`.
- Unsupported upload type:
  - use one of: TXT, PDF, DOCX, PPTX.
