---
id: 1643
title: 'Bounding Boxes for all US Counties'
date: '2021-11-05T07:12:23+00:00'
author: Anthony Louis D'Agostino
layout: post
guid: 'https://anthonylouisdagostino.com/?p=1643'
permalink: /bounding-boxes-for-all-us-counties/
image: /wp-content/uploads/2021/11/county_boundingboxes.png
categories:
    - Data
    - Geography
    - GIS
    - Visualization
---

![image](/wp-content/uploads/2021/11/county_boundingboxes.png)
A post from several years back contained the [bounding box coordinates of all US states](https://anthonylouisdagostino.com/bounding-boxes-for-all-us-states/) and has been one of the more viewed pages on this site. Unfortunately, if your area of interest is below the state-level, these bounding boxes may only get you part of the way to your destination. Why waste time expanding a geographic search to areas beyond your narrow AOI?

If you’re running county-level analyses and need the latitude and longitude bounding box endpoints, then this is the table for you. These values were generated using the TIGER/Line 2021 shapefiles based on Census 2020 geographies, which you can grab from the [Census FTP site](https://www2.census.gov/geo/tiger/TIGER2021/).

Need to construct bounding boxes for some geometry collection of your choice? This *sf/tidyverse* chunk might be a good start. ```
<pre class="wp-block-code has-small-font-size">```
library(tidyverse)
library(sf)
us.counties <- st_read("tl_2021_us_county.shp", stringsAsFactors = FALSE)
bb.rbind.sf <- split(us.counties, 1:nrow(us.counties)) %>% map( ~ st_bbox(.x) %>% st_as_sfc() %>% as_tibble())  %>% do.call("rbind", . ) %>%
  st_as_sf()
```
```

<div class="is-layout-flow wp-block-group"><div class="wp-block-group__inner-container"></div></div><script src="https://gist.github.com/a8dx/7e550680f7ea6a68f20da00e21d7ce9b.js"></script>
