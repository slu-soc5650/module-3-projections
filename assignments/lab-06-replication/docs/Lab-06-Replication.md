Lab 06 Replication
================
Branson Fox, BA and Christopher Prener, PhD
(March 12, 2021)

## Introduction

This notebook replicates the results of lab 06

## Dependencies

This notebook requires the following packages to load and manipulate our
data.

``` r
# tidyverse packages
library(dplyr)          # data manipulation
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(ggplot2)        # static mapping
library(readr)          # import tabular data

# spatial packages
library(sf)             # methods for spatial data
```

    ## Linking to GEOS 3.8.1, GDAL 3.1.4, PROJ 6.3.1

``` r
library(mapview)        # interactive map preview
```

    ## GDAL version >= 3.1.0 | setting mapviewOptions(fgb = TRUE)

``` r
library(tigris)         # TIGER/Line Data
```

    ## To enable 
    ## caching of data, set `options(tigris_use_cache = TRUE)` in your R script or .Rprofile.

``` r
# other packages
library(here)           # file path management
```

    ## here() starts at /Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication

``` r
library(RColorBrewer)   # color palettes

# functions
source(here("source", "map_breaks.R"))
```

## Load Data

These are the data we need to complete this lab.

``` r
# spatial 
county_bnd <-st_read(here("data","US_BOUNDARY_Counties","US_BOUNDARY_Counties.shp"))
```

    ## Reading layer `US_BOUNDARY_Counties' from data source `/Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication/data/US_BOUNDARY_Counties/US_BOUNDARY_Counties.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 3220 features and 5 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -179.1743 ymin: 17.91377 xmax: 179.7739 ymax: 71.35256
    ## geographic CRS: GRS 1980(IUGG, 1980)

``` r
# tabular
capitals <- read_csv(here("data","stateCapitals.csv"))
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   name = col_character(),
    ##   description = col_character(),
    ##   latitude = col_double(),
    ##   longitude = col_double()
    ## )

``` r
health_ins <- read_csv(here("data","USHealth","US_HEALTH_noIns.csv"))
```

    ## 
    ## ── Column specification ────────────────────────────────────────────────────────
    ## cols(
    ##   GEOID = col_character(),
    ##   state = col_character(),
    ##   county = col_character(),
    ##   noIns = col_double()
    ## )

## Part 1

### Section A

We’ll start by projecting the capitals data we loaded earlier:

``` r
capitals_sf <- st_as_sf(capitals, coords = c(x = "longitude", y = "latitude"), crs = 4269)
```

Then, we’ll check that we projected properly using `mapview`:

``` r
mapview(capitals_sf)
```

![](Lab-06-Replication_files/figure-gfm/p1-q1-a2-1.png)<!-- -->

Next, we’ll save the data as a shapefile using the `st_write()`
function.

``` r
st_write(capitals_sf, dsn = here("data", "cleanData", "stateCapitals.shp"), delete_dsn = TRUE)
```

    ## Warning in abbreviate_shapefile_names(obj): Field names abbreviated for ESRI
    ## Shapefile driver

    ## Deleting source `/Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication/data/cleanData/stateCapitals.shp' using driver `ESRI Shapefile'
    ## Writing layer `stateCapitals' to data source `/Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication/data/cleanData/stateCapitals.shp' using driver `ESRI Shapefile'
    ## Writing 50 features with 2 fields and geometry type Point.

### Section B

Next, we’ll work with the county boundary and health insurance data.
First, we want to join these data. `GEOID` appears to be a common
variable among them, but it is character in one data object and numeric
in the other. To fix this, we can `mutate` one class to match the other.

``` r
health_ins <- mutate(health_ins, GEOID = as.numeric(GEOID))
```

Now we can table join as we have before. We’ll subset down to the bare
minimum of counties before we do that, however.

``` r
## subset columns
county_bnd <- select(county_bnd, GEOID)

## join
county_health <- left_join(county_bnd, health_ins, by = "GEOID")
```

### Section C

We want to subset our data to remove the observations with missing data.
Since we know that `-1` denotes missing data, we’ll `filter` for
observations where `noIns` is `>=0`

``` r
county_health <- filter(county_health, noIns >= 0)
```

### Section D

We’re almost ready to export the data, but first we need to check the
projection. We can do this with `st_crs()`

