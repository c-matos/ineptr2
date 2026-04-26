# Gracefully handle HTTP request failures

Validates connectivity, performs the request, and downgrades HTTP errors
to messages instead of stopping.

## Usage

``` r
gracefully_fail(request, path = NULL)
```

## Arguments

- request:

  An httr2 request object.

- path:

  Optional file path to save the response body to disk.

## Value

An httr2 response object, or `invisible(NULL)` on failure.
