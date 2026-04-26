# Filter cached data frame to match current dimension filters

Filter cached data frame to match current dimension filters

## Usage

``` r
filter_cached_data(data, current_filters, dim_values)
```

## Arguments

- data:

  Cached data.frame

- current_filters:

  Normalized dimension filters from current request

- dim_values:

  Dimension values tibble from metadata (output of extract_dim_values)

## Value

Filtered data.frame