``` r
st_crs(county_health)
```

    ## Coordinate Reference System:
    ##   User input: GRS 1980(IUGG, 1980) 
    ##   wkt:
    ## GEOGCRS["GRS 1980(IUGG, 1980)",
    ##     DATUM["D_unknown",
    ##         ELLIPSOID["GRS80",6378137,298.257222101,
    ##             LENGTHUNIT["metre",1,
    ##                 ID["EPSG",9001]]]],
    ##     PRIMEM["Greenwich",0,
    ##         ANGLEUNIT["Degree",0.0174532925199433]],
    ##     CS[ellipsoidal,2],
    ##         AXIS["longitude",east,
    ##             ORDER[1],
    ##             ANGLEUNIT["Degree",0.0174532925199433]],
    ##         AXIS["latitude",north,
    ##             ORDER[2],
    ##             ANGLEUNIT["Degree",0.0174532925199433]]]

We find that there is not the correct coordinate system. We’ll use
`st_transform()` to re-project the data to NAD 1983:

``` r
county_health <- st_transform(county_health, crs = 4269)
```

Finally, we are ready to export:

``` r
st_write(county_health, dsn = here("data", "cleanData", "countyHealth.shp"), delete_dsn = TRUE)
```

    ## Deleting source `/Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication/data/cleanData/countyHealth.shp' using driver `ESRI Shapefile'
    ## Writing layer `countyHealth' to data source `/Users/chris/GitHub/slu-soc5650/content/module-3-projections/assignments/lab-06-replication/data/cleanData/countyHealth.shp' using driver `ESRI Shapefile'
    ## Writing 3135 features with 4 fields and geometry type Multi Polygon.

### Section E

Next, we want to assign map breaks to our data before we get to mapping.
We’ll also do this before we subset our data so that all three maps use
the same breaks.

``` r
county_health <- map_breaks(county_health, var = "noIns", newvar = "map_breaks", classes = 5, style = "fisher")
```

### Section F

Next, we’ll download state boundary data from Tiger/Line using the
`tigris` package and prepare it for mapping.

