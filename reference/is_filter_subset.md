# Check if current dimension filters are a subset of cached filters

Check if current dimension filters are a subset of cached filters

## Usage

``` r
is_filter_subset(current, cached)
```

## Arguments

- current:

  Named list of current dimension filters (normalized)

- cached:

  Named list of cached dimension filters (normalized)

## Value

TRUE if every value in current is available in cached
