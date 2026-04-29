# hunt-data

Pre-extracted enrichment data consumed by the **hunt** mobile app. One JSON
file per Minjastofnun fornleifaskráning project; the app fetches them
on-demand and caches each in Hive.

This directory is intended to live as its **own repository** served via
GitHub Pages. While it is sitting inside the parent `hunt/` repo for
development, treat it as a standalone artifact — nothing app-related
imports from it as code.

## Layout

```
verkefni/<num>.json    one file per Minjastofnun project
index.json             catalogue of all available projects
```

Each `verkefni/<num>.json` contains:

```json
{
  "verkefni": 2195,
  "source_url": "https://skraning.minjastofnun.is/Verkefni_2195.pdf",
  "title": "Fornleifar í Kalmansvík. Deiliskráning 2018. FS689-16262",
  "authors": "Adolf Friðriksson | Lilja Laufey Davíðsdóttir",
  "page_count": 18,
  "site_count": 13,
  "sites": [
    {
      "id": "BO-058:001",
      "lat": 64.32651,
      "lon": -22.07368,
      "description": "...",
      "heimild": "...",
      "hættumat": "..."
    }
  ]
}
```

The Flutter client looks up sites by **spatial proximity** (≤100 m to the
WFS marker) since `flakar/linur/punktar` records use `2195-NN` ids while
the PDFs use `BO-058:NNN` — the two id schemes don't share a key.

## Regeneration

From the parent `hunt/` repo:

```bash
poetry run python scripts/build_minja_index.py \
    --bbox=-22.10,64.30,-21.95,64.36 \
    --out-dir=hunt-data
```

Pass a country-wide bbox to crawl everything in one shot. PDFs are cached
under `hunt/cache/pdfs/` so repeated runs only re-parse, never re-download.

## Hosting via GitHub Pages

1. Create a new public repo, e.g. `hunt-data`, and push the contents of
   this directory (everything except this README is generated; commit
   only `verkefni/`, `index.json`, and the README).
2. In **Settings → Pages**, enable Pages from the `main` branch root.
3. The base URL becomes `https://<user>.github.io/hunt-data/`.
4. Update the Flutter app's `huntDataBaseUrlProvider` (in
   `lib/sources/minja_enrichment.dart`) to that URL.

GitHub Pages serves with `Access-Control-Allow-Origin: *` by default, so
no CORS configuration is needed.

## Attribution

Source PDFs are published by **Minjastofnun Íslands** and licensed for
re-use. Each project sidecar carries the original PDF URL in
`source_url`. The Flutter app surfaces these as `Heimild:` rows in the
teaser/almanac.
