---
id: 87
title: 'Calculating degree days with NumPy'
date: '2015-01-27T18:51:58+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=87'
permalink: /calculating-degree-days-with-numpy/
geo_public:
    - '0'
categories:
    - Agriculture
    - Computing
---

[![2010-08-08 at 11-41-43](https://pathindependence.files.wordpress.com/2015/01/2010-08-08-at-11-41-43.jpg?w=300&resize=519%2C352)](https://pathindependence.files.wordpress.com/2015/01/2010-08-08-at-11-41-43.jpg)

Michael Roberts offers an extremely [helpful post at G-FEED](http://www.g-feed.com/2015/01/searching-for-critical-thresholds-in.html), including R code for generating the season growing degree days measure used in the seminal [Schlenker Roberts 2009](http://www.pnas.org/content/106/37/15594) paper on nonlinear crop responses to temperature. Since many of the papers claiming to follow the approach don’t actually adhere to it, he’s posted code in the interest of encouraging people to use the sinusoidal fitting method and perhaps improve on their standard errors.

I’ve been reliant on the wonderful [Climate Data Operators](https://code.zmaw.de/projects/cdo) library for working with netCDF files and have therefore been basing most of my current weather/climate data analysis in Python (see the Python bindings for CDO [here](https://github.com/Try2Code/cdo-bindings)). I thought it might be helpful to provide the Python analogue of the R code so that folks in a Python workflow might be up and running. I’ve modified the function to address a single pixel at a time (i.e., you’ll need to write the loop that spans all relevant grid cells), and hence there’s no cell count variable here to average over.

Head over to Michael’s original post, since I’ve offered the code snippet below without documentation. Drop a comment if you find this to be a helpful addition.

\[code language=”python”\]  
def days\_in\_range(t0, t1, tMin, tMax):

 n = len(tMin)  
 t0 = np.array(\[t0\] \* n)  
 t1 = np.array(\[t1\] \* n)  
 t0\[np.where(t0 &lt; tMin)\] = tMin\[np.where(t0 &lt; tMin)\]  
 t1\[np.where(t1 &gt; tMax)\] = tMax\[np.where(t1 &gt; tMax)\]  
 outside = (t0 &gt; tMax) | (t1 &lt; tMin)  
 inside = np.logical\_not(outside)

 def u(z, ind, t\_min, t\_max):  
 z\_t = np.array(\[val for x, val in zip(ind, z) if x\])  
 tMax\_t = np.array(\[val for x, val in zip(ind, t\_max) if x\])  
 tMin\_t = np.array(\[val for x, val in zip(ind, t\_min) if x\])  
 return np.divide(z\_t – tMin\_t, tMax\_t – tMin\_t)

 time\_at\_range = (2/pi)\*((np.arcsin(u(t1, inside, tMin, tMax)) – np.arcsin(u(t0, inside, tMin, tMax))))  
 return np.sum(time\_at\_range)

\[/code\]

To ensure that all results from calculations in the functions (e.g., the vector division in the u function) produces floats, ensure that

\[code language=”python”\] t0, t1 \[/code\]

are treated as floats by including a decimal afterwards (the range of 21-22C should be 21.0 and 22.0).

In a follow-up post, I’ll include snippets that span all pixels in the dataset and summarize the data and folder management system I’ve been using when exporting GDDs for use by Stata or ArcGIS.

I disclaim any responsibility for issues (emotional, financial, or other) associated with use of this code.