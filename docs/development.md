# Development

Run commands from the repository root unless a command explicitly changes directory.

## Prerequisites

- Node.js 22.12 or later and npm.
- Python 3.12 and the dependencies in `ingest/requirements.txt`.
- Tesseract for local OCR. On macOS, install it with `brew install tesseract`; the Docker image installs it through apt. Additional OCR languages require their language data.
- Supabase for Postgres, authentication and storage. `npm run supabase:start` uses the bundled CLI with its experimental stack enabled. If your CLI/runtime uses containers, start Docker Desktop or Colima first.

## Local library setup

Follow the installation commands in the [README](../README.md#run-locally). `npm run setup:local -- --skip-siri` reads the running local Supabase configuration and creates the root `.env` and `player/.env.local`. It refuses to overwrite either file. Keep existing configuration when restarting an established checkout.

Local email sign-up is enabled without email confirmation in `supabase/config.toml`. Open [localhost:5174](http://localhost:5174) and create a local account. Model credentials can be saved in Profile & Settings or supplied as `MODEL_API_KEY` in the root `.env`.

`npm run dev` starts three processes: the API on port 3001, the worker, and Vite on port 5174. The worker does not watch source changes; restart it after changing server code. Vite proxies `/api` to the local API. An HTTP health check is available at [localhost:3001/health](http://localhost:3001/health).

On Colima, if Supabase cannot mount the default temporary directory:

```sh
mkdir -p .data/tmp
TMPDIR="$PWD/.data/tmp" npm run supabase:start
```

Manage the stack with `npm run supabase:status` and `npm run supabase:stop`. Apply migrations with `npm run db:migrate`; do not reset an existing database to pick up changes.

## Configuration

[`.env.example`](../.env.example) lists supported environment variables. Keep server credentials out of `VITE_` variables: those are public browser configuration.

| Setting | Purpose |
| --- | --- |
| `MODEL_API_KEY` | Optional shared Meta key; accounts can supply their own key instead |
| `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` | Server database and Supabase access |
| `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` | Public browser Supabase configuration |
| `APP_ORIGINS` | Allowed browser origins for the API |
| `PAPER_AGENT_ENABLED` | Defaults to true; false selects the older guided harness for new jobs |
| `PYTHON_BIN` | Optional Python executable override; defaults to `ingest/.venv/bin/python` |
| `ELEVENLABS_API_KEY`, `OPENAI_TTS_API_KEY` | Optional server speech-provider keys |

Generation uses document text and rendered images in provider requests. Live generation and API speech can incur charges. The normal build and unit tests do not require paid provider requests.

### Speech

Browser voices require no speech-provider setup. On supported Apple silicon Macs, `npm run setup:siri` installs the local Siri CLI and downloads missing voices. Setup and the first API start may trigger this automatically unless disabled; set `SIRI_TTS_AUTO_SETUP=false` in the root `.env` to disable automatic setup. `--skip-siri` skips only the setup helper's installation step.

ElevenLabs and OpenAI are optional, explicitly selected providers. See [Reading and audio](reading-and-audio.md) for credentials, alignment, caching and limitations. Available voices depend on the selected provider and local installation.

### Diagnostics

`AI_SDK_DEVTOOLS=true` enables development timing and token diagnostics. With server development dependencies installed, run `npm --prefix server exec -- devtools`. The integration omits prompts, generated text, raw requests/replies and image bytes, and is disabled in production.

## Checks

```sh
npm run build
npm test
npm run test:integration
```

The [CI workflow](../.github/workflows/ci.yml) installs Node, Python and Tesseract, then runs the build and unit suites without provider keys. Database-backed tests are skipped without `TEST_DATABASE_URL`.

`npm test` expects the Python environment at `ingest/.venv`. The integration suite needs local Supabase and its configured test environment. Inspect `scripts/test-integration.mjs` for its service setup.

The following is a separate live-provider smoke test. It uses the configured local test account, uploads a synthetic diagram PDF, creates a tour and consumes API tokens:

```sh
npm run test:smoke:muse
```

Use `?silent=1` for timed narration without speech, or `?rate=4` for faster playback checks. Press `?` in the reader for keyboard shortcuts; `D` displays source element IDs.

## Standalone guided harness

`server/run-tour.ts` runs the older guided harness without the library or Supabase. It requires a configured Meta key and makes real provider calls. It does not run the default persistent agent.

```sh
mkdir -p .data/documents/mydoc
cp path/to/document.pdf .data/documents/mydoc/paper.pdf
ingest/.venv/bin/python ingest/ingest.py .data/documents/mydoc/paper.pdf
cd server
node --import tsx run-tour.ts standard ../.data/runs/mydoc ../.data/documents/mydoc mydoc
```

From the repository root, inspect the resulting run with:

```sh
node scripts/scorecard.mjs .data/runs/mydoc
```

The script reads saved artifacts and makes no provider calls. Its counts describe the saved run; source references and output completeness still need inspection. Other ingestion utilities include `validate.py`, `outline.py`, `check_tour.py` and the [local PDF workflows](document-workflows.md).

## Docker deployment

[`Dockerfile`](../Dockerfile) builds the player and installs the server/Python runtime. [`compose.yaml`](../compose.yaml) runs the API and worker; it expects an existing Supabase database, auth and storage service.

Configure `.env` from `.env.example`, including the deployed app's origin and public Supabase settings. Set `VITE_API_URL` when the API has a separate origin. Public Vite values are build arguments, so changing them requires rebuilding.

```sh
docker compose up --build
```

Compose serves the application on port 3001. For Supabase running on the host, set `DOCKER_DATABASE_URL` and `DOCKER_SUPABASE_URL` to addresses reachable from containers; loopback addresses inside a container refer to that container. Apply migrations against the intended database before starting the application. The Compose file does not provision the Supabase services themselves.
