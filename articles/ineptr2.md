# Getting started

This article walks through a typical ineptr2 workflow: creating a
client, exploring an indicator, and fetching data.

## Install and load

``` r

# From GitHub
install.packages("ineptr2")
```

``` r

library(ineptr2)
```

## Create a client

The `INEClient` object holds your configuration — language, caching
preferences, timeouts — so you don’t have to repeat them on every call.

``` r

ine <- INEClient$new(lang = "EN")
ine
#> <INEClient>
#>   Language:          EN
#>   Cache:             disabled
#>   Row limit:         1,000,000
#>   Max retries:       3
#>   Progress interval: 10
#>   Timeout (s):       300
```

You can change settings at any time:

``` r

ine$lang <- "PT"
ine$lang <- "EN"
```

## Inspect an indicator

Suppose you want to work with indicator `"0008273"` (Resident
population). Start by checking what it contains:

``` r

ine$info("0008273")
#> <INE Indicator>
#>   Code: 0008273
#>   Name: Resident population (No.) by Place of residence (NUTS - 2013), Sex and Age group; Annual - Statistics Portugal, Annual estimates of resident population
#>   Periodicity: Annual (2011 - 2023)
#>   Last updated: 2024-06-18
#>   Dimensions: 4
#>     dim1: Data reference period (13 values)
#>     dim2: Place of residence (NUTS - 2013) (344 values)
#>     dim3: Sex (3 values)
#>     dim4: Age group (19 values)
```

This shows the indicator name, periodicity, time range, last update
date, and a breakdown of dimensions with their sizes.

For more detail on each dimension, use `get_dim_info()`:

``` r

ine$get_dim_info("0008273")
#>   dim_num                             abrv versao
#> 1       1            Data reference period  XXXXX
#> 2       2 Place of residence (NUTS - 2013)  03505
#> 3       3                              Sex  00305
#> 4       4                        Age group  00708
#>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        nota_dsg
#> 1 2011-2020, Final Resident Population Estimates - revised intercensal estimates (general regular revision) taking as reference the results of the 2011 and 2021 Censuses. From 2021 onwards, Provisional Resident Population Estimates - postcensal estimates based on the results of the 2021 Census .<BR>2022, Provisional Resident Population Estimates revised in June 2024, which include in the resident population displaced persons from Ukraine who are beneficiaries of the Temporary Protection regime in Portugal.
#> 2                                                                                                                                                                                                                                         Resident Population Estimates according to the administrative territorial division corresponding to the version of the Official Administrative Map of Portugal in force at the date of the 2021 Census (CAOP 2020) and the version of NUTS (NUTS 2013) in force from January 1, 2015.
#> 3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          <NA>
#> 4                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          <NA>
```

To see the actual values available in each dimension:

``` r

# All dimensions
vals <- ine$get_dim_values("0008273")
head(vals)
#>                  dim_num  cat_id categ_cod categ_dsg categ_ord categ_nivel
#> Dim_Num1_S7A2011       1 S7A2011   S7A2011      2011  20110101           1
#> Dim_Num1_S7A2012       1 S7A2012   S7A2012      2012  20120101           1
#> Dim_Num1_S7A2013       1 S7A2013   S7A2013      2013  20130101           1
#> Dim_Num1_S7A2014       1 S7A2014   S7A2014      2014  20140101           1
#> Dim_Num1_S7A2015       1 S7A2015   S7A2015      2015  20150101           1
#> Dim_Num1_S7A2016       1 S7A2016   S7A2016      2016  20160101           1

# Only dimensions 1 and 3
ine$get_dim_values("0008273", dims = c(1, 3))
#>                  dim_num  cat_id categ_cod categ_dsg categ_ord categ_nivel
#> Dim_Num1_S7A2011       1 S7A2011   S7A2011      2011  20110101           1
#> Dim_Num1_S7A2012       1 S7A2012   S7A2012      2012  20120101           1
#> Dim_Num1_S7A2013       1 S7A2013   S7A2013      2013  20130101           1
#> Dim_Num1_S7A2014       1 S7A2014   S7A2014      2014  20140101           1
#> Dim_Num1_S7A2015       1 S7A2015   S7A2015      2015  20150101           1
#> Dim_Num1_S7A2016       1 S7A2016   S7A2016      2016  20160101           1
#> Dim_Num1_S7A2017       1 S7A2017   S7A2017      2017  20170101           1
#> Dim_Num1_S7A2018       1 S7A2018   S7A2018      2018  20180101           1
#> Dim_Num1_S7A2019       1 S7A2019   S7A2019      2019  20190101           1
#> Dim_Num1_S7A2020       1 S7A2020   S7A2020      2020  20200101           1
#> Dim_Num1_S7A2021       1 S7A2021   S7A2021      2021  20210101           1
#> Dim_Num1_S7A2022       1 S7A2022   S7A2022      2022  20220101           1
#> Dim_Num1_S7A2023       1 S7A2023   S7A2023      2023  20230101           1
#> Dim_Num3_T             3       T         T        MF         1           1
#> Dim_Num3_1             3       1         1         M         2           2
#> Dim_Num3_2             3       2         2         F         3           2
```

The `categ_cod` column contains the codes you pass to `get_data()` as
dimension filters.

## Preview before downloading

Before fetching data, you can check how many API requests will be
needed. 1 API request = 1 chunk:

