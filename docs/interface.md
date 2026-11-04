# Interface

The library, settings and reader share controls in [`player/src/ui/`](../player/src/ui/). React, Base UI, Animate UI and Motion provide focus handling, interaction state and transitions. Application colors come from the semantic theme tokens in `player/src/themes.css`.

## Library and reader

Document preparation and tour generation have separate states. An interrupted newer generation does not hide an existing saved tour. Model selection saves the account preference explicitly. Search, sorting and filters remain useful when some documents need attention.

Desktop reader navigation can collapse beside the PDF; mobile navigation uses a sheet. The question composer remains anchored while its contents expand. Region selection supplies visual cursor feedback. Status details open on request rather than occupying the reading surface throughout playback.

## Motion, sound and feedback

Shared motion tokens live in `ui/motion.ts`. Controls use real async state for saving and failure feedback. Upload confirmations use toast lifecycle handling with hover/focus suspension. Native keyboard and focus behavior remain part of the component contract.

Optional interface sounds default off and unlock after interaction. They stop when the page is hidden and remain silent during narration. Sound preferences are browser-local. The companion's fluid-ink renderer follows activity and narration callbacks; it does not measure microphone or playback amplitude. Its reduced-motion and rendering fallbacks are described in the [renderer notes](../player/src/activity/orb/README.md).

## Assets and attribution

The [README demo gallery](../README.md#demo-a-paper-explained-on-the-page) shows source highlighting, table spotlights, figure annotations, reported results and illustrative constructions from a guided paper tour. Gallery thumbnails link to the original screenshots. Constructions labelled illustrative are explanatory examples rather than measured paper results.

Adapted UI components retain their notices under [`ui/vendor/`](../player/src/ui/vendor/). See [Credits](credits.md) for attribution. Component registry configuration in `player/components.json` is optional development tooling, not a runtime service dependency.
