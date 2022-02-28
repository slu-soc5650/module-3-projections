Meeting Examples, Completed
================
Christopher Prener, Ph.D.
(February 28, 2022)

## Introduction

This notebook provides examples of:

-   working with projections with `st_transform()`,
-   projecting points with `st_as_sf()`,
-   saving geometric data with `st_read()`,
-   calculating area with `st_area()` and the `measurements` package,
-   and use the `shape` and `size` arguments with `ggplot2`.

## Dependencies

This notebook requires a number of packages:

``` r
# tidyverse packages
library(dplyr)       # data cleaning
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
library(ggplot2)     # static mapping
library(readr)       # read/write tabular data

# spatial packages
library(mapview)     # preview spatial data
library(sf)          # spatial data tools
```

    ## Linking to GEOS 3.8.1, GDAL 3.2.1, PROJ 7.2.1

``` r
library(tigris)      # access TIGER/line data
```

    ## To enable 
    ## caching of data, set `options(tigris_use_cache = TRUE)` in your R script or .Rprofile.

``` r
# other packages
library(here)        # file path management
```

    ## here() starts at /Users/prenercg/GitHub/slu-soc5650/module-3-projections

``` r
library(measurements) # unit conversion
library(naniar)     # missing data
```

## Load Data

This notebook requires one set of data:

``` r
sluPlaces <- read_csv(here("data", "example-data", "sluPlaces.csv"))
```

    ## Rows: 6 Columns: 4

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): name
    ## dbl (3): id, lng, lat

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Data Cleaning Notes

Make sure your `x` and `y` coordinate variables are numeric or double:

``` r
str(sluPlaces)
```

    ## spec_tbl_df [6 × 4] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ id  : num [1:6] 1 2 3 4 5 6
    ##  $ name: chr [1:6] "Morrissey Hall" "Starbucks" "Simon Rec" "Pius Library" ...
    ##  $ lng : num [1:6] -90.2 -90.2 -90.2 -90.2 -90.2 ...
    ##  $ lat : num [1:6] 38.6 38.6 38.6 38.6 38.6 ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   id = col_double(),
    ##   ..   name = col_character(),
    ##   ..   lng = col_double(),
    ##   ..   lat = col_double()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

If they are not, use `mutate()` with `as.numeric()` to convert them.

Next, we need to make sure there are no missing data in our coordinate
columns. This will cause errors when we go to project these data.

``` r
miss_var_summary(sluPlaces)
```

    ## # A tibble: 4 × 3
    ##   variable n_miss pct_miss
    ##   <chr>     <int>    <dbl>
    ## 1 id            0        0
    ## 2 name          0        0
    ## 3 lng           0        0
    ## 4 lat           0        0

Great! We have no missing data in `lng` and `lat`. If we did, we would
need to use `filter()` from `dplyr` to remove those missing values. Also
be aware of coordinates that are set to 0,0. These can sometimes be used
to identify missing data as well.

``` r
sluPlaces %>%
  filter(lng == 0 | lat == 0)
```

    ## # A tibble: 0 × 4
    ## # … with 4 variables: id <dbl>, name <chr>, lng <dbl>, lat <dbl>

Again, excellent, no missing data!

## Identifying Coordinates

Identifying the coordinate system that `lng` and `lat` represent can be
challenging:

-   Sometimes, the “metadata” that your tabular data come with (if they
    come with any at all) will state what coordinate system was used.
    This is not typical, however.
-   Decimal degrees are one common way that we represent points. It’s
    helpful to know roughly the longitude and latitude of the area you
    are working in. For example, St. Louis is roughly located at 38
    degrees north and 90.2 degrees west. When I see `-90.2` in the
    longitude column, this immediately suggests to me that we have
    decimal degrees data here. If the data originate in the United
    States, they’re typically encoded using NAD 1983. If they are
    international data, they’ll use WGS 1984.
