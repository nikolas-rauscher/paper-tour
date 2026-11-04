# Architecture

Paper Tour separates document extraction, model work and playback. A PDF analysis is the shared source of truth: it contains the text, element IDs and geometry used by tours, reading and questions.

## Document ingestion

Uploads enter the private library through the Fastify API. Python extraction produces `doc.json`, figure crops and other assets. PyMuPDF supplies text and PDF geometry; local OCR and OpenCV help measure components inside figures. Sentences, blocks, sections, figures, equations and exercise parts receive source IDs.

The worker processes analysis jobs outside the browser request. Supabase holds private files and analysis records. Source-derived artifacts are tied to an immutable analysis, so an old tour is not silently rebound to a different extraction.

Entry points: [`service.ts`](../server/src/service.ts), [`worker.ts`](../server/src/worker.ts), [`ingest.py`](../ingest/ingest.py), [`process.py`](../ingest/process.py). The persisted contracts are described in [Tour format](tour-format.md).

## Models and provider integration

`server/src/ai/` adapts Meta to the Vercel AI SDK. `server/src/muse/` owns model discovery, pricing metadata, source context and provider-specific authoring schemas. The default reasoning model is Muse Spark 1.3; an account may select another supported model. Availability comes from the account's live model response, while price metadata is maintained separately.

The persistent agent uses the Meta Responses interface through the SDK's `ToolLoopAgent`. Structured questions use `Output.object`. SAM selection and Muse voice transcription have their own adapters. Speech synthesis is separate from the reasoning model.

The application sends text and source images to the configured provider. Responses history can also be stored by the provider. Account-scoped application storage does not imply provider requests remain on the user's device.

## Durable tour generation

New tours normally use `PaperAgent`, whose instructions and source context are designed for research papers. Its teaching profile captures structure, explanation depth, audience familiarity and optional custom instructions. Saved generations retain their selected engine and effective profile.

One queued job advances one model step. Tools return validated state transitions; the durable journal commits changes and results. A read/write classification permits compatible read operations while serializing state changes.

| Tool | Responsibility |
| --- | --- |
| `search_paper` | Find source passages |
| `read_elements` | Read identified elements in context |
| `view_visuals` | Inspect PDF visuals |
| `record_visual_observations` | Save observations and measured component context |
| `set_tour_plan` | Save the chapter plan |
| `upsert_tour_chapter` | Write or revise a chapter with source and visual checks |
| `finish_tour` | Validate and publish the complete tour |

Completed leading chapters are published as playable partials. Source references, speech anchors and required visuals are checked before chapter publication. Visual review is associated with the content it inspected; changing that content invalidates the review.

Postgres and pg-boss own job claims, checkpoints and recovery. A closed tab does not cancel the job. Explicit cancellation fences publication by stale workers. Budget limits pause work at a checkpoint; Resume grants another segment. An uncertain provider-dispatch outcome pauses for explicit recovery instead of blindly issuing the same call again.

Entry points: [`generation/runtime.ts`](../server/src/generation/runtime.ts), [`agent/runner.ts`](../server/src/agent/runner.ts), [`agent/store.ts`](../server/src/agent/store.ts), [`agent/tools/`](../server/src/agent/tools/), [`ai/response-agent.ts`](../server/src/ai/response-agent.ts), [`queue.ts`](../server/src/queue.ts).

### Guided engine

`PAPER_AGENT_ENABLED=false` selects the guided harness for new generations. It provides document-specific workflows for papers, lecture slides, exercise sheets, contracts and letters: plan the document, author explanations and visuals, then assemble a saved tour. Profile-based paper tours use a source-grounded teaching map with contribution, prerequisite, mechanism, evidence and limitation roles. Stable concept IDs connect prerequisites and citations; validation rejects missing sources, cycles and incomplete visual decisions.

The guided workspace retains content-addressed provider drafts, direction repairs and inspected visuals within a generation. A cached provider draft must still pass validation. An explicit retry can reuse the same generation's compatible work; a new generation starts a fresh workspace. Failed required visuals in profile jobs cannot be silently removed to publish a finished summary. Older profileless jobs retain their recorded degradation behavior.

Entry points: [`harness/generator.ts`](../server/src/harness/generator.ts), [`teaching-plan.ts`](../server/src/harness/teaching-plan.ts), [`ai/workspace.ts`](../server/src/ai/workspace.ts), [`harness/direction-repair.ts`](../server/src/harness/direction-repair.ts).

## Questions and conversations

Q&A is a separate structured-answer workflow. The service resolves the document analysis, selected model, question, optional region and conversation context. Deterministic retrieval ranks source passages. The author receives source text and images from the original PDF, writes a short answer tour, and passes it through grounding, narration and visual checks. Checked steps can stream to the reader before the whole answer is complete.

Questions share tour contracts and visual inspection with generation, and use their own structured-answer workflow. Conversations and visual revisions persist independently of saved tours. Returning to the tour preserves its paused playback position.

| Module | Responsibility |
| --- | --- |
| [`harness/ask.ts`](../server/src/harness/ask.ts) | Request orchestration, cancellation and repairs |
| [`ask-retrieval.ts`](../server/src/harness/ask-retrieval.ts) | Source selection |
| [`ask-authoring.ts`](../server/src/harness/ask-authoring.ts) | Provider instructions and wire schemas |
| [`ask-answer.ts`](../server/src/harness/ask-answer.ts) | Conversion and local validation |
| [`ask-assets.ts`](../server/src/harness/ask-assets.ts) | Original-PDF image context |
| [`conversations.ts`](../server/src/conversations.ts) | Conversation persistence |

Read-aloud object explanations reuse this answer path with reading-specific context and instructions. Prose preparation has a stricter word-preservation contract; see [Reading and audio](reading-and-audio.md).

## Visuals and playback

The model supplies source references and semantic visual data. Native boards use shared validators and a renderer registry. Custom `program` visuals execute in QuickJS and produce validated SVG; `plotly` visuals use restricted figure data and host-computed functions. Inspection and repair happen before publication. See [Visual explanations](visuals.md).

The React reader resolves source IDs to PDF geometry. Its engine coordinates camera movement, page annotations, board state and narration. Actions can be attached to word positions, keeping a highlight or mathematical step aligned with the spoken explanation. Source regions and custom scene geometry may contain coordinates; source IDs do not eliminate all geometry from the contract.

Entry points: [`Reader.tsx`](../player/src/Reader.tsx), [`engine.ts`](../player/src/engine.ts), [`camera.ts`](../player/src/camera.ts), [`scene.ts`](../player/src/scene.ts), [`narration/`](../player/src/narration/).

## Data boundaries

Supabase authenticates API tokens, while service ownership checks and database/storage policies scope documents and derived records to their owner. The storage bucket is private; the server issues time-limited URLs for authorized reads. Personal model and speech keys use Vault references and are not returned to browser roles.

Document deletion removes associated application records and assets according to their lifecycle. Provider-side retention is governed by the chosen provider and is distinct from application deletion. Contributor-model use requires explicit consent.
