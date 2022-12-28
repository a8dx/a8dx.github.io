---
id: 1292
title: "Identify nth-Degree Neighbors Using R's Simple Features Package, Simply"
date: '2018-01-19T21:21:32+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=1292'
permalink: /identify-nth-degree-neighbors-using-rs-simple-features-package-simply/
timeline_notification:
    - '1516396896'
image: /wp-content/uploads/2018/01/chatham_nc_neighbors.jpeg
categories:
    - Computing
    - Data
    - R
    - Visualization
---

You’re more likely to complain about the [neighbors upstairs who are making noise](https://www.youtube.com/watch?v=4IRB0sxw-YU) after midnight than those in an apartment two buildings away. Proximity matters and that’s patently obvious, but oftentimes it takes a bit of work to identify who is close and who isn’t. While raster data is packaged in a consistent gridded format for which inverse distance weighting schemes readily can be applied, shapefiles with oddly-shaped features, like [these gerrymandered districts](https://www.washingtonpost.com/news/wonk/wp/2014/05/15/americas-most-gerrymandered-congressional-districts/?utm_term=.c9373f790cfe), may present more of a challenge. Fortunately the [simple features library in R](https://github.com/r-spatial/sf) can save the day and with little sweat on your brow.

This brief tutorial walks through an example of identifying all first- (all my neighbors who share a common edge with me) and second-degree neighbors (all neighbors of my neighbors) of a given county in North Carolina. Since the map is bundled with the library install, no additional downloading is needed. Everything is generalizable and extending to higher-order neighbors is straightforward, but probably unnecessary. While some great R packages exist to perform this computation in a graph setting, I think sf is superior for spatial data.

A couple years ago my buddy and co-author [Eyal Frank](http://www.eyalfrank.com/) published a post outlining a similar ArcGIS workflow, but since then the [sf library](https://github.com/r-spatial/sf) for R has come online and greatly expands R’s spatial analysis capabilities, to the extent that much of what was previously achievable (and easy) through Arc products can now be run natively in R – plus you don’t have to pray that your session will complete before the program crashes. We can achieve similar ends with less code, and without giving ESRI all of our money. *You can instead give it to [R Consortium](https://www.r-consortium.org/projects) which provides financial support to awesome R developers like [Edzer Pebesma](https://github.com/edzer) and [Jeroen Ooms](https://github.com/jeroen) for new creations.*

The complete code is [here](https://gist.github.com/a8dx/7f588b7da531e93049b2b269a3670c89).

Let’s start with the North Carolina county-level map packaged with your sf install, as an example. Let’s select Chatham County, since it’s in the middle of the state and will therefore give us many second-degree neighbors without needing to cross state lines. For now we start with the entire state, and then will drill down to just Chatham.

The heavy lifting is done in the FIRSTdegreeNeighbors function which uses `st_touches` to return index values corresponding to all adjacent entries. This is performed for every county in the state and with some finagling, we can produce a data frame of all positive matches.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist85730378" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-firstdegreeneighbors-r"><div class="Box-body p-0 blob-wrapper data type-r  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | <span class="pl-en">FIRSTdegreeNeighbors</span> <span class="pl-k">&lt;-</span> <span class="pl-k">function</span>(<span class="pl-smi">x</span>) { |
|---|---|
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — use sf functionality and output to sparse matrix format for minimizing footprint </span> |
|  | <span class="pl-smi">first.neighbor</span> <span class="pl-k">&lt;-</span> st\_touches(<span class="pl-smi">x</span>, <span class="pl-smi">x</span>, <span class="pl-v">sparse</span> <span class="pl-k">=</span> <span class="pl-c1">TRUE</span>) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — convert results to data.frame (via sparse matrix)</span> |
|  | <span class="pl-smi">n.ids</span> <span class="pl-k">&lt;-</span> sapply(<span class="pl-smi">first.neighbor</span>, <span class="pl-smi">length</span>) |
|  | <span class="pl-smi">vals</span> <span class="pl-k">&lt;-</span> unlist(<span class="pl-smi">first.neighbor</span>) |
|  | <span class="pl-smi">out</span> <span class="pl-k">&lt;-</span> sparseMatrix(<span class="pl-smi">vals</span>, rep(seq\_along(<span class="pl-smi">n.ids</span>), <span class="pl-smi">n.ids</span>)) |
|  |  |
|  | <span class="pl-smi">out.summ</span> <span class="pl-k">&lt;-</span> summary(<span class="pl-smi">out</span>) <span class="pl-c"><span class="pl-c">\#</span> — this is currently only generating row values, need to map to actual obs \[next line\]</span> |
|  | <span class="pl-k">data.frame</span>(<span class="pl-v">county</span> <span class="pl-k">=</span> <span class="pl-smi">x</span>\[<span class="pl-smi">out.summ</span><span class="pl-k">$</span><span class="pl-smi">j</span>,\]<span class="pl-k">$</span><span class="pl-smi">NAME</span>, |
|  | <span class="pl-v">countyid</span> <span class="pl-k">=</span> <span class="pl-smi">x</span>\[<span class="pl-smi">out.summ</span><span class="pl-k">$</span><span class="pl-smi">j</span>,\]<span class="pl-k">$</span><span class="pl-smi">CNTY\_ID</span>, |
|  | <span class="pl-v">firstdegreeneighbors</span> <span class="pl-k">=</span> <span class="pl-smi">x</span>\[<span class="pl-smi">out.summ</span><span class="pl-k">$</span><span class="pl-smi">i</span>,\]<span class="pl-k">$</span><span class="pl-smi">NAME</span>, |
|  | <span class="pl-v">firstdegreeneighborid</span> <span class="pl-k">=</span> <span class="pl-smi">x</span>\[<span class="pl-smi">out.summ</span><span class="pl-k">$</span><span class="pl-smi">i</span>,\]<span class="pl-k">$</span><span class="pl-smi">CNTY\_ID</span>, |
|  | <span class="pl-v">stringsAsFactors</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>) |
|  | } |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/318ad0cb61befb3a740907e28792a6cd/raw/23b4e0fac62097a0c821da31f71c4e2219a5b90e/FIRSTdegreeNeighbors.r)  
 [  
 FIRSTdegreeNeighbors.r  
 ](https://gist.github.com/a8dx/318ad0cb61befb3a740907e28792a6cd#file-firstdegreeneighbors-r)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>Since we’re relying on sparse matrices to hold our results, you could supply a large feature collection (e.g., 50,000+) and not worry about crashing R because you’ve exceeded memory caps that dense matrices will blow through. \[h/t [Aaron](https://stackoverflow.com/questions/4942361/how-to-turn-a-list-of-lists-to-a-sparse-matrix-in-r-without-using-lapply?rq=1)\]

The output is a symmetric matrix which can be plotted using the county ID values, where count is the number of first degree neighbor matches over some defined hexagonal size. You can control this size, which directly affects total counts per ‘bin.’ You might use this as a sanity check that only nearby features are getting picked up, but why stop there — we should plot the results and make sure it’s doing what we expect.

![NC_FirstNeighbors_SymmetricMatrix](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/01/nc_firstneighbors_symmetricmatrix.png?resize=594%2C522&ssl=1)

The code shows how easy it is to iterate the process through subsetting and merging to generate the set of second-degree neighbors. There’s some extra work to ensure that none of our first-degree neighbors appear in the list of second-degree neighbors, but that’s easy enough to do using set functions and subsets.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist85730804" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-seconddegreeneighbors-r"><div class="Box-body p-0 blob-wrapper data type-r  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | <span class="pl-smi">county.2nd</span> <span class="pl-k">&lt;-</span> merge(<span class="pl-smi">county.1st</span>, <span class="pl-smi">nc.neighbors</span>, <span class="pl-v">by.x</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>firstiterationfirstdegree<span class="pl-pds">"</span></span>, <span class="pl-v">by.y</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>county<span class="pl-pds">"</span></span>) |
|---|---|
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — find second degree neighbors that are not in first degree neighbor set </span> |
|  | <span class="pl-smi">county.2nd.only</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">county.2nd</span>\[<span class="pl-k">!</span><span class="pl-smi">county.2nd</span><span class="pl-k">$</span><span class="pl-smi">firstdegreeneighbors</span> <span class="pl-k">%in%</span> <span class="pl-smi">county.1st</span><span class="pl-k">$</span><span class="pl-smi">firstiterationfirstdegree</span>,\] |
|  | <span class="pl-smi">county.2nd.only</span> <span class="pl-k">&lt;-</span> subset(<span class="pl-smi">county.2nd.only</span>, <span class="pl-k">!</span><span class="pl-smi">firstdegreeneighbors</span> <span class="pl-k">%in%</span> <span class="pl-smi">county.of.interest</span>) |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/6d2e63a87a95b57af61ddb6b19fcf936/raw/d649cb190edea524106324478eb6b93670463a7d/SecondDegreeNeighbors.r)  
 [  
 SecondDegreeNeighbors.r  
 ](https://gist.github.com/a8dx/6d2e63a87a95b57af61ddb6b19fcf936#file-seconddegreeneighbors-r)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>Now let’s plot. If we use the basic `plot` command, we can create a relatively decent looking figure without much work, with Chatham County in purple, its first-degree neighbors in red and its second-degree neighbors in blue.  
![NthDegreeNeighborsBase](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/01/nthdegreeneighborsbase.png?resize=712%2C267&ssl=1)

Alternatively we can convert our `sf` objects to the Spatial class, and then layer them onto a Leaflet basemap. This map uses the [CartoDB positron tiling](https://rstudio.github.io/leaflet/basemaps.html), which provides a nice background, especially when visualizing land areas bordering large bodies of water.

![Chatham_NC_Neighbors](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/01/chatham_nc_neighbors.jpeg?resize=680%2C264&ssl=1)
