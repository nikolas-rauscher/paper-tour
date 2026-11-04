# Recorded evaluation: 30 October 2026

These are historical guided-harness results, transcribed from the previous README. They do not measure the current default persistent agent or establish current latency, pricing or teaching quality. The original run artifacts are not included in this checkout.


Ten documents, one run each, on 2026-10-30: four papers (machine translation, a robot fish, quantum field theory,
ant ecology), one lecture deck, two exercise sheets (English probability, German linear algebra), a 63-page UK
tenancy agreement, a terms-of-service document and a German data-protection letter. All with `muse-spark-1.3`, the
`standard` preset and prompt version `paper-tour-harness@33`. This is ten documents, not a benchmark: there is no
baseline, no repeated run and no human rating. The PDFs and generated tours are not bundled with the repository. The `attention` row came from a separate run.

The numbers come from `node scripts/scorecard.mjs`, which reads the finished tours and makes no model calls:

```
tour           type      steps  words   min  chap  calls  tok k   usd   secs  resent  degr      ids  lines  back  front
-----------------------------------------------------------------------------------------------------------------------
attention      paper        11    526   3.5     4     15    217  0.34   71.2       1     0    37/37      -     0      0
eval-fish      paper        14    726   4.8     5     21    332  0.50  103.6       4     1    43/43      -     2      0
eval-casimir   paper        14    684   4.5     5     18    348  0.53   84.8       1     0    41/41      -     0      0
eval-ants      paper        14    674   4.4     5     19    348  0.51  145.8       2     1    42/42      -     0      0
eval-slides    lecture      12    923   6.0     4     16    212  0.38  115.7       3     2    60/60      -     1      0
eval-sheet-en  sheet        20    840   5.6     7     12    130  0.30   73.8       0     0    56/56   7/13     0      0
eval-sheet-de  sheet        22    768   5.1     7     12    177  0.38   87.9       0     0    55/55   9/15     0      0
eval-lease     contract     10    725   4.7     6     15    541  0.77  102.6       3     2    30/30      -     4      0
eval-terms     contract      9    648   4.2     6     13    221  0.33   90.5       2     1    28/28      -     2      0
eval-letter    letter        5    312   2.0     4      9     71  0.14   67.7       1     0    17/17      -     0      0
-----------------------------------------------------------------------------------------------------------------------
all 10                     131   6826  44.8    53    150   2597  4.18  943.6      17     7  409/409  16/28     9      0

tour           type      boards  kinds  doc  in text  illus  other  repeat  dropped  checkpoints
------------------------------------------------------------------------------------------------
attention      paper          4      4    1        1      2      1       0        0            1
eval-fish      paper          4      4    1        1      1      2       0        1            1
eval-casimir   paper          5      5    0        0      3      2       0        0            1
eval-ants      paper          4      4    1        1      2      1       0        1            1
eval-slides    lecture        3      3    1        1      2      0       0        1            1
eval-sheet-en  sheet         10      3    1        1      0      9       0        0            5
eval-sheet-de  sheet         12      3    0        0      0     12       0        0            5
eval-lease     contract       5      5    1        1      2      2       0        0            0
eval-terms     contract       5      4    2        0      0      3       0        0            0
eval-letter    letter         2      2    0        0      1      1       0        0            0
------------------------------------------------------------------------------------------------
all 10                       54     10    8        6     13     33       0        3           15


camera page of each step
attention      2 6 6 8 7 6 7 7 7 7 10
eval-fish      1 5 5 6 5 4 7 7 7 5 6 8 6 8
eval-casimir   1 2 3 9 9 9 10 10 10 9 9 10 11 11
eval-ants      2 5 4 6 6 6 8 8 8 9 10 11 15 16
eval-slides    3 7 11 13 15 19 20 13 21 25 28 31
eval-sheet-en  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 1
eval-sheet-de  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 5
eval-lease     10 26 36 7 23 17 41 13 17 5
eval-terms     2 5 7 12 19 17 16 15 1
eval-letter    1 1 1 1 1
```

What the numbers say, and what they do not:

- **Every document got a tour.** All ten completed, including the 63-page lease; writing one took 68 to 146 seconds
  with 9 to 21 model calls, about $0.14 to $0.77 at Muse Spark's list price
  ([pricing code](../server/src/muse/pricing.ts)).
- **Size.** Papers, the deck and the practical documents took 5 to 14 steps and 2 to 6 minutes of narration. A sheet
  takes an overview, a takeaway and a try-first checkpoint plus solution steps per exercise, so 20 and 22 steps.
- **Reading order.** A paper tour reads forward: after its first chapter no step goes back to the abstract or the
  introduction (`front`), and in 53 steps of the four papers the camera jumps back by more than a page twice (`back`).
  Before claims were anchored where the paper develops them (2026-10-29, `harness@32`), the same papers went back to
  the front 12 times and jumped back 9 times. Contracts are read by topic, not in page order.
- **Diversity.** All ten board kinds appear across the tours; each paper, deck or contract uses 3 to 5 different kinds,
  and no board repeats the data of an earlier one.
- **Document values.** Of 54 boards, 8 present numbers as the document's own and 6 more use values printed in the
  text; 13 are declared illustrative and 33 are equations, flows, functions, timelines or drawings.
- **Grounding.** All 409 IDs the tours point at exist in `doc.json`.
- **Checks at work.** The checks sent back 17 drafts, which were written again. Seven steps lost a part and kept their
  narration (`degr`): three boards (one failed its checks, two took too long to write) and four worked examples the
  plan had not asked for, which failed their checks. The activity checklist names each with its cause. No step and no
  tour was lost.
- **Exercise lines.** Of 28 solution lines, 16 checked as correct, none as wrong, and 12 could not be checked
  numerically and are marked unverified.

To score a new guided-harness run, use `server/run-tour.ts` as described in [Development](development.md#standalone-guided-harness), then run `node scripts/scorecard.mjs <tour-folder>` from the repository root. This does not reproduce the original measurements or evaluate the default persistent agent.
It looks up `doc.json` in `.data/documents/<folder name>`; `--docs <dir>` changes that, and `--json` prints the numbers as JSON.

## Related recorded experiments

The original October model notes recorded a SAM selection spike with 42 of 46 accepted snaps across 23 selected objects and a median reported latency of 1.2 seconds. They also recorded a small Muse Wonder versus Spark planning experiment that kept Spark as the default. These are historical observations from a small selection of documents, not a reproducible current benchmark; raw provider/run artifacts are not included here.

Current model availability comes from the connected account's catalog. Historical prices and model timing in this page should not be used as a live quote.
