---
id: 928
title: 'Visualizations Gone Wild'
date: '2017-10-06T01:13:00+00:00'
author: admin
layout: post
guid: 'https://anthonylouisdagostino.com///?p=928'
permalink: /visualizations-gone-wild/
image: /wp-content/uploads/2017/10/desertmountainsandsun.png
categories:
    - Computing
    - Data
    - R
---

> Who said art had to be intentional?

I don’t spend as much time making art as I would like, so it’s always a pleasant surprise when bad code inadvertently helps me fix that.

## Under a Yellowcake Sun

This is “Desert Mountains, Sunken Sun” and is brought to you by `ggplot2`.

![DesertMountainsandSun](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2017/10/desertmountainsandsun.png?resize=487%2C421&ssl=1)

Using`stat_binhex()`you can create colorful heat map density plots with hexagons representing a collection of observations with adjacent values. There is a problem with stretch, such that hexagons will becomes exceptionally large if few observations with comparable values exist.

What’s the quick solution? Toggle the number of bins over which the data is represented using `stat_binhex(bins = n)`, where the default `n` is 30, and consider applying a log or log10 transform (e.g., `trans = "log10"` in your `scale_fill_XXXX` command) which controls the values that correspond to the legend’s tick marks. You’ll likely have to toy around with a few combinations before finding the most visually informative values.

## A Fragmented Country

![PM2p5_ZCTA5_AnyBreaks](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/07/pm2p5_zcta5_anybreaks-e1531162151710.png?resize=712%2C448&ssl=1)

This was a byproduct of a poorly executed filtering procedure using a [Census ZCTA5 shapefile](https://www.census.gov/geo/maps-data/data/cbf/cbf_zcta.html). `plot(st_geometry(object["column"))` helped drive us home.