``` r
## download and remove territories
states <- states() %>%
  filter(NAME %in% c("United States Virgin Islands", "Commonwealth of the Northern Mariana Islands", "Guam",
                     "American Samoa", "Puerto Rico") == FALSE)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |                                                                      |   1%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |======                                                                |   8%  |                                                                              |======                                                                |   9%  |                                                                              |=======                                                               |   9%  |                                                                              |=======                                                               |  10%  |                                                                              |=======                                                               |  11%  |                                                                              |========                                                              |  11%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |=========                                                             |  14%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  17%  |                                                                              |============                                                          |  18%  |                                                                              |=============                                                         |  18%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |==============                                                        |  21%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  23%  |                                                                              |=================                                                     |  24%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |====================                                                  |  29%  |                                                                              |=====================                                                 |  29%  |                                                                              |=====================                                                 |  30%  |                                                                              |=====================                                                 |  31%  |                                                                              |======================                                                |  31%  |                                                                              |======================                                                |  32%  |                                                                              |=======================                                               |  32%  |                                                                              |=======================                                               |  33%  |                                                                              |========================                                              |  34%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  36%  |                                                                              |==========================                                            |  37%  |                                                                              |==========================                                            |  38%  |                                                                              |===========================                                           |  38%  |                                                                              |===========================                                           |  39%  |                                                                              |============================                                          |  39%  |                                                                              |============================                                          |  40%  |                                                                              |============================                                          |  41%  |                                                                              |=============================                                         |  41%  |                                                                              |=============================                                         |  42%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |===============================                                       |  44%  |                                                                              |================================                                      |  45%  |                                                                              |================================                                      |  46%  |                                                                              |=================================                                     |  46%  |                                                                              |=================================                                     |  47%  |                                                                              |=================================                                     |  48%  |                                                                              |==================================                                    |  48%  |                                                                              |==================================                                    |  49%  |                                                                              |===================================                                   |  49%  |                                                                              |===================================                                   |  50%  |                                                                              |===================================                                   |  51%  |                                                                              |====================================                                  |  51%  |                                                                              |====================================                                  |  52%  |                                                                              |=====================================                                 |  52%  |                                                                              |=====================================                                 |  53%  |                                                                              |======================================                                |  54%  |                                                                              |======================================                                |  55%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |==========================================                            |  61%  |                                                                              |===========================================                           |  61%  |                                                                              |===========================================                           |  62%  |                                                                              |============================================                          |  62%  |                                                                              |============================================                          |  63%  |                                                                              |=============================================                         |  64%  |                                                                              |=============================================                         |  65%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  67%  |                                                                              |===============================================                       |  68%  |                                                                              |================================================                      |  68%  |                                                                              |================================================                      |  69%  |                                                                              |=================================================                     |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |===================================================                   |  72%  |                                                                              |===================================================                   |  73%  |                                                                              |====================================================                  |  74%  |                                                                              |====================================================                  |  75%  |                                                                              |=====================================================                 |  75%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  76%  |                                                                              |======================================================                |  77%  |                                                                              |======================================================                |  78%  |                                                                              |=======================================================               |  78%  |                                                                              |=======================================================               |  79%  |                                                                              |========================================================              |  79%  |                                                                              |========================================================              |  80%  |                                                                              |========================================================              |  81%  |                                                                              |=========================================================             |  81%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |===========================================================           |  84%  |                                                                              |===========================================================           |  85%  |                                                                              |============================================================          |  85%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  87%  |                                                                              |=============================================================         |  88%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |===============================================================       |  91%  |                                                                              |================================================================      |  91%  |                                                                              |================================================================      |  92%  |                                                                              |=================================================================     |  92%  |                                                                              |=================================================================     |  93%  |                                                                              |==================================================================    |  94%  |                                                                              |==================================================================    |  95%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  97%  |                                                                              |====================================================================  |  98%  |                                                                              |===================================================================== |  98%  |                                                                              |===================================================================== |  99%  |                                                                              |======================================================================|  99%  |                                                                              |======================================================================| 100%

``` r
## create lower 48, AK, and HI
lower48_boundary <- filter(states, NAME %in% c("Alaska", "Hawaii") == FALSE)
ak_boundary <- filter(states, NAME == "Alaska")
hi_boundary <- filter(states, NAME == "Hawaii")
```

Now we have data that are specific to the three maps we’ll be making.

### Section G

We’ll also remove Alaska and Hawaii from our other data sources as well:

``` r
# capitals data
lower48_capitals <- filter(capitals_sf, name %in% c("Alaska", "Hawaii") == FALSE)
ak_capital <- filter(capitals_sf, name == "Alaska")
hi_capital <- filter(capitals_sf, name == "Hawaii")

# health insurance data
lower48_health <- filter(county_health, state %in% c("Alaska", "Hawaii") == FALSE)
ak_health <- filter(county_health, state == "Alaska")
hi_health <- filter(county_health, state == "Hawaii")
```

Now we have our other data prepared for mapping as well.

## Part 2

### Question 2

We’ll use the [Albers Contiguous
USA](https://spatialreference.org/ref/esri/usa-contiguous-albers-equal-area-conic/)
projected coordinate system, whose EPSG code is not a part of `R`’s
ecosystem.

``` r
## state capitals
lower48_capitals <- st_transform(lower48_capitals, crs = "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs")

## state boundary
lower48_boundary <- st_transform(lower48_boundary, crs = "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs")

## health insurance data
lower48_health <- st_transform(lower48_health, crs = "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs")
```

Other options include the more general [Albers North
America](https://spatialreference.org/ref/esri/102008/) coordinate
system and the [Lambert North
America](https://spatialreference.org/ref/esri/north-america-lambert-conformal-conic/)
coordinate system.

### Question 3

Next, we’ll use these data to make a map of the lower 48 states.

``` r
p1 <- ggplot() +
  geom_sf(data = lower48_boundary, fill = "#ffffff", color = NA) +
  geom_sf(data = lower48_health, mapping = aes(fill = map_breaks), lwd = .4) +
  geom_sf(data = lower48_boundary, fill = NA, color = "#2a2a2a", size = .7) +
  geom_sf(data = lower48_capitals, shape = 18) +
  scale_fill_brewer(palette = "RdPu", name = "% of Residents\nwithout Insurance") +
  labs(
    title = "Insurance Rates by County",
    subtitle = "Contiguous United States",
    caption = "Data via the CDC and Christopher Prener, PhD"
  )

