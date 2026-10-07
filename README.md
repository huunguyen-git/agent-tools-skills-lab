# Agentic AI — Tool Use and Skill Use

Implementation of the two D05 assignments using a five-stage agent lab.

- [Block 1 assignment](https://pub.nndkhoa9.win/agentic-ai-engineering/d05-tool-use-skill-use/exercise-block-1.html): discover policy files and apply the correct refund policy through a skill.
- [Block 2 assignment](https://pub.nndkhoa9.win/agentic-ai-engineering/d05-tool-use-skill-use/exercise-block-2.html): validate workload CSVs, calculate owner totals and identify overload using a reusable script.
- [Combined assignment analysis](analysis.md) explains the implementation, observed model behavior and trace events.
- [Evidence](evidence/) contains selected real-model traces, direct results, test summaries and generated workload reports.

## Stages

| Stage | Capabilities and assignment role |
|---|---|
| `stage-00-chat` | Chat-only baseline |
| `stage-01-files` | File tools plus `list_files` for policy discovery |
| `stage-02-skills` | Refund-policy skill and answer reference |
| `stage-03-bash` | Bash baseline with model-generated Python |
| `stage-04-script-skill` | CSV quality/workload skill, report reference and deterministic script |

Assignment data and skills are present in both `workspace/` and `fixtures/`. Resetting a stage restores its fixtures and removes generated output. Selected evidence is preserved separately.

## Setup

Requires Python 3.11+ and uv. From the repository root:

```bash
uv sync --all-packages --locked
```

In each stage you intend to run, copy `.env.example` to `.env` and configure:

```dotenv
OPENAI_API_KEY=your-provider-key
MODEL_NAME=your-tool-calling-model
OPENAI_BASE_URL=your-compatible-endpoint
```

Credentials are private and excluded from Git. Leave the base URL unset for the default OpenAI endpoint. For Gemini's OpenAI-compatible endpoint, use `https://generativelanguage.googleapis.com/v1beta/openai/` and the included launcher so tool-call metadata survives subsequent requests.

Run a stage from its own directory:

```bash
cd stage-02-skills
uv run --locked streamlit run runtime/gemini_app.py
```

Open the local URL printed by Streamlit. Set `--server.port` to run multiple stages at once. The original `app.py` can be used with providers that do not require the Gemini adapter.

Stages 03–04 require POSIX Bash. On Windows, start Docker Desktop, then run from the root:

```powershell
powershell -ExecutionPolicy Bypass -File scripts/start_bash_stages.ps1
```

This mounts the repository directly into Linux and uses a separate Python environment. Stage 03 runs at `http://127.0.0.1:8513`; Stage 04 at `http://127.0.0.1:8514`.

## Validation

Run each stage's tests from that stage's directory:

```bash
uv run --locked pytest -q -rs
```

For the Bash stages on Windows:

```powershell
docker exec -w /lab/stage-03-bash agent-lab-posix uv run --locked pytest -q -rs
docker exec -w /lab/stage-04-script-skill agent-lab-posix uv run --locked pytest -q -rs
```

Recorded verification: Stage 01 **43 passed, 4 skipped**; Stage 02 **52 passed, 4 skipped**; Linux Stage 03 **48 passed**; Linux Stage 04 **73 passed**. The Windows skips concern symlink-creation privileges; separate Windows directory-junction tests passed. Automated agent tests use mock models; the selected evidence traces use real Gemini calls.

For Block 2, run from `stage-04-script-skill/workspace`:

```bash
python skills/csv-quality/scripts/check_csv.py --input data/workload.csv --max-hours 8
python skills/csv-quality/scripts/check_csv.py --input data/workload.csv --max-hours 9
python skills/csv-quality/scripts/check_csv.py --input data/workload-edge.csv --max-hours 0
```

The main fixture yields Lan 9 and Minh 3. Only Lan exceeds 8; nobody exceeds 9. Invalid rows and subsequent duplicate IDs are excluded with reasons. The edge fixture accepts only Minh 0 because an invalid first occurrence still reserves its ID. Missing or invalid thresholds and input failures return nonzero exit codes. The script never modifies its CSV input.

See the combined analysis and named evidence cases for the exact refund prompts, renamed-file checks, clarification cases and workload results.