-   Another common way to represent data are with State Plane coordinate
    systems. There isn’t an intuitive way to identify these coordinates.
    It’s also important to know that the State Plane system ships with
    both feet and meters measurement options. Points encoded in Missouri
    State Plane East, for example, will either be encoded using feet or
    meters. Unfortunately, the `x,y` pairs for State Plane meters do not
    correspond to State Plane feet. So, we need to experiment here. Many
    local municipalities rely on State Plane for mapping out of
    tradition, and it’s therefore common to run into both the feet and
    meters versions when working with local data.
-   It’s also worth remember that some users may encode their points
    using UTM zones. This is not nearly as common as State Plane.

Once we have candidates for projecting our points, we need to identify
the CRS/EPSG values (or, alternatively, the Proj4 strings) that
correspond to our coordinate system candidate(s). For this, we’ll use
websites like [EPSG.io](https://epsg.io/) and [Spatial
Reference](https://spatialreference.org/).

## Project Data

First, we want to convert these data to from a `tibble` to an `sf`
object with `st_as_sf()`. We use the `lng` variable as our `x` variable
and `lat` as our `y` variable, and use `4269` for our `crs` argument
since these data are in decimal degrees and this corresponds to the
likely coordinate system we identified above:

``` r
sluPlaces_sf <- st_as_sf(sluPlaces, coords = c("lng", "lat"), crs = 4269)
```

Next, we want to confirm that this worked:

``` r
mapview(sluPlaces_sf)
```

    ## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.

<div id="htmlwidget-c0583b52c46257cff35b" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-c0583b52c46257cff35b">{"x":{"options":{"minZoom":1,"maxZoom":52,"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}},"preferCanvas":false,"bounceAtZoomLimits":false,"maxBounds":[[[-90,-370]],[[90,370]]]},"calls":[{"method":"addProviderTiles","args":["CartoDB.Positron","CartoDB.Positron","CartoDB.Positron",{"errorTileUrl":"","noWrap":false,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["CartoDB.DarkMatter","CartoDB.DarkMatter","CartoDB.DarkMatter",{"errorTileUrl":"","noWrap":false,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["OpenStreetMap","OpenStreetMap","OpenStreetMap",{"errorTileUrl":"","noWrap":false,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["Esri.WorldImagery","Esri.WorldImagery","Esri.WorldImagery",{"errorTileUrl":"","noWrap":false,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["OpenTopoMap","OpenTopoMap","OpenTopoMap",{"errorTileUrl":"","noWrap":false,"detectRetina":false,"pane":"tilePane"}]},{"method":"createMapPane","args":["point",440]},{"method":"addCircleMarkers","args":[[38.637547,38.635959,38.635283,38.636741,38.634973,38.636897],[-90.237104,-90.234837,-90.235841,-90.234972,-90.232955,-90.236313],6,null,"sluPlaces_sf",{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}},"pane":"point","stroke":true,"color":"#333333","weight":1,"opacity":0.9,"fill":true,"fillColor":"#6666FF","fillOpacity":0.6},null,null,["<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>1&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>1&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>Morrissey Hall&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>","<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>2&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>2&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>Starbucks&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>","<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>3&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>3&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>Simon Rec&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>","<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>4&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>4&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>Pius Library&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>","<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>5&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>5&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>BSC&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>","<div class='scrollableContainer'><table class=mapview-popup id='popup'><tr class='coord'><td><\/td><th><b>Feature ID&emsp;<\/b><\/th><td>6&emsp;<\/td><\/tr><tr><td>1<\/td><th>id&emsp;<\/th><td>6&emsp;<\/td><\/tr><tr><td>2<\/td><th>name&emsp;<\/th><td>The Pond&emsp;<\/td><\/tr><tr><td>3<\/td><th>geometry&emsp;<\/th><td>sfc_POINT&emsp;<\/td><\/tr><\/table><\/div>"],{"maxWidth":800,"minWidth":50,"autoPan":true,"keepInView":false,"closeButton":true,"closeOnClick":true,"className":""},["1","2","3","4","5","6"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]},{"method":"addScaleBar","args":[{"maxWidth":100,"metric":true,"imperial":true,"updateWhenIdle":true,"position":"bottomleft"}]},{"method":"addHomeButton","args":[-90.237104,38.634973,-90.232955,38.637547,true,"sluPlaces_sf","Zoom to sluPlaces_sf","<strong> sluPlaces_sf <\/strong>","bottomright"]},{"method":"addLayersControl","args":[["CartoDB.Positron","CartoDB.DarkMatter","OpenStreetMap","Esri.WorldImagery","OpenTopoMap"],"sluPlaces_sf",{"collapsed":true,"autoZIndex":true,"position":"topleft"}]},{"method":"addLegend","args":[{"colors":["#6666FF"],"labels":["sluPlaces_sf"],"na_color":null,"na_label":"NA","opacity":1,"position":"topright","type":"factor","title":"","extra":null,"layerId":null,"className":"info legend","group":"sluPlaces_sf"}]}],"limits":{"lat":[38.634973,38.637547],"lng":[-90.237104,-90.232955]},"fitBounds":[38.634973,-90.237104,38.637547,-90.232955,[]]},"evals":[],"jsHooks":{"render":[{"code":"function(el, x, data) {\n  return (\n      function(el, x, data) {\n      // get the leaflet map\n      var map = this; //HTMLWidgets.find('#' + el.id);\n      // we need a new div element because we have to handle\n      // the mouseover output separately\n      // debugger;\n      function addElement () {\n      // generate new div Element\n      var newDiv = $(document.createElement('div'));\n      // append at end of leaflet htmlwidget container\n      $(el).append(newDiv);\n      //provide ID and style\n      newDiv.addClass('lnlt');\n      newDiv.css({\n      'position': 'relative',\n      'bottomleft':  '0px',\n      'background-color': 'rgba(255, 255, 255, 0.7)',\n      'box-shadow': '0 0 2px #bbb',\n      'background-clip': 'padding-box',\n      'margin': '0',\n      'padding-left': '5px',\n      'color': '#333',\n      'font': '9px/1.5 \"Helvetica Neue\", Arial, Helvetica, sans-serif',\n      'z-index': '700',\n      });\n      return newDiv;\n      }\n\n\n      // check for already existing lnlt class to not duplicate\n      var lnlt = $(el).find('.lnlt');\n\n      if(!lnlt.length) {\n      lnlt = addElement();\n\n      // grab the special div we generated in the beginning\n      // and put the mousmove output there\n\n      map.on('mousemove', function (e) {\n      if (e.originalEvent.ctrlKey) {\n      if (document.querySelector('.lnlt') === null) lnlt = addElement();\n      lnlt.text(\n                           ' lon: ' + (e.latlng.lng).toFixed(5) +\n                           ' | lat: ' + (e.latlng.lat).toFixed(5) +\n                           ' | zoom: ' + map.getZoom() +\n                           ' | x: ' + L.CRS.EPSG3857.project(e.latlng).x.toFixed(0) +\n                           ' | y: ' + L.CRS.EPSG3857.project(e.latlng).y.toFixed(0) +\n                           ' | epsg: 3857 ' +\n                           ' | proj4: +proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +no_defs ');\n      } else {\n      if (document.querySelector('.lnlt') === null) lnlt = addElement();\n      lnlt.text(\n                      ' lon: ' + (e.latlng.lng).toFixed(5) +\n                      ' | lat: ' + (e.latlng.lat).toFixed(5) +\n                      ' | zoom: ' + map.getZoom() + ' ');\n      }\n      });\n\n      // remove the lnlt div when mouse leaves map\n      map.on('mouseout', function (e) {\n      var strip = document.querySelector('.lnlt');\n      if( strip !==null) strip.remove();\n      });\n\n      };\n\n      //$(el).keypress(67, function(e) {\n      map.on('preclick', function(e) {\n      if (e.originalEvent.ctrlKey) {\n      if (document.querySelector('.lnlt') === null) lnlt = addElement();\n      lnlt.text(\n                      ' lon: ' + (e.latlng.lng).toFixed(5) +\n                      ' | lat: ' + (e.latlng.lat).toFixed(5) +\n                      ' | zoom: ' + map.getZoom() + ' ');\n      var txt = document.querySelector('.lnlt').textContent;\n      console.log(txt);\n      //txt.innerText.focus();\n      //txt.select();\n      setClipboardText('\"' + txt + '\"');\n      }\n      });\n\n      }\n      ).call(this.getMap(), el, x, data);\n}","data":null},{"code":"function(el, x, data) {\n  return (function(el,x,data){\n           var map = this;\n\n           map.on('keypress', function(e) {\n               console.log(e.originalEvent.code);\n               var key = e.originalEvent.code;\n               if (key === 'KeyE') {\n                   var bb = this.getBounds();\n                   var txt = JSON.stringify(bb);\n                   console.log(txt);\n\n                   setClipboardText('\\'' + txt + '\\'');\n               }\n           })\n        }).call(this.getMap(), el, x, data);\n}","data":null}]}}</script>

Excellent!

## Transform Our Projection

We’ve already used `st_transform()`, but now can do so with purpose. For
example, to convert our data to State Plane (meters). We’ll use the data
based on the 2007 update to NAD 1983:

``` r
sluPlaces_statePlane <- st_transform(sluPlaces_sf, crs = 3601)
```

If we need our data in feet, there are also options. However, these are
ESRI products that are not included in the `sf` package. How do we know?
We can use the EPSG value `102696`.

``` r
st_transform(sluPlaces_sf, crs = 102696)
```

Note that error includes this language -
`GDAL Error 1: PROJ: proj_create_from_database: crs not found`. Even so,
we can still use the coordinate system by specifying the `Proj4` string
value instead of the CRS number:

``` r
sluPlaces_statePlane_ft <- st_transform(sluPlaces_sf, crs = "ESRI:102696")
```

Sometimes this doesn’t work, and so it is also useful to know how to
apply `Proj4` strings as well.

``` r
sluPlaces_statePlane_ft <- st_transform(sluPlaces_sf, crs = "+proj=tmerc +lat_0=35.83333333333334 +lon_0=-90.5 +k=0.9999333333333333 +x_0=250000 +y_0=0 +ellps=GRS80 +datum=NAD83 +to_meter=0.3048006096012192 +no_defs ")
```

This should give us correctly projected data. Using this trick with the
`Proj4` strings also works, by the way, with `st_as_sf()` as well.

## Write Data

Finally, we’ll write our data:

``` r
st_write(sluPlaces_statePlane, here("data", "example-data", "clean-data", "sluPlaces.shp"), delete_dsn = TRUE)
```

    ## Deleting source `/Users/prenercg/GitHub/slu-soc5650/module-3-projections/data/example-data/clean-data/sluPlaces.shp' using driver `ESRI Shapefile'
    ## Writing layer `sluPlaces' to data source 
    ##   `/Users/prenercg/GitHub/slu-soc5650/module-3-projections/data/example-data/clean-data/sluPlaces.shp' using driver `ESRI Shapefile'
    ## Writing 6 features with 2 fields and geometry type Point.

The `st_write()` function identifies the file type from what you include
at the end of the `here()` statement. If I am working solely in `R`, I
tend to use `.geojson` because:

1.  It does not impose limits on column names or data types.
2.  It is an open standard for storing data that is plain text.
3.  It can be previewed live on GitHub.com (where I share most my data).

However, if you are going to be working in the ESRI ecosystem, saving
data as shapefiles is suggested. Note that \#1 above needs to be
addressed - keep variable names short (8 characters or less) and convert
big numbers to strings or remove them completely before saving.

## Using Projections to Calculate Area

One final task we have when working with projections is to use them as
the basis for making calculations. Often, this entails calculating area
so that we can use it for normalizing our data. Sometimes our data come
with measurements, but we don’t know what those units are. Other times,
we don’t have an area measure at all. Consider these data from `tigris`:

``` r
moCounties <- counties(state = 29) %>%
  select(GEOID, NAMELSAD, ALAND, AWATER)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |                                                                      |   1%  |                                                                              |=                                                                     |   1%  |                                                                              |=                                                                     |   2%  |                                                                              |==                                                                    |   2%  |                                                                              |==                                                                    |   3%  |                                                                              |==                                                                    |   4%  |                                                                              |===                                                                   |   4%  |                                                                              |===                                                                   |   5%  |                                                                              |====                                                                  |   5%  |                                                                              |====                                                                  |   6%  |                                                                              |=====                                                                 |   6%  |                                                                              |=====                                                                 |   7%  |                                                                              |=====                                                                 |   8%  |                                                                              |======                                                                |   8%  |                                                                              |=======                                                               |  10%  |                                                                              |========                                                              |  12%  |                                                                              |=========                                                             |  13%  |                                                                              |=========                                                             |  14%  |                                                                              |==========                                                            |  14%  |                                                                              |==========                                                            |  15%  |                                                                              |===========                                                           |  15%  |                                                                              |===========                                                           |  16%  |                                                                              |============                                                          |  16%  |                                                                              |============                                                          |  17%  |                                                                              |=============                                                         |  19%  |                                                                              |==============                                                        |  20%  |                                                                              |==============                                                        |  21%  |                                                                              |===============                                                       |  21%  |                                                                              |===============                                                       |  22%  |                                                                              |================                                                      |  22%  |                                                                              |================                                                      |  23%  |                                                                              |=================                                                     |  25%  |                                                                              |==================                                                    |  26%  |                                                                              |===================                                                   |  27%  |                                                                              |===================                                                   |  28%  |                                                                              |====================                                                  |  28%  |                                                                              |====================                                                  |  29%  |                                                                              |======================                                                |  31%  |                                                                              |=======================                                               |  33%  |                                                                              |========================                                              |  35%  |                                                                              |=========================                                             |  35%  |                                                                              |=========================                                             |  36%  |                                                                              |==========================                                            |  38%  |                                                                              |============================                                          |  40%  |                                                                              |=============================                                         |  41%  |                                                                              |=============================                                         |  42%  |                                                                              |==============================                                        |  42%  |                                                                              |==============================                                        |  43%  |                                                                              |================================                                      |  45%  |                                                                              |=================================                                     |  47%  |                                                                              |==================================                                    |  49%  |                                                                              |====================================                                  |  51%  |                                                                              |=====================================                                 |  53%  |                                                                              |=======================================                               |  55%  |                                                                              |=======================================                               |  56%  |                                                                              |========================================                              |  56%  |                                                                              |========================================                              |  57%  |                                                                              |========================================                              |  58%  |                                                                              |=========================================                             |  58%  |                                                                              |=========================================                             |  59%  |                                                                              |==========================================                            |  59%  |                                                                              |==========================================                            |  60%  |                                                                              |===========================================                           |  61%  |                                                                              |============================================                          |  63%  |                                                                              |============================================                          |  64%  |                                                                              |=============================================                         |  64%  |                                                                              |=============================================                         |  65%  |                                                                              |==============================================                        |  65%  |                                                                              |==============================================                        |  66%  |                                                                              |===============================================                       |  68%  |                                                                              |=================================================                     |  69%  |                                                                              |=================================================                     |  70%  |                                                                              |=================================================                     |  71%  |                                                                              |==================================================                    |  71%  |                                                                              |==================================================                    |  72%  |                                                                              |====================================================                  |  74%  |                                                                              |=====================================================                 |  76%  |                                                                              |======================================================                |  76%  |                                                                              |======================================================                |  77%  |                                                                              |======================================================                |  78%  |                                                                              |========================================================              |  80%  |                                                                              |=========================================================             |  82%  |                                                                              |==========================================================            |  82%  |                                                                              |==========================================================            |  83%  |                                                                              |==========================================================            |  84%  |                                                                              |============================================================          |  86%  |                                                                              |=============================================================         |  88%  |                                                                              |==============================================================        |  88%  |                                                                              |==============================================================        |  89%  |                                                                              |===============================================================       |  89%  |                                                                              |===============================================================       |  90%  |                                                                              |================================================================      |  92%  |                                                                              |==================================================================    |  94%  |                                                                              |==================================================================    |  95%  |                                                                              |===================================================================   |  95%  |                                                                              |===================================================================   |  96%  |                                                                              |====================================================================  |  96%  |                                                                              |===================================================================== |  98%  |                                                                              |======================================================================| 100%

There are both `ALAND` and `AWATER` columns, but it isn’t immediately
clear what their units are (they are in meters, FYI). If we want the
total area of these units, we can combine a few functions from `sf`,
`dplyr`, and `measurements` to achieve a few different outcomes. First,
we’ll re-calculate the area based on a **projected** coordinate system.
We need to move from the current coordinate system to a projected
coordinate system. To get a sense of our starting place, we’ll use
`st_crs()`:

``` r
st_crs(moCounties)
```

    ## Coordinate Reference System:
    ##   User input: NAD83 
    ##   wkt:
    ## GEOGCRS["NAD83",
    ##     DATUM["North American Datum 1983",
    ##         ELLIPSOID["GRS 1980",6378137,298.257222101,
    ##             LENGTHUNIT["metre",1]]],
    ##     PRIMEM["Greenwich",0,
    ##         ANGLEUNIT["degree",0.0174532925199433]],
    ##     CS[ellipsoidal,2],
    ##         AXIS["latitude",north,
    ##             ORDER[1],
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##         AXIS["longitude",east,
    ##             ORDER[2],
    ##             ANGLEUNIT["degree",0.0174532925199433]],
    ##     ID["EPSG",4269]]

This confirms that we are using a geographic coordinate system (this is
typical for TIGER/Line data). We’ll switch to Albers Equal Area Conic
for the contiguous United States:

``` r
moCounties <- st_transform(moCounties, crs = "ESRI:102003")
# moCounties <- st_transform(moCounties, crs = "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +ellps=GRS80 +datum=NAD83 +units=m +no_defs")
```

I’ve also included the `Proj4` string, but it is commented out. Notice
how `+units=2` appears in the `Proj4` string. This means are data are in
meters. If we wanted to recalculate our area units meters, we can use
the `st_area()` function and the `geometry` column to do so:

``` r
moCounties %>%
  select(-c("ALAND", "AWATER")) %>%
  mutate(sq_m = as.numeric(st_area(geometry)), .after = NAMELSAD) %>%
  mutate(sq_km = conv_unit(sq_m, from = "m2", to = "km2"), .after = sq_m) -> moCounties
```

We can do the same thing if we want to convert to square miles:

``` r
moCounties <- mutate(moCounties, sq_mi = conv_unit(sq_m, from = "m2", to = "mi2"), .after = sq_km)
```

Writing the `sq_m` column to shapefile might be tricky, so its best to
do these conversions and then remove the source column before saving!

## Symbolizing Points

One thing I wanted to share quickly this week is how we can approach
displaying points with `ggplot2`, since we happen to have some point
data to work with. We’ll use the `shape` and `size` arguments to make
simple adjustments:

``` r
ggplot() +
  geom_sf(data = sluPlaces_sf, shape = 18, size = 4)
```

![](meeting-3-1-examples-complete_files/figure-gfm/symbolize-points-1.png)<!-- -->

Some point symbols - those with values 21 through 25 - can be customized
with fills and colors as well:

``` r
ggplot() +
  geom_sf(data = sluPlaces_sf, shape = 22, size = 6, fill = "#9d4a9d", color = "#5d92e5")
```

![](meeting-3-1-examples-complete_files/figure-gfm/symbolize-points-custom-1.png)<!-- -->
