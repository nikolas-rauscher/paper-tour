# Reading and audio

Guided tours and Read aloud share playback infrastructure but have different content contracts. A tour teaches a selected explanation; Read aloud preserves the source prose and explains the visual objects encountered along the way.

## Preparing a reading

Source prose is grouped into natural units, including consecutive headings with their following paragraph. Cleanup may repair layout whitespace, soft hyphens and recognized citation markers, but original words, numbers, operators and negations must survive in order. A changed draft receives a correction request; unresolved failures pause reading.

Author metadata, repeated page furniture and recognized figure labels are excluded from spoken prose. Bibliography and citation handling are explicit settings. Captions associated with a figure or table accompany its explanation rather than becoming a second literal reading. Unmatched captions remain source text.

Figures, tables, equations and recognized procedures use the structured explanation workflow with their original PDF region and nearby context. The model explains the object in context rather than reciting OCR fragments. This recognition is not a guarantee that every possible mathematical or procedural passage is detected correctly.

Prepared speech and visual actions are saved under an owner/document/analysis/source/model/language/version identity. Failed or cancelled preparation is not stored as valid content. Cache versions distinguish prose and object explanations so a visual correction need not invalidate unchanged prose.

Entry points: [`reading-speech.ts`](../server/src/reading-speech.ts), [`reading-cache.ts`](../server/src/reading-cache.ts), [`shared/reading.ts`](../shared/reading.ts), [`readingOrder.ts`](../shared/readingOrder.ts), [`readingPreparation.ts`](../player/src/readingPreparation.ts).

## Keeping playback moving

Entering Read aloud starts sequential background preparation from the restored position. Foreground playback joins matching work and takes priority after navigation. Seeking discards obsolete preparation; leaving the mode or opening an answer stops it. Authentication/provider outages suspend the queue rather than issuing a failing request for every remaining paragraph. A failed passage is not silently skipped.

This text/visual preparation is separate from audio lookahead. API speech prepares the current bounded sentence group and one following group. During the penultimate chunk it can prepare the first clip of the next already-authored passage. Short sentences may share a group; chunking preserves words and their global positions.

Pending requests are shared and completed audio is cached within account/version and size/lifetime bounds. Already-requested audio may finish into cache after playback pauses. Selecting a voice does not itself synthesize the whole paper.

## Speech providers

| Provider | Setup | Timing |
| --- | --- | --- |
| Browser voices | No speech API key; available voices depend on the browser | Native boundaries where available, otherwise estimated |
| Siri | Supported local Apple silicon Mac with the optional CLI and installed voices | Depends on the local voice/engine |
| ElevenLabs | Explicitly selected cloud connection | Word timestamps when returned alignment matches the exact input |
| OpenAI | Explicitly selected cloud connection | Estimated word timing |

Provider/voice changes pause playback and invalidate old audio versions. A failed provider request pauses with a visible error rather than automatically switching to another provider. Optional keys are server-only or stored as personal Vault references; deleting a personal key leaves that connection unconfigured.

Run `npm run setup:siri` to prepare local Siri. `SIRI_TTS_AUTO_SETUP=false` disables automatic installation at API startup. See [Development](development.md#speech) for setup and environment variables.

Audio time drives source and visual word callbacks, including playback-speed changes. Estimated timing is less precise than provider alignment; pronunciation and OCR quality also depend on the input and selected voice.

Entry points: [`speech/service.ts`](../server/src/speech/service.ts), [`speech/providers.ts`](../server/src/speech/providers.ts), [`narration/`](../player/src/narration/), [`shared/speech.ts`](../shared/speech.ts).

## Spoken questions

Hold the microphone button, speak and release. The browser produces mono 16 kHz WAV audio and Muse Voice Transcribe returns editable text. Document keywords and language can bias transcription. The learner sends the question with Ask; recording or transcription alone does not submit the question.

The server authenticates the request, checks document access and sends transcription to Meta. Source terms are hints, not a guarantee against transcription errors. Entry points: [`ask/voice.ts`](../player/src/ask/voice.ts), [`muse/transcribe.ts`](../server/src/muse/transcribe.ts), [`ai/transcription.ts`](../server/src/ai/transcription.ts).
