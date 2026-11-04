# Document and tour format

Tours reference source elements by ID; the player resolves their PDF coordinates from `doc.json`. To point
inside a figure without a panel ID, a tour gives a `region` normalised to the target's bbox (`[0, 0, 0.5, 1]` is the
left half).

Coordinates in `doc.json` are PDF points with the origin at the top-left of the page and y growing downward (PyMuPDF's
convention). A bbox is `[x0, y0, x1, y1]`.

| ID | Example | Points at |
|---|---|---|
| block | `p3.b2` | Text block 2 on page 3 (reading order) |
| sentence | `p3.s12` | Sentence 12 on page 3; `rects` has one rect per line fragment |
| section | `sec-3.2`, `sec-abstract` | The section's heading block |
| figure / table | `fig1`, `tab2` | The graphic or table body, by printed number, without the caption |
| panel | `fig2.a` | A sub-figure, in reading order |
| equation | `eq1`, `equ2` | A numbered or unnumbered display equation |
| page | `page4` | A whole page |

Exercise sheets also have task and part IDs in `doc.json` (`ex1`, `ex1.a`) that the planner uses. Steps point at the
blocks and sentences they cover.

**A tour** has a `title`, the `paper` folder it belongs to, the `doc_sha256` of the `doc.json` it was written against,
an optional `level`, `voices`, the `preset` it was written for, `suggested_questions` and `left_out` (what the writer
planned but could not put in, each with its cause, which the reader says on its cards), and `chapters` of steps.
`doc_type` is `research_paper`, `exercise_sheet`, `lecture`, `contract`, `letter` or `other`; the reader uses it to pick
its mode, so a lecture plays slide by slide.

**A step** has `say` (the narration), `actions`, an optional `hold_ms` pause, optional pre-rendered `audio` and an
optional `checkpoint`: `{ prompt, hints }` with up to three hints, from a nudge to almost the answer. A step with a
checkpoint pauses after its narration so the learner can try first, and the next step reveals the answer.

Actions fire at the start of the step, or at word `at_word` of the narration. Overlays disappear when the step ends
unless `persist` is set; `clear` removes persistent ones.

| Action | Effect |
|---|---|
| `focus` | Scroll the target into view at reading zoom |
| `zoom` | Fit the target (or a `region` of it) to the viewport |
| `highlight` | Marker or underline on the text lines of each target |
| `box` | Outline with an optional label |
| `spotlight` | Dim everything except the target |
| `note` | Callout card next to the anchor |
| `arrow`, `circle` | Hand-drawn pen mark pointing at, or around, the target or a region of it |
| `clear` | Remove persistent overlays |
| `board` | Show a board item beside the page; it stays across steps until another item or `board_clear` |
| `board_focus` | Light up one part of the board item by index |
| `board_clear` | Remove the board item |

**Board kinds.** The model fills a template with semantic data; code checks it, lays it out and animates it.

| Kind | Shows | `board_focus` part |
|---|---|---|
| `formula` | An equation built term by term, each term a coloured substring of the LaTeX | term |
| `flow` | Boxes and arrows of a pipeline or dependency, revealed in order | node |
| `bars` | Numbers the document reports, as animated bars | bar |
| `plot` | Function curves from a safe expression, with an optional parameter sweep, log axes, marks computed by code (roots, extrema, intersections) and data points from the document | curve |
| `scene` | Positioned vectors, matrices, text, shapes and plots, changing state frame by frame for a worked example | object |
| `heatmap` | A grid with labelled rows and columns: weights, masks, correlations, ablations. Can show a row-wise softmax, a causal mask or curves between tokens. Its `source` is `document`, `illustrative` or `formula` (the server computes the values) | row |
| `scatter` | Points on two measured axes, with log axes and a shaded "better" corner, for trade-offs | point |
| `shapes` | Matrix dimensions through an operation; the code checks that inner dimensions match | matrix |
| `timeline` | Dated or ordered events, spans and deadlines on one axis; the code resolves relative dates such as "14 days after the notice" and checks span lengths | event, span |
| `distribution` | One or two distributions from parameters or document counts, morphing from one to the other; the probability of a shaded region is computed by code | dist, region |

Narration uses the selected speech provider or pre-rendered step `audio` with optional per-word timings. See [Reading and audio](reading-and-audio.md).

Agent-authored `program` and `plotly` visuals extend the native board catalog. See [Visual explanations](visuals.md) for their execution and validation contracts. The schemas in [`schema/`](../schema/) and shared TypeScript types in [`shared/`](../shared/) define the accepted formats.
