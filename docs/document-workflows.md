# Local PDF workflows

These optional Python tools prepare PDFs without model calls. Keep outputs under `.data/`. Output directories must be new; the tools do not overwrite the original input documents.

## Combine a course pack

```sh
ingest/.venv/bin/python ingest/workflows.py pack .data/course slides.pdf script.pdf exam.pdf
```

The output `paper.pdf` preserves input order and adds source bookmarks. `sources.json` records file hashes and page ranges. Upload the combined PDF to ask questions across the included sources. The tool does not automatically connect exam questions to theorems across separate library documents. Inputs are limited to 12 files and 100 pages in total; encrypted inputs are rejected.

## Compare two versions

```sh
ingest/.venv/bin/python ingest/workflows.py compare .data/revision old.pdf new.pdf
```

`changes.html` shows inserted and removed text blocks with source-page references. `changes.json` includes the changes and both source hashes. `paper.pdf` contains the unchanged originals, old then new, for upload to the reader.

This is a literal text comparison. Layout, image and OCR differences can be missed; it is not a contract assessment or an automatic calculation of financial consequences.

## Make a photographed page searchable

```sh
ingest/.venv/bin/python ingest/workflows.py photo .data/letter photo.jpg --language deu --tessdata .data/tessdata
```

Place the required Tesseract language data in the specified directory. Local OCR preserves the image and adds a searchable text layer. Multiple languages can be combined, for example `deu+eng`; `--rotate 90` rotates before OCR. Input limits are 20 megapixels and 50 MiB. Perspective correction is not implemented.

This preprocessing does not enable an automatic OCR fallback for every library upload. Check critical numbers and dates against the image. Uploading the resulting PDF for model explanation can still send its content to the configured provider.

## Inspect an analysis

```sh
ingest/.venv/bin/python ingest/ingest.py document.pdf --out .data/document
ingest/.venv/bin/python ingest/outline.py .data/document
ingest/.venv/bin/python ingest/validate.py .data/document
```

`outline.py` prints source elements and IDs. `validate.py` checks document/tour files and references when present; see its command-line help for available options. `check_tour.py` renders saved tour targets for inspection.
