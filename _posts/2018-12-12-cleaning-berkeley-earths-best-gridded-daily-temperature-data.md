---
id: 1366
title: "Cleaning Berkeley Earth's BEST Gridded Daily Temperature Data"
date: '2018-12-12T09:33:41+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=1366'
permalink: /cleaning-berkeley-earths-best-gridded-daily-temperature-data/
timeline_notification:
    - '1544607224'
image: /wp-content/uploads/2018/12/levelplot_example-1568x1040.png
categories:
    - 'climate change'
    - Data
    - R
---

You may have recently seen air quality maps produced by the [Berkeley Earth](http://berkeleyearth.org/) group, especially in the wake of the [horrific Camp Fire](https://www.sfgate.com/california-wildfires/article/Camp-Fire-Death-toll-rises-to-86-after-13458956.php) whose death toll now exceeds 80. For example, [here’s](http://berkeleyearth.org/air-quality-real-time-map/) their real-time visualization of PM2.5 concentrations.

 For several years, the Berkeley Earth team had primarily worked on producing gridded weather datasets with solid historical coverage, going back to 1850 in some cases. I’ve been particularly interested in their work given the [global daily temperature datasets they have on tap](http://berkeleyearth.org/data/) – which are publicly available! This is all very exciting, given that many of the global datasets available, like [Willmott and Matsuura (UDEL)](https://www.esrl.noaa.gov/psd/data/gridded/data.UDel_AirT_Precip.html) and [CRU](https://crudata.uea.ac.uk/cru/data/hrg/), are only at monthly resolution. However, there’s something to be desired in making their offerings truly accessible to researchers:

- Their netCDF files are packaged with “temperature anomaly” and “climatology” layers, but no “temperature” layer. This necessitates a bit of wrangling on your part to overcome some inherent dimensional consistency issues to create a temperature object. It would’ve struck me to provide the temperature values, and allow researchers to determine which climatology period they want to construct, not the reverse.
- For example, the climatology is a 365-day stack. Great, except that the anomalies data is based on true dates and includes leap years. There’s not an immediate way to sum the two into temperature estimates. If you ignore the mismatch and are performing analysis that spans many decades, by the end of your time-series you’ll be off by several weeks (amateurish!).
- The .nc’s are stripped of an informative time axis (see below figure). Instead, the files have date\_number, day, month, and year attributes. Why couldn’t they have provided an out of the box date object, and left it to researchers to parse the month and year when necessary?
- As a result, you wouldn’t even know whether leap dates are included unless you manually inspected the layer count for a decadal file.
- And lastly, the absence of that time axis means you’re not able to discern dates when visually inspecting the data in a viewer like [Panoply](https://www.giss.nasa.gov/tools/panoply/download/). If you wanted to see what the temperature \[anomalies\] were for Jan 12, 2013, then you’ll have to manually count how many days passed since the Jan 1, 2010 start date of that decadal extract.

<figure class="wp-block-image">![](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/12/best_data.png?resize=712%2C520&ssl=1)<figcaption>BEST temperature data does not come packaged with a useful time axis  
</figcaption></figure>I’ve spent a fair amount of time trying to address these issues, and wanted to share `R` code so that you don’t have to recreate the process. The only packages you’ll need to run this are `raster` and `tidyverse`.

In brief, this function 1.) reads in a BEST .nc file, which you may or may not have previously spatial subsetted, 2.) creates a stacked climatology that properly accounts for leap years by duplicating Feb 28 climatology values for Feb 29, and 3.) outputs temperature estimates with a date explicit axis.

