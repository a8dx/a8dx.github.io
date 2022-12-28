---
id: 1303
title: 'A Better ZIP5-County Crosswalk'
date: '2018-03-28T19:55:09+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=1303'
permalink: /a-better-zip5-county-crosswalk/
timeline_notification:
    - '1522266913'
image: /wp-content/uploads/2018/03/resordershare-1568x1140.png
categories:
    - Data
    - Geography
---

I use a healthcare expenditure dataset with observations geographically coded at the 5-digit zipcode level, but I’d also like to know which county an observation ‘belongs’ to. Maybe I want to cluster standard errors by county, or control for county-specific trends. You’d imagine this would be straightforward, but I haven’t yet found a government crosswalk that is comprehensive in all the ZIP5s that appear in my data. What follows is the best solution I’m aware of, to ensure that I match as many ZIP5s as possible. While this only increases the number of ZIP5-county matches by about 110 over what HUD offers, it’s an improvement of more than 6,000 over the Census crosswalk.

Mapping ZIP5s to counties is slightly tricky because they,

1. Refer to postal routes, and are therefore collections of lines, not polygons
2. May straddle multiple counties, necessitating an apportionment decision

The [US Census Bureau](https://www.census.gov/geo/reference/zctas.html) addresses 1.) by devising a Zip Code Tabulation Area (ZCTA) which approximates the area served in a ZIP using Census blocks. These are by and large mapped 1:1 such that, for example, the 31211 zip5 in Macon, Georgia also appears as 31211 in a ZCTA database.

As for 2.), Available crosswalks do provide several importance measures to aid the apportionment decision, under the assumption that your analysis requires that a singular county be mapped to a ZIP. HUD provides [crosswalks updated quarterly](https://www.huduser.gov/portal/datasets/usps_crosswalk.html) for several administrative levels (CBSA, CBSA division, Census tract, etc.), and [includes the share](https://www.huduser.gov/portal/datasets/usps_crosswalk.html#codebook) of that zipcode’s residential addresses, business addresses, other addresses, and total addresses that lie in any of the relevant counties.

So we’re done, right? Not yet. First, it’s not clear that an address count is ideal. Imagine a scenario where a zipcode broaches several adjacent counties and features two types of housing: senior housing and homes occupied by extended families. If the average number of residents per unit in the retirement homes is 1.5, but 6 in the latter, then the share of addresses is a very imperfect proxy for population shares.

The [Census Bureau](https://www2.census.gov/geo/docs/maps-data/data/rel/zcta_county_rel_10.txt) addresses this with their lookup table and includes 2010 Census population data as well as area (total and land-only). [Here is a helpful guide](https://www2.census.gov/geo/pdfs/maps-data/data/rel/explanation_zcta_county_rel_10.pdf) they’ve created. The crosswalk includes some relevant information in the county –&gt; ZIP5 direction as well, such as — `COPOPPCT`— the county population percentage residing in that ZIP5.

Let’s first get the [Census listing of county FIPS codes and names](https://www.census.gov/geo/reference/codes/cou.html).

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist88663164" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-gistfile1-txt"><div class="Box-body p-0 blob-wrapper data type-text  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | clear |
|---|---|
|  | clear all |
|  | clear matrix |
|  | set more off |
|  | set maxvar 25000, permanently |
|  |  |
|  |  |
|  |  |
|  | loc basePath "&lt;your path here&gt;" |
|  |  |
|  |  |
|  | // read in county codes, downloaded from: https://www.census.gov/geo/reference/codes/cou.html |
|  | import delimited using "`basePath'/national\_county.txt", delim(",") clear stringcols(\_all) varnames(nonames) |
|  |  |
|  | ren v1 state |
|  | ren v2 state\_fips |
|  | ren v3 county\_fips |
|  | ren v4 county\_name |
|  | ren v5 fipsclasscode |
|  |  |
|  | gen county = state\_fips + county\_fips |
|  |  |
|  | tempfile counties |
|  | save `counties' |
|  |  |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/20a27a330a14ba3907fff455e5994fed/raw/e80d962fb3e7efa5574c19de1e053b862521f2bb/gistfile1.txt)  
 [  
 gistfile1.txt  
 ](https://gist.github.com/a8dx/20a27a330a14ba3907fff455e5994fed#file-gistfile1-txt)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>The `county` variable is concatenated from the state FIPS and county FIPS codes, and will be used to link with the ZIP-County crosswalks.

Now let’s start with that [HUD crosswalk](https://www.huduser.gov/portal/datasets/usps_crosswalk.html). I’ve randomly selected the December 2017 version, but the following applies to any vintage. This file includes 39,455 unique zipcodes. There is substantial change over time, as an equally-randomly selected March 2011 file features 36,413 unique zipcodes. We’ll merge this in with our county FIPS file and identify the county for which a given zipcode has the largest share of total addresses, and residential addresses in.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist88663626" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-gistfile1-txt"><div class="Box-body p-0 blob-wrapper data type-text  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | import excel using "`basePath'/ZIP\_COUNTY\_122017.xlsx", firstrow clear |
|---|---|
|  |  |
|  | // 39455 unique entries before any transformations |
|  | merge m:1 county using `counties', update replace |
|  | tab \_merge |
|  | drop if \_merge == 2 |
|  | drop \_merge |
|  |  |
|  | gsort zip -tot\_ratio |
|  | bys zip: gen totOrder = \_n |
|  |  |
|  | gsort zip -res\_ratio |
|  | bys zip: gen resOrder = \_n |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/e739c69c435911fb76a3a77004782468/raw/76ef1d4601f6474a389f27ecf4c39d40ba08c23f/gistfile1.txt)  
 [  
 gistfile1.txt  
 ](https://gist.github.com/a8dx/e739c69c435911fb76a3a77004782468#file-gistfile1-txt)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>![Screen Shot 2018-03-28 at 11.52.48 AM](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/03/screen-shot-2018-03-28-at-11-52-48-am.png?resize=712%2C233&ssl=1)

The vast majority of county matches with the greatest share of residential addresses also have the highest share of total addresses. We’ll opt for the former as our apportioning variable. And to give you a sense of how important apportionment is, more than 10,000 zip5’s reside in at least two counties.

Is this problematic in any way? Let’s first inspect the residential address share for those counties that were first in the `gsort` sorting above. As expected, the vast majority of them are above 50%, with the lion’s share at 100%. These “100% zipcodes” are interior to a single county, at least in regards to residential address location. ![ResOrderShare.png](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/03/resordershare.png?resize=712%2C518&ssl=1)

What’s troubling is that mass at 0%. The sort function should not have loaded on these counties at all, yet eye-balling them suggests they are exclusively contained within single counties, so there’s not a problem. In fact, these zipcodes may simply contain only business/other addresses, in which case then they’re unlikely to further our healthcare expenditure data analysis.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist88664098" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-gistfile1-txt"><div class="Box-body p-0 blob-wrapper data type-text  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | import excel using "`basePath'/ZIP\_COUNTY\_122017.xlsx", firstrow clear |
|---|---|
|  |  |
|  | // 39455 unique entries before any transformations |
|  | merge m:1 county using `counties', update replace |
|  | tab \_merge |
|  | drop if \_merge == 2 |
|  | drop \_merge |
|  |  |
|  | gsort zip -tot\_ratio |
|  | bys zip: gen totOrder = \_n |
|  |  |
|  | gsort zip -res\_ratio |
|  | bys zip: gen resOrder = \_n |
|  |  |
|  | tw (hist res\_ratio if resOrder == 1), xtitle("Residential Address Percent") graphregion(color(white) lwidth(large)) |
|  | keep if resOrder == 1 |
|  |  |
|  | // some basic cleaning |
|  | replace county\_name = "Oglala Lakota, SD" if county == "46102" |
|  | replace state = "SD" if county == "46102" |
|  | replace state\_fips = "46" if county == "46102" |
|  | replace county\_fips = "102" if county == "46102" |
|  |  |
|  | // see http://www.nws.noaa.gov/om/notification/scn17-57kusilvak\_ak.htm |
|  | replace county\_name = "Kusilvak Census Area" if county == "02158" |
|  | replace state = "AK" if county == "02158" |
|  | replace state\_fips = "02" if county == "02158" |
|  | replace county\_fips = "158" if county == "02158" |
|  |  |
|  | drop resOrder totOrder |
|  | ren zip zip5 |
|  | destring, replace |
|  |  |
|  | tempfile zip5\_county |
|  | save `zip5\_county' |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/65d82728d11c3fa5f6c5f10ba197eb72/raw/6c19a1209b03bd9a915335a9c5261a283069bddf/gistfile1.txt)  
 [  
 gistfile1.txt  
 ](https://gist.github.com/a8dx/65d82728d11c3fa5f6c5f10ba197eb72#file-gistfile1-txt)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>We’ll now download that [Census crosswalk,](https://www2.census.gov/geo/docs/maps-data/data/rel/zcta_county_rel_10.txt) and perform some similar procedures, again apportioning to the county for which the largest share of a zipcode’s population resides.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist88664158" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-gistfile1-txt"><div class="Box-body p-0 blob-wrapper data type-text  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | // downloaded from: https://www2.census.gov/geo/docs/maps-data/data/rel/zcta\_county\_rel\_10.txt |
|---|---|
|  | import delimited using "`basePath'/zcta\_county\_rel\_10.txt", delim(",") clear stringcols(\_all) |
|  |  |
|  | \*\* 33120 unique ZCTA5 |
|  | ren state state\_fips\_rel |
|  | ren county county\_fips\_rel |
|  |  |
|  | gen county = state\_fips\_rel + county\_fips\_rel |
|  |  |
|  | merge m:1 county using `counties', update replace |
|  | tab \_merge |
|  | drop if \_merge == 2 |
|  | drop \_merge |
|  |  |
|  | destring, replace |
|  |  |
|  | ren state state\_rel |
|  | ren county county\_rel |
|  | ren county\_name county\_name\_rel |
|  |  |
|  | // apportionment: keep county matches with largest pop ZCTA5 share |
|  | gsort zcta5 -poppt |
|  | bys zcta5: gen popOrder = \_n |
|  | keep if popOrder == 1 |
|  |  |
|  | ren zpop pop\_zip5 |
|  | ren zcta5 zip5 |
|  | keep zip5 pop\_zip5 zpoppct county\_rel state\_rel county\_name\_rel state\_fips\_rel county\_fips\_rel |

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/41a46d3ca4e9d24659c7f7fda9d6e923/raw/9aec8477d87fbf51f99c96b3bd1b11fc719c3ab2/gistfile1.txt)  
 [  
 gistfile1.txt  
 ](https://gist.github.com/a8dx/41a46d3ca4e9d24659c7f7fda9d6e923#file-gistfile1-txt)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>I’ve suffixed several of the variables with “\_rel” since a naive merge with our HUD crosswalk generates several merge conflicts. We can more carefully identify disagreements between the two crosswalks this way.

We now merge and get the following:

![Screen Shot 2018-03-28 at 12.22.40 PM](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/03/screen-shot-2018-03-28-at-12-22-40-pm.png?resize=554%2C468&ssl=1)

so that there’s 33,014 zipcodes appearing in both crosswalks, while the HUD database adds 106 to those found in the Census file.

Here’s our first problem – major inconsistencies in the county results from our two apportionment procedures. ![Screen Shot 2018-03-28 at 12.26.58 PM](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/03/screen-shot-2018-03-28-at-12-26-58-pm.png?resize=657%2C425&ssl=1)

That’s pretty bad, but it’s even worse when we’re not even agreeing on the same state.

![Screen Shot 2018-03-28 at 12.29.08 PM.png](https://i0.wp.com/anthonylouisdagostino.com///wp-content/uploads/2018/03/screen-shot-2018-03-28-at-12-29-08-pm.png?resize=712%2C76&ssl=1)

In these cases, HUD tells us that 100% of residential addresses are located in the designated county, while Census gives values for these 3 of between 53% and 69%. Population apportionment would therefore direct us to the state\_rel values and respective counties.

So a final decision to make is how these disagreements should be reconciled. The good news is that this need applies only to 357 zipcodes which have conflicting information coming from both crosswalks. In many cases, county information is only coming from one of the crosswalks.

I think the optimal approach is the following and is the algorithm used in producing the following dataset:

1. Rely first on Census population-apportioned county matching
2. Then fill missing values with HUD’s residential address apportioned matches
3. When both are present but in conflict, rely on Census values which should be considered to have more integrity than HUD values derived from Census raw data

The final dataset will have 39,561 unique zipcodes, if you were to download the HUD crosswalk vintage referred to above. It marries the reliability of Census population data without the potential for errors percolating from another government agency’s analysis, with the timeliness of the quarterly HUD datasets which expands the scope of included ZIP5s.

You can [download the final Stata .dta crosswalk file here](https://www.dropbox.com/s/mycgewaljt9ic17/ZIP5_County_Crosswalk.dta?dl=0), while the [entire Stata script is available here](https://gist.github.com/a8dx/7e9d5af24101fc66aafa739577713b59).