``` r

ine$preview_chunks("0008206")
#> Indicator '0008206' would require 66 chunks (estimated 63,293,076 rows)
```

This is especially useful for large indicators. You can also preview
with filters to see how they reduce the number of chunks:

``` r

ine$preview_chunks("0008206", dim1 = "S7A2023")
#> Indicator '0008206' would require 3 chunks (estimated 1,471,932 rows)
```

## Fetch data

The simplest way to get data is `get_data()`, which downloads and
processes in one step:

``` r

df <- ine$get_data("0008273", dim1 = "S7A2023", dim2 = c("PT", "1", "2", "3"))
head(df)
#>   dim_1 geocod                     geodsg dim_3 dim_3_t dim_4     dim_4_t
#> 1  2023      3 Região Autónoma da Madeira     1       M    11 0 - 4 years
#> 2  2023      3 Região Autónoma da Madeira     2       F    11 0 - 4 years
#> 3  2023      3 Região Autónoma da Madeira     T      MF    11 0 - 4 years
#> 4  2023      3 Região Autónoma da Madeira     1       M    12 5 - 9 years
#> 5  2023      3 Região Autónoma da Madeira     2       F    12 5 - 9 years
#> 6  2023      3 Região Autónoma da Madeira     T      MF    12 5 - 9 years
#>   ind_string valor
#> 1      4 819  4819
#> 2      4 504  4504
#> 3      9 323  9323
#> 4      5 226  5226
#> 5      5 105  5105
#> 6     10 331 10331
```

Dimension filters use the `categ_cod` values from `get_dim_values()`. If
you omit a dimension, all values are returned.

## Download large indicators

For large indicators, you may prefer to download first and load later.
This streams data to disk without holding it in memory:

``` r

ine$use_cache <- TRUE
ine$cache_dir <- file.path(tempdir(), "ineptr2_vignette")
ine$download_data("0008273", dim1 = "S7A2023")
#> Metadata cached at: /tmp/Rtmpj8kpsp/ineptr2_vignette/ine_0008273_EN_meta.json
#> Downloaded chunk 1/1
#> All 1 chunks downloaded and cached
#> Data downloaded to: /tmp/Rtmpj8kpsp/ineptr2_vignette/ine_0008273_EN_chunks
```

If a download is interrupted, calling `download_data()` again resumes
from where it left off.

Load the raw data later (even in a new session):

``` r

raw <- ine$load_raw_data("0008273")
str(raw, max.level = 2)
#> List of 2
#>  $ responses:List of 1
#>   ..$ :List of 1
#>  $ urls     : chr "https://www.ine.pt/ine/json_indicador/pindica.jsp?op=2&varcd=0008273&lang=EN&Dim1=S7A2023"
```

## Enable caching

By default, every call hits the API. Enable caching to avoid redundant
requests:

``` r

ine$use_cache <- TRUE
```

With caching on, `get_data()` stores the processed result. Subsequent
calls with the same or narrower filters are served from cache. See the
[How caching
works](https://c-matos.github.io/ineptr2/articles/caching.html) article
for details.

## Check for updates

If you’ve previously downloaded data and want to know whether the
indicator has been updated since:

``` r

ine$is_updated("0008273", metadata = ine$get_metadata("0008273"))
#> Using cached metadata
#> [1] FALSE
# OR
ine$is_updated("0008273", last_updated = "2024-01-01")
#> [1] TRUE
```

Assuming you had cached metadata for indicator `0008206`, the field
`DataUltimaAtualizacao` would be used. Alternatively, provide a specific
`last_updated` date.

## Validate an indicator

Not sure if an indicator code exists?

``` r

ine$is_valid("0008273")
#> Using cached metadata
#> [1] TRUE
ine$is_valid("0000000")
#> Metadata cached at: /tmp/Rtmpj8kpsp/ineptr2_vignette/ine_0000000_EN_meta.json
#> [1] FALSE
```

## Browse the indicator catalog

Not sure which indicator you need? You can download the full INE catalog
— a table with all 13,000+ indicators, their codes, names, and themes:

``` r

ine$download_catalog()
catalog <- ine$get_catalog()
head(catalog)
```

The catalog download is slow (~10 minutes) but is cached on disk, so
subsequent calls return instantly.

## Switch language

All output, including dimension labels and descriptions, follows the
client’s language setting:

``` r

ine_pt <- INEClient$new(lang = "PT")
ine_pt$info("0008273")
#> <INE Indicator>
#>   Código: 0008273
#>   Nome: População residente (N.º) por Local de residência (NUTS - 2013), Sexo e Grupo etário; Anual - INE, Estimativas anuais da população residente
#>   Periodicidade: Anual (2011 - 2023)
#>   Últ. atualização: 2024-06-18
#>   Dimensões: 4
#>     dim1: Período de referência dos dados (13 valores)
#>     dim2: Local de residência (NUTS - 2013) (344 valores)
#>     dim3: Sexo (3 valores)
#>     dim4: Grupo etário (19 valores)
```

## Clear cache

No longer need the indicator? Clean the cache on your way out.

``` r

# cleanup - single indicator
ine$clear_cache("0008273")
#> Cache cleared for indicator: 0008273

# everything
# WARNING: This will remove `cache_dir` and all its contents.
ine$clear_cache()
#> Cache cleared: /tmp/Rtmpj8kpsp/ineptr2_vignette
```
