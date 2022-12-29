---
id: 1335
title: Code
date: '2018-08-28T23:31:02+00:00'
author: admin
layout: page
guid: 'https://anthonylouisdagostino.com///?page_id=1335'
geo_public:
    - '0'
---

[My github repositories](https://github.com/a8dx)

Fuzzy Wuzzy String Matching

<figure class="wp-block-embed"><div class="wp-block-embed__wrapper"><style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist90649891" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-districtnamematching-py"><div class="Box-body p-0 blob-wrapper data type-python  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"><div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span> This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters. [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars) </span><div class="flash-action" data-view-component="true"> [ Show hidden characters ](<{{ revealButtonHref }}>)</div></div></template><template class="js-line-alert-template"> <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg></span></template> |  | <span class="pl-c">\# — DistrictNameMatching.py </span> |
|---|---|
|  | <span class="pl-c">\# Author: Anthony Louis D'Agostino (ald2187 \[at\] columbia.edu)</span> |
|  | <span class="pl-c">\# Purpose: Given CSV lists of district-state name pairs, identifies the best match given the fuzzywuzzy library</span> |
|  | <span class="pl-c">\# Notes: Default number of matches currently set to 3, though can be modified as input argument.</span> |
|  |  |
|  | <span class="pl-k">import</span> <span class="pl-s1">os</span> |
|  | <span class="pl-k">import</span> <span class="pl-s1">numpy</span> <span class="pl-k">as</span> <span class="pl-s1">np</span> |
|  | <span class="pl-k">from</span> <span class="pl-s1">fuzzywuzzy</span> <span class="pl-k">import</span> <span class="pl-s1">fuzz</span> |
|  | <span class="pl-k">from</span> <span class="pl-s1">fuzzywuzzy</span> <span class="pl-k">import</span> <span class="pl-s1">process</span> |
|  | <span class="pl-k">import</span> <span class="pl-s1">pandas</span> <span class="pl-k">as</span> <span class="pl-s1">pd</span> |
|  |  |
|  |  |
|  |  |
|  | <span class="pl-k">def</span> <span class="pl-en">districtMatch</span>(<span class="pl-s1">master</span>, <span class="pl-s1">using</span>, <span class="pl-s1">master\_dist</span>, <span class="pl-s1">master\_state</span>, <span class="pl-s1">using\_dist</span>, <span class="pl-s1">using\_state</span>, <span class="pl-s1">outFile</span>, <span class="pl-s1">num\_match</span> <span class="pl-c1">=</span> <span class="pl-c1">3</span>): |
|  | <span class="pl-s">"""</span> |
|  | <span class="pl-s"> This function takes two sets of district-state names, and produces a DTA with a set number (default=3) </span> |
|  | <span class="pl-s"> of matches with a flag for whether the district name has been completely matched. </span> |
|  | <span class="pl-s"></span> |
|  | <span class="pl-s"> Manual work is then required for districts where a perfect match has not been made. </span> |
|  | <span class="pl-s"></span> |
|  | <span class="pl-s"> master: file containing the master list of districts </span> |
|  | <span class="pl-s"> using: file containing using list of districts, eg., each of these districts is compared against these</span> |
|  | <span class="pl-s"> universe of master districts from the master file </span> |
|  | <span class="pl-s"> master\_dist: variable name pertaining to districts in master file</span> |
|  | <span class="pl-s"> master\_state: variable name pertaining to states in master file</span> |
|  | <span class="pl-s"> using\_dist: variable name pertaining to districts in using file </span> |
|  | <span class="pl-s"> using\_state: variable name pertaining to states in using file </span> |
|  | <span class="pl-s"> num\_match: number of matches generated, default is 3 </span> |
|  | <span class="pl-s"> outFile: includes path and filename for an outputted DTA file – should be "\*.dta"</span> |
|  | <span class="pl-s"> """</span> |
|  |  |
|  | <span class="pl-s1">master\_dists</span> <span class="pl-c1">=</span> <span class="pl-s1">pd</span>.<span class="pl-en">read\_csv</span>(<span class="pl-s1">master</span>, <span class="pl-s1">quotechar</span> <span class="pl-c1">=</span> <span class="pl-s">'"'</span>, <span class="pl-s1">skipinitialspace</span> <span class="pl-c1">=</span> <span class="pl-c1">True</span>, <span class="pl-s1">sep</span> <span class="pl-c1">=</span> <span class="pl-c1">None</span>) |
|  | <span class="pl-k">print</span> <span class="pl-s">" \*\*\* Now printing column values for master file \*\*\* "</span> |
|  | <span class="pl-k">print</span> <span class="pl-en">list</span>(<span class="pl-s1">master\_dists</span>.<span class="pl-s1">columns</span>.<span class="pl-s1">values</span>) |
|  |  |
|  | <span class="pl-s1">using\_dists</span> <span class="pl-c1">=</span> <span class="pl-s1">pd</span>.<span class="pl-en">read\_csv</span>(<span class="pl-s1">using</span>, <span class="pl-s1">quotechar</span> <span class="pl-c1">=</span> <span class="pl-s">'"'</span>, <span class="pl-s1">skipinitialspace</span> <span class="pl-c1">=</span> <span class="pl-c1">True</span>, <span class="pl-s1">sep</span> <span class="pl-c1">=</span> <span class="pl-c1">None</span>) |
|  | <span class="pl-k">print</span> <span class="pl-s">" \*\*\* Now printing column values for using file \*\*\* "</span> |
|  | <span class="pl-k">print</span> <span class="pl-en">list</span>(<span class="pl-s1">using\_dists</span>.<span class="pl-s1">columns</span>.<span class="pl-s1">values</span>) |
|  |  |
|  | <span class="pl-c">\# — concatenate district and state names </span> |
|  | <span class="pl-s1">master\_names</span> <span class="pl-c1">=</span> <span class="pl-s1">master\_dists</span>\[<span class="pl-s1">master\_dist</span>\].<span class="pl-en">map</span>(<span class="pl-s1">str</span>) <span class="pl-c1">+</span> <span class="pl-s">", "</span> <span class="pl-c1">+</span> <span class="pl-s1">master\_dists</span>\[<span class="pl-s1">master\_state</span>\] |
|  | <span class="pl-s1">using\_names</span> <span class="pl-c1">=</span> <span class="pl-s1">using\_dists</span>\[<span class="pl-s1">using\_dist</span>\].<span class="pl-en">map</span>(<span class="pl-s1">str</span>) <span class="pl-c1">+</span> <span class="pl-s">", "</span> <span class="pl-c1">+</span> <span class="pl-s1">using\_dists</span>\[<span class="pl-s1">using\_state</span>\] |
|  |  |
|  | <span class="pl-s1">fhp\_new</span> <span class="pl-c1">=</span> \[<span class="pl-s1">process</span>.<span class="pl-en">extract</span>(<span class="pl-s1">x</span>, <span class="pl-s1">master\_names</span>, <span class="pl-s1">limit</span><span class="pl-c1">=</span><span class="pl-s1">num\_match</span>) <span class="pl-k">for</span> <span class="pl-s1">x</span> <span class="pl-c1">in</span> <span class="pl-s1">using\_names</span>\] |
|  |  |
|  | <span class="pl-c">\# — generate column names </span> |
|  | <span class="pl-s1">lab</span> <span class="pl-c1">=</span> <span class="pl-s">""</span> |
|  | <span class="pl-s1">i</span> <span class="pl-c1">=</span> <span class="pl-c1">1</span> |
|  | <span class="pl-k">while</span> <span class="pl-s1">i</span> <span class="pl-c1">&lt;=</span> <span class="pl-s1">num\_match</span>: |
|  | <span class="pl-s1">lab</span> <span class="pl-c1">=</span> <span class="pl-s1">lab</span> <span class="pl-c1">+</span> <span class="pl-s">" "</span> <span class="pl-c1">+</span> <span class="pl-s">"Match"</span> <span class="pl-c1">+</span> <span class="pl-en">str</span>(<span class="pl-s1">i</span>) |
|  | <span class="pl-s1">i</span> <span class="pl-c1">+=</span> <span class="pl-c1">1</span> |
|  |  |
|  |  |
|  |  |
|  | <span class="pl-s1">fhp\_matches</span> <span class="pl-c1">=</span> <span class="pl-s1">pd</span>.<span class="pl-v">DataFrame</span>(<span class="pl-s1">fhp\_new</span>, <span class="pl-s1">columns</span> <span class="pl-c1">=</span> <span class="pl-s1">lab</span>.<span class="pl-en">split</span>()) |
|  |  |
|  | <span class="pl-s1">d</span><span class="pl-c1">=</span>{} |
|  | <span class="pl-k">for</span> <span class="pl-s1">x</span> <span class="pl-c1">in</span> <span class="pl-en">range</span>(<span class="pl-c1">1</span>,<span class="pl-s1">num\_match</span><span class="pl-c1">+</span><span class="pl-c1">1</span>): |
|  | <span class="pl-s1">d</span>\[<span class="pl-s">"Match{0}"</span>.<span class="pl-en">format</span>(<span class="pl-s1">x</span>)\]<span class="pl-c1">=</span>\[<span class="pl-s1">y</span>\[<span class="pl-c1">0</span>\] <span class="pl-k">for</span> <span class="pl-s1">y</span> <span class="pl-c1">in</span> <span class="pl-s1">fhp\_matches</span>\[<span class="pl-s">'Match'</span><span class="pl-c1">+</span><span class="pl-en">str</span>(<span class="pl-s1">x</span>)\]\] |
|  |  |
|  |  |
|  | <span class="pl-s1">d</span>\[<span class="pl-s">'using\_original'</span>\] <span class="pl-c1">=</span> <span class="pl-s1">using\_names</span> |
|  |  |
|  |  |
|  | <span class="pl-c">\#match1 = \[x\[0\] for x in fhp\_matches\['Match1'\]\] </span> |
|  | <span class="pl-s1">d</span>\[<span class="pl-s">'perfect\_match'</span>\] <span class="pl-c1">=</span> <span class="pl-s1">d</span>\[<span class="pl-s">'Match1'</span>\] <span class="pl-c1">==</span> <span class="pl-s1">d</span>\[<span class="pl-s">'using\_original'</span>\] |
|  |  |
|  | <span class="pl-c">\#fhp\_matches\['perfect\_match'\] = pd.Series(perfect\_match, index = fhp\_matches.index)</span> |
|  | <span class="pl-s1">out</span> <span class="pl-c1">=</span> <span class="pl-s1">pd</span>.<span class="pl-v">DataFrame</span>(<span class="pl-s1">d</span>) |
|  | <span class="pl-c">\#out.to\_stata(str(outFile + ".dta"))</span> |
|  | <span class="pl-s1">out</span>.<span class="pl-en">to\_csv</span>(<span class="pl-en">str</span>(<span class="pl-s1">outFile</span> <span class="pl-c1">+</span> <span class="pl-s">".csv"</span>)) |
|  | <span class="pl-k">print</span> <span class="pl-s">"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"</span> |
|  | <span class="pl-k">print</span> <span class="pl-s">"\*\*\* Your analysis has been completed! \*\*\* "</span> |
|  | <span class="pl-k">print</span> <span class="pl-s">"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"</span> |
|  |  |
|  | <span class="pl-k">return</span> <span class="pl-s1">out</span> |
|  |  |
|  |  |
|  | <span class="pl-s">"""</span> |
|  | <span class="pl-s">BASIC FILES/PATHS WHOSE USE IS REPEATED</span> |
|  | <span class="pl-s">"""</span> |
|  |  |
|  |  |
|  | <span class="pl-s1">baseDir</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s">"&lt;insert path&gt;"</span>) |
|  |  |
|  |  |
|  | <span class="pl-s1">outDir</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">baseDir</span>, <span class="pl-s">"Matched\_Results"</span>) |
|  |  |
|  | <span class="pl-k">if</span> <span class="pl-c1">not</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">exists</span>(<span class="pl-s1">outDir</span>): |
|  | <span class="pl-s1">os</span>.<span class="pl-en">makedirs</span>(<span class="pl-s1">outDir</span>) |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  | <span class="pl-s">"""</span> |
|  | <span class="pl-s">ICRISAT and 1971 Polygon borders </span> |
|  | <span class="pl-s">"""</span> |
|  |  |
|  | <span class="pl-s1">master\_file</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">baseDir</span>, <span class="pl-s">"ICRISAT\_Meso\_Names.csv"</span>) |
|  | <span class="pl-s1">input\_1971</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">baseDir</span>,<span class="pl-s">"District\_Records\_1971\_Clean.csv"</span>) |
|  |  |
|  | <span class="pl-s1">outFile</span> <span class="pl-c1">=</span> <span class="pl-s1">os</span>.<span class="pl-s1">path</span>.<span class="pl-en">join</span>(<span class="pl-s1">outDir</span>, <span class="pl-s">"ICRI\_1971\_DistrictMatches"</span>) |
|  | <span class="pl-s1">icri\_1971\_match</span> <span class="pl-c1">=</span> <span class="pl-en">districtMatch</span>(<span class="pl-s1">input\_1971</span>, <span class="pl-s1">master\_file</span>, <span class="pl-s">'NAME'</span>, <span class="pl-s">'STATE\_UT'</span>, <span class="pl-s">'distname'</span>, <span class="pl-s">'stname'</span>, <span class="pl-s1">outFile</span>) |
|  |  |
|  | <span class="pl-c">\# — alternatively, don't save as a workspace object</span> |
|  | <span class="pl-en">districtMatch</span>(<span class="pl-s1">input\_1971</span>, <span class="pl-s1">master\_file</span>, <span class="pl-s">'NAME'</span>, <span class="pl-s">'STATE\_UT'</span>, <span class="pl-s">'distname'</span>, <span class="pl-s">'stname'</span>, <span class="pl-s1">outFile</span>) |
|  |  |

</div> </div> </div></div> </div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/20caedaf942f1810e16994bfdb57e8dc/raw/ec412554285823b873e37d523142082d79cd36d5/DistrictNameMatching.py) [ DistrictNameMatching.py ](https://gist.github.com/a8dx/20caedaf942f1810e16994bfdb57e8dc#file-districtnamematching-py) hosted with ❤ by [GitHub](https://github.com) </div> </div></div></div></figure>ZIP5 – County Lookup

<figure class="wp-block-embed"><div class="wp-block-embed__wrapper"><style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist88664726" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-zip5_county_lookup-do"><div class="Box-body p-0 blob-wrapper data type-stata  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"><div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span> This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters. [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars) </span><div class="flash-action" data-view-component="true"> [ Show hidden characters ](<{{ revealButtonHref }}>)</div></div></template><template class="js-line-alert-template"> <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg></span></template> |  | <span class="pl-c"><span class="pl-c">/\*</span></span> |
|---|---|
|  | <span class="pl-c">Filename: ZIP5\_County\_Lookup.do</span> |
|  | <span class="pl-c">Author: Anthony D'Agostino (ald at satanford dot edu) </span> |
|  | <span class="pl-c">Date Created: 02/09/2018</span> |
|  | <span class="pl-c">Last Edited: 03/27/2018</span> |
|  | <span class="pl-c">Purpose: Generate a merged zip5-county crosswalk that leverages the strengths of both the US Census and HUD ZCTA crosswalks. </span> |
|  | <span class="pl-c"></span> |
|  | <span class="pl-c"><span class="pl-c">\*/</span></span> |
|  |  |
|  |  |
|  | clear |
|  | clear all |
|  | clear matrix |
|  | set more off |
|  | set maxvar 25000, permanently |
|  |  |
|  |  |
|  |  |
|  | <span class="pl-k">loc</span> basePath <span class="pl-s"><span class="pl-pds">"</span>&lt;your path here&gt;<span class="pl-pds">"</span></span> |
|  |  |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// read in county codes, downloaded from: https://www.census.gov/geo/reference/codes/cou.html </span></span> |
|  | import delimited using <span class="pl-s"><span class="pl-pds">"</span>`basePath'/national\_county.txt<span class="pl-pds">"</span></span>, delim(<span class="pl-s"><span class="pl-pds">"</span>,<span class="pl-pds">"</span></span>) clear stringcols(\_all) varnames(nonames) |
|  |  |
|  | ren v1 state |
|  | ren v2 state\_fips |
|  | ren v3 county\_fips |
|  | ren v4 county\_name |
|  | ren v5 fipsclasscode |
|  |  |
|  | <span class="pl-k">gen</span> county <span class="pl-k">=</span> state\_fips <span class="pl-k">+</span> county\_fips |
|  |  |
|  | <span class="pl-k">tempfile</span> counties |
|  | save `counties' |
|  |  |
|  | <span class="pl-c"><span class="pl-c"> \*import excel using "`basePath'/ZIP\_COUNTY\_032011.xlsx", firstrow clear </span></span> |
|  | import excel using <span class="pl-s"><span class="pl-pds">"</span>`basePath'/ZIP\_COUNTY\_122017.xlsx<span class="pl-pds">"</span></span>, firstrow clear |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// 39455 unique entries before any transformations</span></span> |
|  | merge m:1 county using `counties', update replace |
|  | tab \_merge |
|  | drop if \_merge <span class="pl-k">==</span> 2 |
|  | drop \_merge |
|  |  |
|  | gsort zip <span class="pl-k">–</span>tot\_ratio |
|  | <span class="pl-k">by<span class="pl-k">s</span></span> zip: <span class="pl-k">gen</span> totOrder <span class="pl-k">=</span> \_n |
|  |  |
|  | gsort zip <span class="pl-k">–</span>res\_ratio |
|  | <span class="pl-k">by<span class="pl-k">s</span></span> zip: <span class="pl-k">gen</span> resOrder <span class="pl-k">=</span> \_n |
|  |  |
|  | tw (hist res\_ratio if resOrder <span class="pl-k">==</span> 1), xtitle(<span class="pl-s"><span class="pl-pds">"</span>Residential Address Percent<span class="pl-pds">"</span></span>) graphregion(color(white) lwidth(large)) |
|  | keep if resOrder <span class="pl-k">==</span> 1 |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// some basic cleaning </span></span> |
|  | <span class="pl-k">replace</span> county\_name <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>Oglala Lakota, SD<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>46102<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> state <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>SD<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>46102<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> state\_fips <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>46<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>46102<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> county\_fips <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>102<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>46102<span class="pl-pds">"</span></span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// see http://www.nws.noaa.gov/om/notification/scn17-57kusilvak\_ak.htm </span></span> |
|  | <span class="pl-k">replace</span> county\_name <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>Kusilvak Census Area<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>02158<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> state <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>AK<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>02158<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> state\_fips <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>02<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>02158<span class="pl-pds">"</span></span> |
|  | <span class="pl-k">replace</span> county\_fips <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>158<span class="pl-pds">"</span></span> if county <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span>02158<span class="pl-pds">"</span></span> |
|  |  |
|  | drop resOrder totOrder |
|  | ren zip zip5 |
|  | destring, replace |
|  |  |
|  | <span class="pl-k">tempfile</span> zip5\_county |
|  | save `zip5\_county' |
|  |  |
|  |  |
|  | import delimited using <span class="pl-s"><span class="pl-pds">"</span>`basePath'/zcta\_county\_rel\_10.txt<span class="pl-pds">"</span></span>, delim(<span class="pl-s"><span class="pl-pds">"</span>,<span class="pl-pds">"</span></span>) clear stringcols(\_all) <span class="pl-c"><span class="pl-c">// varnames(nonames) </span></span> |
|  |  |
|  | <span class="pl-c"><span class="pl-c"> \*\* 33120 unique ZCTA5 </span></span> |
|  | ren state state\_fips\_rel |
|  | ren county county\_fips\_rel |
|  |  |
|  | <span class="pl-k">gen</span> county <span class="pl-k">=</span> state\_fips\_rel <span class="pl-k">+</span> county\_fips\_rel |
|  |  |
|  | merge m:1 county using `counties', update replace |
|  | tab \_merge |
|  | drop if \_merge <span class="pl-k">==</span> 2 |
|  | drop \_merge |
|  |  |
|  | destring, replace |
|  |  |
|  | ren state state\_rel |
|  | ren county county\_rel |
|  | ren county\_name county\_name\_rel |
|  | <span class="pl-k">gen</span> popabove50 <span class="pl-k">=</span> cond(zpoppct <span class="pl-k">&gt;</span><span class="pl-k">=</span> 50, 1, 0) |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// apportionment: keep county matches with largest pop ZCTA5 share </span></span> |
|  |  |
|  | gsort zcta5 <span class="pl-k">–</span>poppt |
|  | <span class="pl-k">by<span class="pl-k">s</span></span> zcta5: <span class="pl-k">gen</span> popOrder <span class="pl-k">=</span> \_n |
|  | keep if popOrder <span class="pl-k">==</span> 1 |
|  |  |
|  | ren zpop pop\_zip5 |
|  | ren zcta5 zip5 |
|  | keep zip5 pop\_zip5 zpoppct county\_rel state\_rel county\_name\_rel state\_fips\_rel county\_fips\_rel |
|  |  |
|  |  |
|  | merge 1:1 zip5 using `zip5\_county', update replace |
|  | tab \_merge |
|  | drop \_merge |
|  |  |
|  |  |
|  | <span class="pl-c"><span class="pl-c">// prioritize Census values, replace with HUD where missing </span></span> |
|  | <span class="pl-k">foreach</span> x <span class="pl-k">in</span> <span class="pl-s"><span class="pl-pds">"</span>county<span class="pl-pds">"</span></span> <span class="pl-s"><span class="pl-pds">"</span>state\_fips<span class="pl-pds">"</span></span> <span class="pl-s"><span class="pl-pds">"</span>county\_fips<span class="pl-pds">"</span></span> { |
|  | <span class="pl-k">gen</span> `x'\_final <span class="pl-k">=</span> `x'\_rel |
|  | <span class="pl-k">replace</span> `x'\_final <span class="pl-k">=</span> `x' if `x'\_final <span class="pl-k">==</span> . |
|  | drop `x' |
|  | ren `x'\_final `x' |
|  | } |
|  |  |
|  | <span class="pl-k">foreach</span> x <span class="pl-k">in</span> <span class="pl-s"><span class="pl-pds">"</span>county\_name<span class="pl-pds">"</span></span> <span class="pl-s"><span class="pl-pds">"</span>state<span class="pl-pds">"</span></span> { |
|  | <span class="pl-k">gen</span> `x'\_final <span class="pl-k">=</span> `x'\_rel |
|  | <span class="pl-k">replace</span> `x'\_final <span class="pl-k">=</span> `x' if `x'\_final <span class="pl-k">==</span> <span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span> |
|  | drop `x' |
|  | ren `x'\_final `x' |
|  | } |
|  |  |
|  | keep zip5 county county\_fips county\_name state state\_fips |
|  | save <span class="pl-s"><span class="pl-pds">"</span>`basePath'/ZIP5\_County\_Crosswalk<span class="pl-pds">"</span></span>, replace |

</div> </div> </div></div> </div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/7e9d5af24101fc66aafa739577713b59/raw/bf4f4c31cf565aa920dd6a10a4522f8e5708ea40/ZIP5_County_Lookup.do) [ ZIP5\_County\_Lookup.do ](https://gist.github.com/a8dx/7e9d5af24101fc66aafa739577713b59#file-zip5_county_lookup-do) hosted with ❤ by [GitHub](https://github.com) </div> </div></div></div></figure>