<figure class="wp-block-embed is-type-rich"><div class="wp-block-embed__wrapper"><style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist93531237" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-clean_best_data-r"><div class="Box-body p-0 blob-wrapper data type-r  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"><div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span> This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters. [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars) </span><div class="flash-action" data-view-component="true"> [ Show hidden characters ](<{{ revealButtonHref }}>)</div></div></template><template class="js-line-alert-template"> <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg></span></template> |  |  |
|---|---|
|  | <span class="pl-en">returnBESTtemp</span> <span class="pl-k">&lt;-</span> <span class="pl-k">function</span>(<span class="pl-smi">tempFile</span>) { |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> tempFile: complete path to a netcdf BEST temperature file of daily records \[need not be geographic subset\]</span> |
|  |  |
|  | <span class="pl-smi">ave</span> <span class="pl-k">&lt;-</span> brick(<span class="pl-smi">tempFile</span>, <span class="pl-v">var</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>climatology<span class="pl-pds">"</span></span>) |
|  | <span class="pl-smi">anomalies</span> <span class="pl-k">&lt;-</span> brick(<span class="pl-smi">tempFile</span>, <span class="pl-v">var</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>temperature<span class="pl-pds">"</span></span>) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — deal with absence of leap year in anomaly series</span> |
|  | <span class="pl-smi">ave.dates</span> <span class="pl-k">&lt;-</span> as\_date(as.numeric(unlist(lapply(strsplit(names(<span class="pl-smi">ave</span>), <span class="pl-s"><span class="pl-pds">"</span>X<span class="pl-pds">"</span></span>), <span class="pl-k">function</span>(<span class="pl-smi">x</span>) <span class="pl-smi">x</span>\[\[<span class="pl-c1">2</span>\]\]))), <span class="pl-v">origin</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>1949-12-31<span class="pl-pds">"</span></span>) |
|  | which(<span class="pl-smi">ave.dates</span> <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>1950-02-28<span class="pl-pds">"</span></span>) <span class="pl-c"><span class="pl-c">\#</span> 59th doy</span> |
|  |  |
|  | <span class="pl-smi">leap.ave</span> <span class="pl-k">&lt;-</span> stack(<span class="pl-smi">ave</span>\[\[<span class="pl-c1">1</span><span class="pl-k">:</span><span class="pl-c1">59</span>\]\], <span class="pl-smi">ave</span>\[\[<span class="pl-c1">59</span>\]\], <span class="pl-smi">ave</span>\[\[<span class="pl-c1">60</span><span class="pl-k">:</span><span class="pl-c1">365</span>\]\]) <span class="pl-c"><span class="pl-c">\#</span> leap year version duplicates climatology from feb 28 for leap day</span> |
|  | <span class="pl-smi">leap.ave.bd</span> <span class="pl-k">&lt;-</span> format(as\_date(<span class="pl-c1">1</span><span class="pl-k">:</span><span class="pl-c1">366</span>, <span class="pl-v">origin</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>1951-12-31<span class="pl-pds">"</span></span>), <span class="pl-v">format</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>%b-%d<span class="pl-pds">"</span></span>) |
|  |  |
|  | <span class="pl-smi">clima</span> <span class="pl-k">&lt;-</span> stack(<span class="pl-smi">ave</span>, <span class="pl-smi">ave</span>, <span class="pl-smi">leap.ave</span>, <span class="pl-smi">ave</span>) <span class="pl-c"><span class="pl-c">\#</span> since starting with 1950, non-leap, non-leap, leap, non-leap, and then recycle. </span> |
|  | <span class="pl-smi">clima.fullsize</span> <span class="pl-k">&lt;-</span> stack(replicate(<span class="pl-c1">17</span>, <span class="pl-smi">clima</span>)) <span class="pl-c"><span class="pl-c">\#</span> 17 iterations of this 4 year sequence, now comparable in dimensions to anomalies object </span> |
|  |  |
|  | <span class="pl-smi">dates</span> <span class="pl-k">&lt;-</span> as\_date(as.numeric(unlist(lapply(strsplit(names(<span class="pl-smi">anomalies</span>), <span class="pl-s"><span class="pl-pds">"</span><span class="pl-cce">\\\\</span>.<span class="pl-pds">"</span></span>), <span class="pl-k">function</span>(<span class="pl-smi">x</span>) <span class="pl-smi">x</span>\[\[<span class="pl-c1">2</span>\]\]))), <span class="pl-v">origin</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>1949-12-31<span class="pl-pds">"</span></span>) |
|  | <span class="pl-smi">years</span> <span class="pl-k">&lt;-</span> unique(year(<span class="pl-smi">dates</span>)) |
|  | <span class="pl-smi">month.day</span> <span class="pl-k">&lt;-</span> format(<span class="pl-smi">dates</span>, <span class="pl-v">format</span><span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">"</span>%b-%d<span class="pl-pds">"</span></span>) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — drop 2018, which is problematic because it's only a partial year</span> |
|  | <span class="pl-smi">anomalies</span> <span class="pl-k">&lt;-</span> subset(<span class="pl-smi">anomalies</span>, which(<span class="pl-k">!</span>year(<span class="pl-smi">dates</span>) <span class="pl-k">%in%</span> <span class="pl-c1">2018</span>), <span class="pl-v">drop</span> <span class="pl-k">=</span> <span class="pl-c1">T</span>) |
|  | <span class="pl-smi">dates</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">dates</span>\[<span class="pl-k">!</span>year(<span class="pl-smi">dates</span>) <span class="pl-k">%in%</span> <span class="pl-c1">2018</span>\] |
|  |  |
|  | print(length(<span class="pl-smi">dates</span>) <span class="pl-k">==</span> dim(<span class="pl-smi">anomalies</span>)\[<span class="pl-c1">3</span>\]) |
|  | <span class="pl-smi">temp</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">anomalies</span> <span class="pl-k">+</span> <span class="pl-smi">clima</span> <span class="pl-c"><span class="pl-c">\#</span> recycles to length of anomalies </span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — perform spot checks to ensure values are properly summed </span> |
|  | <span class="pl-smi">test.layers</span> <span class="pl-k">&lt;-</span> floor(runif(<span class="pl-c1">20</span>) <span class="pl-k">\*</span> dim(<span class="pl-smi">anomalies</span>)\[<span class="pl-c1">3</span>\]) %<span class="pl-k">&gt;</span>% unique() |
|  | <span class="pl-smi">anomalies.test</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">anomalies</span>\[\[<span class="pl-smi">test.layers</span>\]\] |
|  | <span class="pl-smi">monthday.test</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">month.day</span>\[<span class="pl-smi">test.layers</span>\] |
|  | <span class="pl-smi">indices</span> <span class="pl-k">&lt;-</span> unlist(lapply(<span class="pl-smi">monthday.test</span>, <span class="pl-k">function</span>(<span class="pl-smi">x</span>) which(<span class="pl-smi">leap.ave.bd</span> <span class="pl-k">==</span> <span class="pl-smi">x</span>))) <span class="pl-c"><span class="pl-c">\#</span> julian day values </span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — these all check out — satisfied it's taking the correct sums — # </span> |
|  | <span class="pl-smi">sum.random</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">anomalies</span>\[\[<span class="pl-smi">test.layers</span>\]\] <span class="pl-k">+</span> <span class="pl-smi">leap.ave</span>\[\[<span class="pl-smi">indices</span>\]\] <span class="pl-c"><span class="pl-c">\#</span> manual combination of anomalies layer and climatology</span> |
|  | <span class="pl-smi">true.values</span> <span class="pl-k">&lt;-</span> <span class="pl-smi">temp</span>\[\[<span class="pl-smi">test.layers</span>\]\] <span class="pl-c"><span class="pl-c">\#</span> our combined temperature values layer </span> |
|  | min(<span class="pl-smi">true.values</span> <span class="pl-k">==</span> <span class="pl-smi">sum.random</span>) |
|  |  |
|  |  |
|  | <span class="pl-smi">temp</span> <span class="pl-k">&lt;-</span> setZ(<span class="pl-smi">temp</span>, <span class="pl-smi">dates</span>) |
|  | names(<span class="pl-smi">temp</span>) <span class="pl-k">&lt;-</span> as.character(<span class="pl-smi">dates</span>) |
|  |  |
|  | <span class="pl-smi">temp</span> |
|  | } |

</div> </div> </div></div> </div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/acfb8181ebcdb5375c4d085064f22029/raw/3c166b63e2016e80412e3e440fa3222ef127dfed/clean_BEST_data.r) [ clean\_BEST\_data.r ](https://gist.github.com/a8dx/acfb8181ebcdb5375c4d085064f22029#file-clean_best_data-r) hosted with ❤ by [GitHub](https://github.com) </div> </div></div></div></figure>You’ll notice some hard coded elements, such as the origin date and whether to drop partial years (since BEST is continuously updating their data, a portion of the current year will always be included in the most recent decadal extract). I’ve also included some tests to convince myself that the summed product mirrors values obtained from a manually searched climatology layer combined with the anomalies layer for a specified date.