p1
```

![](Lab-06-Replication_files/figure-gfm/p2-q3-1.png)<!-- -->

### Question 4

With our map created, we’ll save it to the `results/` folder:

``` r
ggsave(plot = p1, filename = here("results", "lower48_map.png"))
```

    ## Saving 7 x 5 in image

## Part 3

### Question 5

We’ll use the [Albers
Alaska](https://spatialreference.org/ref/epsg/3467/) projected
coordinate system, which is specifically designed for state-level
mapping in Alaska.

``` r
## state capitals
ak_capital <- st_transform(ak_capital, crs = 3467)

## state boundary
ak_boundary <- st_transform(ak_boundary, crs = 3467)

## health insurance data
ak_health <- st_transform(ak_health, crs = 3467)
```

The other option is the more general [Albers North
America](https://spatialreference.org/ref/esri/102008/) coordinate
system, though it is preferable to use state-specific coordinate systems
for places like Alaska.

### Question 6

Next, we’ll use these data to make a map of the Alaska. One tweak to our
workflow that go above and beyond what we’ve covered in class is
included here: I’ve used `scale_fill_manual()` to specify the three
darkest colors form our map above. If we use the default skill from
class, `scale_fill_brewer()`, it will select the first three color
values from this palette.

``` r
p2 <- ggplot() +
  geom_sf(data = ak_boundary, fill = "#ffffff", color = NA) +
  geom_sf(data = ak_health, mapping = aes(fill = map_breaks), lwd = .4) +
  geom_sf(data = ak_boundary, fill = NA, color = "#2a2a2a", size = .7) +
  geom_sf(data = ak_capital, shape = 18, size = 4) +
  scale_fill_manual(values = brewer.pal(name = "RdPu", n = 5)[3:5],
                    name = "% of Residents\nwithout Insurance") +
  labs(
    title = "Insurance Rates by County",
    subtitle = "Alaska Focus",
    caption = "Data via the CDC and Christopher Prener, PhD"
  )

p2
```

![](Lab-06-Replication_files/figure-gfm/p3-q6-1.png)<!-- -->

This gets us reasonably close to a production quality map, though some
additional tweaks, such as showing specific counties that have missing
data, might make it a nicer final product.

### Question 7

With our map created, we’ll save it to the `results/` folder:

``` r
ggsave(plot = p2, filename = here("results", "alaska_map.png"))
```

    ## Saving 7 x 5 in image

## Part 4

### Question 8

We’ll use the [Albers
Hawaii](https://spatialreference.org/ref/esri/102007/) projected
coordinate system, which is specifically designed for state-level
mapping in Hawaii.

``` r
## state capitals
hi_capital <- st_transform(hi_capital, crs = "+proj=aea +lat_1=8 +lat_2=18 +lat_0=13 +lon_0=-157 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs ")

## state boundary
hi_boundary <- st_transform(hi_boundary, crs = "+proj=aea +lat_1=8 +lat_2=18 +lat_0=13 +lon_0=-157 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs ")

## health insurance data
hi_health <- st_transform(hi_health, crs = "+proj=aea +lat_1=8 +lat_2=18 +lat_0=13 +lon_0=-157 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs ")
```

The other option is a variation of the UTM system that is specific to
Hawaii - [Hawaii UTM Zone
4N](https://spatialreference.org/ref/sr-org/9086/).

### Question 9

Next, we’ll use these data to make a map of the Hawaii.

``` r
p3 <- ggplot() +
  geom_sf(data = hi_health, mapping = aes(fill = map_breaks), color = "#2a2a2a", size = .7) +
  geom_sf(data = hi_capital, shape = 18, size = 4) +
  scale_fill_brewer(palette = "RdPu", name = "% of Residents\nwithout Insurance") +
  labs(
    title = "Insurance Rates by County",
    subtitle = "Hawaii Focus",
    caption = "Data via the CDC and Christopher Prener, PhD"
  )

p3
```

![](Lab-06-Replication_files/figure-gfm/p4-q9-1.png)<!-- -->

After playing around with this map, I decided not to use the
`hi_boundary` data because it added a number of additional islands that
we do not have data for.

### Question 10

With our map created, we’ll save it to the `results/` folder:

``` r
ggsave(plot = p3, filename = here("results", "hawaii_map.png"))
```

    ## Saving 7 x 5 in image
