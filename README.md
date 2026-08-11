# Strict Reader Source Catalog

This directory is the application repository's reviewed snapshot of the public source catalog.
It contains publisher metadata only. It must never contain user preferences, interaction data,
analytics identifiers, credentials, or private application configuration.

## Validate an update

1. Edit `v1/sources.json` without changing stable source IDs unnecessarily.
   Keep `editorialTier` (`reference|established|standard`) and `publisherType`
   (`public_service|general_news|specialist|institutional|wire|local`) explicit. Leave
   `featuredBylines` empty unless a separate editorial review approves canonical names.
2. Run `..\scripts\verify_source_catalog.ps1` from this directory, or
   `.\scripts\verify_source_catalog.ps1` from the repository root.
3. Inspect `app/build/reports/source-catalog-live/verification.json` and remove or disable any
   source with fewer than two `READY` representative articles out of three samples.
4. Update `v1/verification.json`, run `..\scripts\publish_source_catalog.ps1`, then run the
   normal offline test/build matrix.

## Publish

The public data-only repository is `parx01/strict-reader-catalog`. Its `main` branch should contain:

```text
README.md
v1/sources.json
v1/verification.json
```

Production reads the raw HTTPS URL for `v1/sources.json`. Publish catalog changes only after the
live verifier and normal tests pass. Use `scripts\publish_source_catalog.ps1 -CatalogRepository
<checkout> -Publish` to copy the reviewed artifacts, then review/commit/push that data-only
repository separately. Do not publish the private application repository.

## Roll back

Revert the bad catalog commit in `parx01/strict-reader-catalog` and push the revert. Keep the schema
version at `1` for content-only corrections. The app sends conditional requests and refreshes the
catalog cache on its normal 24-hour cadence; a later valid catalog disables managed sources omitted
from the manifest without deleting cached articles, bookmarks, history, or user-added feeds.
