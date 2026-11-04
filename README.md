# Paper Tour

Paper Tour reads PDFs with you. It narrates the document, highlights passages, explains figures and equations, and answers questions with annotations and visual explanations beside the page.

Upload a research paper, lecture deck, exercise sheet or practical document. Read it aloud, create a guided tour with a teaching profile, or ask about a selected passage or figure. Documents, tours and conversations belong to the signed-in account.

## Run locally

Requires **Node.js 22.12+**, **Python 3.12**, and **Tesseract**. The local database, authentication and storage use Supabase.

```sh
npm ci
npm --prefix server ci
npm --prefix player ci
python3.12 -m venv ingest/.venv
ingest/.venv/bin/pip install -r ingest/requirements.txt
npm run supabase:start
npm run setup:local -- --skip-siri
npm run db:migrate
npm run dev
```

Open [localhost:5174](http://localhost:5174), create a local account, and add a Meta API key in **Profile & Settings**. Alternatively, set `MODEL_API_KEY` in the generated root `.env` and restart the app. Generating tours and answers uses API tokens and sends document text and rendered page/figure images to Meta.

The setup command creates local environment files and refuses to overwrite existing ones. `--skip-siri` skips the optional Apple voice installation. Browser speech is available without a separate speech API key. See [Development](docs/development.md) for prerequisites, configuration, optional speech providers and Docker deployment.

## What it does

- **Guided tours:** choose the structure, explanation depth and reader familiarity. Play completed leading chapters while generation continues; pause, resume or cancel generation through Activity.
- **Read aloud:** read source text with synchronized highlighting and explanations of figures, tables and equations.
- **Questions:** type or dictate a question, select a PDF region, and replay saved answers from document conversations.
- **Visual explanations:** annotate the source or show formulas, charts and mathematical constructions beside it. Interactive visuals can be revised through follow-up questions.
- **Speech:** browser voices, local Siri on supported Macs, and optional ElevenLabs or OpenAI speech providers.

## How it works

1. Python and PyMuPDF extract text, figures and equations with stable IDs and PDF geometry. Local OCR and OpenCV help locate components inside figures.
2. A persistent Vercel AI SDK agent reads the document and builds the tour through source-reading, visual-inspection and editing tools. The default model is `muse-spark-1.3`.
3. Validation checks source IDs, numerical evidence, expressions and rendered visuals. Required visual failures retain saved work for repair rather than publishing an incomplete tour as finished.
4. The React reader synchronizes PDF navigation, annotations and boards with narration. Questions use the same source and visual tools. SAM 3.1 can refine a selected region; voice questions use Muse transcription.

An older guided harness remains available through `PAPER_AGENT_ENABLED=false`. Saved tours and active jobs retain their original engine. See [Generation architecture](notes/GENERATION-ARCHITECTURE.md), [Durable workspace](notes/GENERATION-WORKSPACE.md) and [Tour format](docs/tour-format.md).

## Development

```sh
npm run build
npm test
```

The build checks server and player TypeScript and bundles the player. Tests cover the player, server, Python ingestion and repository scripts. Integration tests require a running local Supabase stack; live smoke tests also consume model tokens. See [test commands](docs/development.md#checks).

| Directory | Contents |
| --- | --- |
| `player/` | React PDF reader, library, settings, narration and visual playback |
| `server/` | Fastify API, generation worker, agent tools and provider integrations |
| `ingest/` | PDF extraction, figure measurement, document workflows and validation |
| `schema/`, `shared/` | JSON schemas, TypeScript contracts and shared validation |
| `supabase/` | Local service configuration and database migrations |
| `scripts/` | Setup, integration checks and evaluation utilities |
| `docs/` | Setup, data format, recorded evaluation and UI reference |
| `notes/` | Technical design notes and dated experiments |

## Documentation

- [Development and deployment](docs/development.md)
- [Document and tour format](docs/tour-format.md)
- [Technical notes](notes/README.md)
- [Recorded evaluation and its limits](docs/evaluation.md)
- [Contributors and project provenance](docs/provenance.md)

Generated explanations can be wrong. Source highlighting and mathematical checks help inspect an answer; they do not establish that every claim is correct. Explanations of contracts and letters describe the document and do not provide legal advice.
