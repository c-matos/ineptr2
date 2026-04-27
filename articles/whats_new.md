# What changed from ineptR

ineptr2 is a ground-up rewrite of
[ineptR](https://c-matos.github.io/ineptR/). This article summarises
what changed and why.

## INE API V2

ineptr2 targets [**version 2** of the INE
API](https://www.ine.pt/xportal/xmain?xpid=INE&xpgid=ine_api_db&INST=719281968&lang=en),
which introduced richer dimension filtering while remaining
backward-compatible with V1 queries. The key additions:

- **`Dim1=T` (all periods)** — setting dimension 1 value to `T`
  retrieves all available reference periods, instead of having to
  perform one call for each period explicitly.
- **Comma-separated values** — multiple values in a single dimension
  parameter (e.g.`Dim1=S7A2019,S7A2020`), enabling multi-value queries
  without multiple API calls.
- **Level-based selection (`lvl@N`)** — retrieve all entries at a given
  hierarchical level (e.g. `lvl@5` on the geographic dimension returns
  all municipalities).
- **Classification subtree (`<*>CODE`)** — retrieve all children under a
  classification node (e.g. `<*>200` on the geographic dimension returns
  a NUTS III - Madeira region and all its municipalities).
- **Up to 1 million rows** - limit greatly increased from 40 000 to 1
  000 000 returned rows per call.

These features let ineptr2 build smarter, more compact URL grids — which
directly reduces the number of HTTP requests needed for filtered
queries. For example, the full `0008206` indicator now requires 66
calls, compared to over 2000 calls with V1.

## R6 client instead of standalone functions

ineptR exported five standalone functions (`get_ine_data()`,
`get_metadata()`, `get_dim_info()`, `get_dim_values()`,
`is_indicator_valid()`). Every call was stateless — you had to pass
`lang` each time. Caching was not implemented:

``` r
# ineptR — repeat lang on every call
df <- get_ine_data("0008273", lang = "EN")
meta <- get_metadata("0008273", lang = "EN")
dims <- get_dim_values("0008273", lang = "EN")
```

ineptr2 wraps everything in a single R6 object. Configure once, call
methods:

``` r
# ineptr2 — configure once, use everywhere
ine <- INEine$new(lang = "EN")
df <- ine$get_data("0008273")
meta <- ine$get_metadata("0008273")
dims <- ine$get_dim_values("0008273")
```

The are now more concise (now: get_data; before: get_ine_data) and
coherent (now: get_data, get_metadata; before: get_ine_data,
get_metadata).

Settings are live — change `ine$lang` mid-session and every subsequent
call picks it up, with validation built in (e.g. `lang` only accepts
`"PT"` or `"EN"`, `row_limit` is capped at the API maximum).

## Smaller footprint

ineptR pulled in **11 packages** at install time:

> dplyr, httr, httr2, lifecycle, magrittr, progressr, purrr, readr,
> stringr, tibble, tidyr

ineptr2 needs only **4 packages**:

> httr2, jsonlite, R6, xml2

All data wrangling now uses base R and the entire tidyverse chain is
gone. The package installs faster and has a smaller footprint.

## Smarter chunking

The INE API limits each response to a maximum number of rows. ineptR
implemented a maximum of 40 000 rows per request (`max_cells`), which
meant that large indicators required hundreds or thousands of small API
calls.

ineptr2 defaults to a much higher limit (`row_limit = 1 000 000`) and
splitting logic is improved (Dimension 1 now accepts multiple values per
call). The result is far fewer HTTP requests for the same data. You can
preview the chunk plan before committing to a download:

``` r
ine$preview_chunks("0008206")
```

## File-based caching

ineptR had no caching — every call hit the API fresh, and all data lived
in memory. When downloading large indicators, the user was responsible
for the cycle *get some data - save to disk - get more data*.

ineptr2 introduces a chunk-based file cache with three layers:

- **Chunk cache** — raw API responses stored as individual JSON files on
  disk
- **Data cache** — processed data stored as RDS, with dimension filters
  tracked so the cache is only reused when the new request is a subset
  of what was cached
- **Metadata cache** — used by `is_updated()` to detect changes without
  a full download

Caching is enabled via `ine$use_cache <- TRUE` or at construction time,
and the cache directory is configurable. See the [How caching
works](https://c-matos.github.io/ineptr2/articles/caching.html) article
for a detailed walkthrough.

## Download and load as separate steps

ineptR always downloaded and processed data in one go via
`get_ine_data()`. For large indicators this meant holding everything in
memory at once and risking out-of-emory errors.

ineptr2 separates the two concerns:

| Step     | Method                | What it does                                              |
|----------|-----------------------|-----------------------------------------------------------|
| Download | `ine$download_data()` | Streams chunks to disk, nothing held in memory            |
| Load     | `ine$load_raw_data()` | Reads cached *raw* chunks back into R.                    |
| Both     | `ine$get_data()`      | Convenience wrapper that downloads, loads and cleans data |

This means you can download overnight, close R, and load the results
later.

## Resume support

If a download is interrupted (network drop, session crash, timeout),
ineptr2 picks up where it left off. Each download creates a JSON
manifest tracking which chunks succeeded. Calling `download_data()`
again skips completed chunks automatically.

## Full indicator catalog

ineptr2 exposes methods to download (`ine$download_catalog()`) and load
(`ine$get_catalog()`) the full indicator catalog with over 13 000
indicators.

## New methods

ineptr2 adds several methods that had no equivalent in ineptR:

- **`ine$info()`** — prints a human-readable summary of an indicator:
  name, periodicity, time range, dimensions and their sizes.
- **`ine$is_updated()`** — compares cached metadata against the live API
  to check if data has been updated since your last download.
- **`ine$preview_chunks()`** — shows how many API calls a download will
  require, before you commit to it.
- **`ine$get_catalog()` / `ine$download_catalog()`** — retrieve the full
  INE indicator catalog.
- **`ine$list_cached()`** — lists all indicators currently in the file
  cache.
- **`ine$clear_cache()`** — clears cached files for one or all
  indicators.

## Summary

|                                    | ineptR                 | ineptr2                                              |
|------------------------------------|------------------------|------------------------------------------------------|
| Interface                          | 5 standalone functions | R6 client object                                     |
| Dependencies                       | 11                     | 4                                                    |
| Rows per API request               | 40 000 (max)           | 1 000 000 (max)                                      |
| File-based caching                 | No                     | Chunk, data, and metadata layers                     |
| Download without loading to memory | No                     | `download_data()` + `load_raw_data()` / `get_data()` |
| Resume interrupted downloads       | No                     | Automatic via manifest                               |
| Preview chunk plan                 | No                     | `preview_chunks()`                                   |
| Indicator catalog                  | No                     | `get_catalog()` / `download_catalog()`               |
