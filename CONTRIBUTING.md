# Contributing

Start with [Development](docs/development.md) for local setup and [Architecture](docs/architecture.md) for the source map.

Use a focused branch and describe the user-visible behavior or concrete problem in commits and pull requests. Run `npm run build` and `npm test` for application changes. Database integration and live-provider smoke checks are separate; live checks can incur API charges.

Keep persisted contract changes compatible with saved documents and tours, or provide an explicit migration. Keep source IDs tied to their original analysis. Shared validation belongs in `shared/`; provider wire-format adaptations should not weaken the canonical contract.

New visual kinds need mathematical/geometry checks and playback coverage. The extension points are described in [Visual explanations](docs/visuals.md#adding-a-native-visual-kind). Preserve pause, seek, cancellation and account-ownership behavior when changing a workflow.

Keep generated runs, PDFs, credentials, local environments and personal tooling under ignored local paths. Public documentation should describe current behavior; dated measurements belong in [Evaluation](docs/evaluation.md) with their limitations.
