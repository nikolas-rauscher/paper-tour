# Visual explanations

Paper Tour can annotate a source figure, render a native mathematical board, or display an authored interactive visual. All three use the same source analysis and narration timeline.

## Representation choices

| Representation | Useful for | Contract |
| --- | --- | --- |
| PDF annotations | A passage, equation, figure component or table already present in the document | Source element IDs, optional normalized regions, labels and timed actions |
| Native boards | Formulas, flows, bars, plots, scenes, heatmaps, scatter, matrix shapes, timelines and distributions | Typed semantic data with shared validators |
| `program` | A custom mathematical construction that changes over narration time or parameters | JavaScript producing SVG from bounded host inputs |
| `plotly` | Interactive quantitative plots | Restricted figure JSON and host-evaluated function expressions |

Visual selection is driven by the explanation. Source annotation is often sufficient; a new construction should clarify a relationship, transformation or comparison. Numerical examples declared illustrative remain distinct from measurements taken from the document.

## Inspection and repair

Native validators check source IDs, data provenance, dimensions, expressions and other kind-specific constraints. Examples and exercise solution lines use mathematical checks where supported; an uncheckable line is not proof of correctness.

Authored programs run at sampled narration times and render to PNG frames for model review alongside source and narration context. Concrete defects feed a targeted repair loop. Plotly functions receive numerical/contract checks, including an independent Python computation. These checks improve inspectability; model review does not guarantee that every explanation is correct.

Required visuals must pass the applicable gates before publication. A persistent failure keeps saved work for correction instead of substituting a finished text summary in profile-based jobs.

Entry points: [`inspect.ts`](../server/src/visuals/inspect.ts), [`model-author.ts`](../server/src/visuals/model-author.ts), [`program-review.ts`](../server/src/visuals/program-review.ts), [`plotly-compute.ts`](../server/src/visuals/plotly-compute.ts).

## Execution boundaries

QuickJS receives strings and numerical inputs rather than application objects or host callbacks. Memory, stack and interruption limits bound execution. The browser uses a terminable worker; validated output is rendered as an inert SVG image. SVG validation rejects external resources, scripts, event handlers and unsupported structure. Server rasterization runs in a bounded subprocess without application credentials.

Plotly uses a restricted data contract. The host samples mathematical functions; generated figures do not need hundreds of model-written coordinates. Browser controls support local exploration without cloud-sharing actions.

Narration controls scene time, so pausing and seeking also pause and seek the construction. A learner's slider changes can accompany the next question. The server accepts bounded numerical overrides against its saved visual rather than accepting replacement code or source data from the browser.

Entry points: [`program-runtime.ts`](../server/src/visuals/program-runtime.ts), [`program-raster.ts`](../server/src/visuals/program-raster.ts), [`program.worker.ts`](../player/src/visuals/program.worker.ts), [`shared/program.ts`](../shared/program.ts), [`shared/plotly.ts`](../shared/plotly.ts).

## Persistence

Visual artifacts belong to an owner, document and analysis. A follow-up question can revise a construction while keeping earlier revisions available. Tour publication and artifact references commit together; a published tour retains its original content. Revision locks prevent one edit from silently overwriting another.

Entry points: [`artifacts.ts`](../server/src/visuals/artifacts.ts), [`persistence.ts`](../server/src/visuals/persistence.ts), [`visualArtifacts.ts`](../shared/visualArtifacts.ts).

## Adding a native visual kind

1. Define its canonical data and validator in `shared/`, updating the persisted tour schema if needed.
2. Register the authoring schema, validator and focus identities in [`harness/board.ts`](../server/src/harness/board.ts).
3. Add the explanation guidance and any explicit Q&A wire-format adapter.
4. Implement rendering and interpolation in the player, preserving narration, pause and seek semantics.
5. Cover the actual mathematical/geometry contract and generation/Q&A conversion in the relevant tests.

Registration makes a kind available; it does not require the model to choose it. See [Tour format](tour-format.md) for native action and board types.
