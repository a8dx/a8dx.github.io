---
id: 760
title: 'Converting .TXT/.GRD Climate Data Files to netCDF Format'
date: '2017-10-08T03:49:22+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=760'
permalink: /converting-txt-grd-climate-data-to-netcdf/
image: /wp-content/uploads/2017/06/screen-shot-2017-06-02-at-9-48-19-am.png
categories:
    - 'climate change'
    - Computing
    - Data
    - India
    - Python
    - R
---

Climate data is packaged and distributed in <del>too</del> many file formats. Under ideal circumstances, you could easily convert data from formats you’re not familiar with (and don’t have scripts to handle), to those that you do. This is why analogous tools like [Stat/Transfer](http://www.stattransfer.com/) for statistical databases often used by social scientists, are so helpful. If a stranger on the street gives you SPSS data, you can on-the-fly convert it to something which is Stata-readable. Albeit, the value of software like Stat/Transfer diminishes as more stat packages have comprehensive in-built conversion tools, like R’s `<a href="https://cran.r-project.org/web/packages/readstata13/README.html" rel="noopener noreferrer" target="_blank">readstata13</a>` and `<a href="http://pandas.pydata.org/pandas-docs/version/0.20/generated/pandas.read_stata.html">read_stata</a>` in pandas. Getting similar functionality with climate data requires a bit more lift.

This post is a tutorial and link to scripts that can convert the .TXT/.GRD file combination format used by the [India Meteorological Department (IMD)](http://www.imd.gov.in/Welcome%20To%20IMD/Welcome.php) into formats that are more usable for people working with climate data. And if you’re not trained in the sciences, but rather as an economist, figuring out how to use this data often comes with its own frustrations. I hope this helps.

I rely on the [Climate Data Operators (CDO)](https://code.mpimet.mpg.de/projects/cdo) to do the heavy lifting in my climate data workflow and work almost exclusively with the [netCDF4 file format.](https://www.unidata.ucar.edu/software/netcdf/) Since IMD provides their climate data in .TXT/.GRD format, extra work is required to turn those files into more familiar formats. Here we’ll convert it to netCDF, which after processing can then be exported to a spreadsheet.

## Data Setup and Processing Steps

IMD provides you daily data for each weather variable (TMAX, TMIN, TAVG) in year-specific .TXT and .GRD files. The .TXT file includes the lat/lon grid boundaries, timestamp, and daily values for each pixel. The .GRD file somehow converts this long .TXT into an array stack.

Step 1 – We first generate year-specific .CTL files which contain the data header, since IMD provides us only a single .CTL. I found doing this in R to be relatively straightforward, since a .CTL can be read as a standard text file. Each of the generated .CTL files designate the source data and the spatial/temporal grid resolution. Here I use a modulo operator to differentiate leap years and accordingly modify the number of time-steps. If leap year date data is not included, then this wouldn’t be a concern. The following R code is an example of how those .CTLs can be auto-generated for a specified year range.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist93320359" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-convert_to_grads-r"><div class="Box-body p-0 blob-wrapper data type-r  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | <span class="pl-c"><span class="pl-c">\#</span> Filename: convert\_to\_GrADS.R </span> |
|---|---|
|  | <span class="pl-c"><span class="pl-c">\#</span> Author: Anthony Louis D'Agostino (ald at stanford dot edu)</span> |
|  | <span class="pl-c"><span class="pl-c">\#</span> Date Created: September 16, 2015</span> |
|  | <span class="pl-c"><span class="pl-c">\#</span> Last Edited: October 07, 2017</span> |
|  | <span class="pl-c"><span class="pl-c">\#</span> Purpose: Modifies the NCC-provided CTL file and generates unique versions for each year of data, each type of temperature</span> |
|  | <span class="pl-c"><span class="pl-c">\#</span> max, min, mean. </span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> Notes: While perhaps an overkill, this is generalizable to a setting where grids are file-specific. </span> |
|  |  |
|  |  |
|  | rm(<span class="pl-v">list</span> <span class="pl-k">=</span> ls()) |
|  |  |
|  |  |
|  | <span class="pl-smi">root.path</span> <span class="pl-k">&lt;-</span> <span class="pl-s"><span class="pl-pds">"</span>/tmp<span class="pl-pds">"</span></span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — create path for generated GrADS control .CTL files. </span> |
|  | <span class="pl-smi">ctl.path</span> <span class="pl-k">&lt;-</span> paste(<span class="pl-smi">root.path</span>, <span class="pl-s"><span class="pl-pds">"</span>CTL\_Files<span class="pl-pds">"</span></span>, <span class="pl-v">sep</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>/<span class="pl-pds">"</span></span>) |
|  | <span class="pl-k">if</span> (file.exists(<span class="pl-smi">ctl.path</span>) <span class="pl-k">==</span> <span class="pl-c1">FALSE</span>){ |
|  | dir.create(<span class="pl-smi">ctl.path</span>) |
|  | } |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — data stored in three separate variable-specific folders</span> |
|  | <span class="pl-v">temp\_types</span> <span class="pl-k">=</span> c(<span class="pl-s"><span class="pl-pds">"</span>MinT<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>MaxT<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>MeanT<span class="pl-pds">"</span></span>) |
|  |  |
|  | <span class="pl-k">for</span> (<span class="pl-smi">x</span> <span class="pl-k">in</span> <span class="pl-smi">temp\_types</span>) { |
|  | <span class="pl-smi">type.path</span> <span class="pl-k">&lt;-</span> paste(<span class="pl-smi">ctl.path</span>, <span class="pl-smi">x</span>, <span class="pl-v">sep</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>/<span class="pl-pds">"</span></span>) |
|  | <span class="pl-k">if</span> (file.exists(<span class="pl-smi">type.path</span>) <span class="pl-k">==</span> <span class="pl-c1">FALSE</span>){ |
|  | dir.create(<span class="pl-smi">type.path</span>) |
|  | } |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — year range for which data is available</span> |
|  | <span class="pl-k">for</span> (<span class="pl-smi">i</span> <span class="pl-k">in</span> <span class="pl-c1">1951</span><span class="pl-k">:</span><span class="pl-c1">2014</span>){ |
|  | print(paste0(<span class="pl-s"><span class="pl-pds">"</span>Now processing year <span class="pl-pds">"</span></span>, <span class="pl-smi">i</span>, <span class="pl-s"><span class="pl-pds">"</span> for variable <span class="pl-pds">"</span></span>, <span class="pl-smi">x</span>)) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — read in and update template .CTL provided by IMD</span> |
|  | <span class="pl-smi">ctl\_file</span> <span class="pl-k">&lt;-</span> read.delim(paste0(<span class="pl-smi">root.path</span>, <span class="pl-s"><span class="pl-pds">"</span>/Temp.ctl<span class="pl-pds">"</span></span>), <span class="pl-v">header</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>, <span class="pl-v">sep</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span> <span class="pl-pds">"</span></span>, <span class="pl-v">stringsAsFactors</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — file reference for source GRD </span> |
|  | <span class="pl-smi">ctl\_file</span>\[<span class="pl-c1">1</span>,<span class="pl-s"><span class="pl-pds">"</span>V2<span class="pl-pds">"</span></span>\] <span class="pl-k">&lt;-</span> paste0(<span class="pl-smi">root.path</span>, <span class="pl-s"><span class="pl-pds">"</span>/<span class="pl-pds">"</span></span>, <span class="pl-smi">x</span>, <span class="pl-s"><span class="pl-pds">"</span>/<span class="pl-pds">"</span></span>, <span class="pl-smi">x</span>, <span class="pl-s"><span class="pl-pds">"</span>\_<span class="pl-pds">"</span></span>, <span class="pl-smi">i</span>, <span class="pl-s"><span class="pl-pds">"</span>.GRD<span class="pl-pds">"</span></span>) |
|  | <span class="pl-k">if</span> (<span class="pl-smi">i</span> <span class="pl-k">%%</span> <span class="pl-c1">4</span> <span class="pl-k">==</span> <span class="pl-c1">0</span>){ |
|  | <span class="pl-smi">ctl\_file</span>\[<span class="pl-c1">7</span>,<span class="pl-s"><span class="pl-pds">"</span>V2<span class="pl-pds">"</span></span>\] <span class="pl-k">&lt;-</span> <span class="pl-c1">366</span> <span class="pl-c"><span class="pl-c">\#</span> account for leap years in total number of timesteps </span> |
|  | } <span class="pl-k">else</span> { |
|  | <span class="pl-smi">ctl\_file</span>\[<span class="pl-c1">7</span>, <span class="pl-s"><span class="pl-pds">"</span>V2<span class="pl-pds">"</span></span>\] <span class="pl-k">&lt;-</span> <span class="pl-c1">365</span> |
|  | } |
|  | <span class="pl-smi">ctl\_file</span>\[<span class="pl-c1">7</span>, <span class="pl-s"><span class="pl-pds">"</span>V5<span class="pl-pds">"</span></span>\] <span class="pl-k">&lt;-</span> paste0(<span class="pl-s"><span class="pl-pds">"</span>1JAN<span class="pl-pds">"</span></span>, <span class="pl-smi">i</span>) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">\#</span> — write updated .CTL to file</span> |
|  | write.table(<span class="pl-smi">ctl\_file</span>, paste(<span class="pl-smi">type.path</span>, paste0(<span class="pl-smi">x</span>, <span class="pl-s"><span class="pl-pds">"</span>\_<span class="pl-pds">"</span></span>, <span class="pl-smi">i</span>, <span class="pl-s"><span class="pl-pds">"</span>.ctl<span class="pl-pds">"</span></span>), <span class="pl-v">sep</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>/<span class="pl-pds">"</span></span>), <span class="pl-v">quote</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>, <span class="pl-v">col.names</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>, <span class="pl-v">na</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>, <span class="pl-v">row.names</span> <span class="pl-k">=</span> <span class="pl-c1">FALSE</span>) |
|  | } |
|  | } |
|  |  |
|  |  |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/59efac81f0414cf5c917478c60641263/raw/87706f5499987ab528a97a80b0f186152d4ca02a/convert_to_GrADS.R)  
 [  
 convert\_to\_GrADS.R  
 ](https://gist.github.com/a8dx/59efac81f0414cf5c917478c60641263#file-convert_to_grads-r)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>Step 2 – CDO enables easy conversion between GrADS and netCDF which we’ll exploit. This could be done at the command line, for reproducibility we’ll use the Python wrappers which can be installed via [pip](https://packaging.python.org/tutorials/installing-packages/). The following script demonstrates how those binaries can be read in, converted to netCDF, and concatenated by temperature variable.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist93320369" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-imd_convertrawdata-py"><div class="Box-body p-0 blob-wrapper data type-python  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | <span class="pl-c">\# Filename: IMD\_convertRawData.py</span> |
|---|---|
|  | <span class="pl-c">\# Author: Anthony Louis D'Agostino (ald at stanford dot edu)</span> |
|  | <span class="pl-c">\# Date Created: 06/01/2017 </span> |
|  | <span class="pl-c">\# Last Edited: 10/07/2017</span> |
|  | <span class="pl-c">\# Data: from NCC ZIP file </span> |
|  | <span class="pl-c">\# Purpose: Reads in GRaDS data files, concatenates them, and then exports netCDF versions </span> |
|  | <span class="pl-c">\# Notes: To be run after "convert\_to\_GrADS.R"</span> |
|  |  |
|  |  |
|  | <span class="pl-k">import</span> <span class="pl-s1">os</span> |
|  | <span class="pl-k">from</span> <span class="pl-s1">cdo</span> <span class="pl-k">import</span> <span class="pl-c1">\*</span> |
|  | <span class="pl-k">from</span> <span class="pl-s1">netCDF4</span> <span class="pl-k">import</span> <span class="pl-v">Dataset</span> |
|  | <span class="pl-k">import</span> <span class="pl-s1">numpy</span> <span class="pl-k">as</span> <span class="pl-s1">np</span> |
|  | <span class="pl-c">\#import pandas as pd </span> |
|  |  |
|  |  |
|  | <span class="pl-s1">cdo</span> <span class="pl-c1">=</span> <span class="pl-v">Cdo</span>() |
|  |  |
|  |  |
|  | <span class="pl-s1">root\_path</span> <span class="pl-c1">=</span> <span class="pl-s">"/tmp"</span> |
|  | <span class="pl-s1">ctl\_root</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">root\_path</span>, <span class="pl-s">"CTL\_Files"</span>) |
|  |  |
|  |  |
|  | <span class="pl-k">def</span> <span class="pl-en">tempOutput</span>(<span class="pl-s1">var</span>, <span class="pl-s1">ctl</span>, <span class="pl-s1">root</span>): |
|  | <span class="pl-s">"""</span> |
|  | <span class="pl-s"> Read in binary data and output as a netCDF file.</span> |
|  | <span class="pl-s"> var: weather variable </span> |
|  | <span class="pl-s"> ctl: root path for all .ctl's</span> |
|  | <span class="pl-s"> root: project root path</span> |
|  | <span class="pl-s"> """</span> |
|  |  |
|  | <span class="pl-c">\# — rename variables for consistency with other projects</span> |
|  | <span class="pl-s1">temp\_rename</span> <span class="pl-c1">=</span> {<span class="pl-s">"MaxT"</span>: <span class="pl-s">"tmax"</span>, <span class="pl-s">"MinT"</span>: <span class="pl-s">"tmin"</span>, <span class="pl-s">"MeanT"</span>: <span class="pl-s">"tmean"</span>} |
|  |  |
|  | <span class="pl-c">\# initialize using first year's data </span> |
|  | <span class="pl-s1">t</span> <span class="pl-c1">=</span> <span class="pl-s1">cdo</span>.<span class="pl-en">import\_binary</span>(<span class="pl-s1">input</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">ctl</span>, <span class="pl-s1">var</span>, <span class="pl-s1">var</span> <span class="pl-c1">+</span> <span class="pl-s">"\_1951.ctl"</span>)) |
|  |  |
|  | <span class="pl-c">\# — loop through each remaining year </span> |
|  | <span class="pl-k">for</span> <span class="pl-s1">y</span> <span class="pl-c1">in</span> <span class="pl-en">range</span>(<span class="pl-c1">1952</span>,<span class="pl-c1">2015</span>): |
|  | <span class="pl-k">print</span> <span class="pl-s">"Now processing "</span> <span class="pl-c1">+</span> <span class="pl-en">str</span>(<span class="pl-s1">y</span>) |
|  | <span class="pl-s1">fn</span> <span class="pl-c1">=</span> <span class="pl-s1">var</span> <span class="pl-c1">+</span> <span class="pl-s">"\_"</span> <span class="pl-c1">+</span> <span class="pl-en">str</span>(<span class="pl-s1">y</span>) <span class="pl-c1">+</span> <span class="pl-s">".ctl"</span> |
|  | <span class="pl-k">print</span> <span class="pl-s">"Processing "</span> <span class="pl-c1">+</span> <span class="pl-en">str</span>(<span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">ctl\_root</span>, <span class="pl-s1">var</span>, <span class="pl-s1">fn</span>)) |
|  |  |
|  | <span class="pl-c">\# — concatenate into a single file</span> |
|  | <span class="pl-s1">data</span> <span class="pl-c1">=</span> <span class="pl-s1">cdo</span>.<span class="pl-en">import\_binary</span>(<span class="pl-s1">input</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">ctl\_root</span>, <span class="pl-s1">var</span>, <span class="pl-s1">fn</span>)) |
|  | <span class="pl-s1">t</span> <span class="pl-c1">=</span> <span class="pl-s1">cdo</span>.<span class="pl-en">cat</span>(<span class="pl-s1">input</span> <span class="pl-c1">=</span> <span class="pl-s">" "</span>.<span class="pl-en">join</span>(\[<span class="pl-s1">t</span>,<span class="pl-s1">data</span>\])) |
|  |  |
|  | <span class="pl-c">\# — save variable-specific file </span> |
|  | <span class="pl-s1">t</span> <span class="pl-c1">=</span> <span class="pl-s1">cdo</span>.<span class="pl-en">copy</span>(<span class="pl-s1">input</span> <span class="pl-c1">=</span> <span class="pl-s1">t</span>, <span class="pl-s1">options</span> <span class="pl-c1">=</span> <span class="pl-s">"-f nc"</span>, <span class="pl-s1">output</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">root</span>, <span class="pl-s1">temp\_rename</span>\[<span class="pl-s1">var</span>\] <span class="pl-c1">+</span> <span class="pl-s">"Proc.nc"</span>)) |
|  |  |
|  |  |
|  | <span class="pl-en">tempOutput</span>(<span class="pl-s">"MaxT"</span>, <span class="pl-s1">ctl\_root</span>, <span class="pl-s1">root\_path</span>) |
|  | <span class="pl-en">tempOutput</span>(<span class="pl-s">"MinT"</span>, <span class="pl-s1">ctl\_root</span>, <span class="pl-s1">root\_path</span>) |
|  | <span class="pl-en">tempOutput</span>(<span class="pl-s">"MeanT"</span>, <span class="pl-s1">ctl\_root</span>, <span class="pl-s1">root\_path</span>) |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/18a88ff1c7866e62f8aedcc046306dfc/raw/b578c331f191cd0c5039d9310ad5464da777336b/IMD_convertRawData.py)  
 [  
 IMD\_convertRawData.py  
 ](https://gist.github.com/a8dx/18a88ff1c7866e62f8aedcc046306dfc#file-imd_convertrawdata-py)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>Step 3 – Now the concatenated output file has been saved as a netCDF, which means you can perform all the standard CDO operators on it. You can also simply read your files in R, and run functions like spatial averaging with a minimum of code, as in [this example](https://gis.stackexchange.com/questions/213493/area-weighted-average-raster-values-within-each-spatialpolygonsdataframe-polygon).

As always, happy to field any questions you might have on the code and workflow!