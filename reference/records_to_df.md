# Convert a list of named lists to a data.frame

Handles NULL values by replacing them with NA, unlike `as.data.frame`
which silently drops NULL elements.

## Usage

``` r
records_to_df(records)
```

## Arguments

- records:

  A list of named lists with consistent field names.

## Value

A data.frame.
