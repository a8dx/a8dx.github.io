---
id: 173
title: 'Tutorial: FuzzyWuzzy String Matching in Python &#8211; Improving Merge Accuracy Across Data Products and Naming Conventions'
date: '2015-10-31T19:18:20+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=173'
permalink: /tutorial-fuzzywuzzy-string-matching-in-python-improving-merge-accuracy-across-data-products-and-naming-conventions/
sharing_disabled:
    - '1'
geo_public:
    - '0'
image: /wp-content/uploads/2015/10/screen-shot-2015-10-31-at-1-31-27-pm.png
categories:
    - Computing
    - Data
    - Python
---

<figure aria-describedby="caption-attachment-175" class="wp-caption aligncenter" id="attachment_175" style="width: 660px">[![District Naming Preview](https://pathindependence.files.wordpress.com/2015/10/screen-shot-2015-10-31-at-1-41-49-pm.png?w=660&resize=660%2C322)](https://pathindependence.files.wordpress.com/2015/10/screen-shot-2015-10-31-at-1-41-49-pm.png)<figcaption class="wp-caption-text" id="caption-attachment-175">Example of Two Datasets with Comparable Variables</figcaption></figure>

If you work with manually-entered string character data or data coming from multiple providers, you may encounter the reality of not being able to a.) merge the data, or b.) produce correct summary statistics. Regarding a.), take the example in the picture of Indian district names exported from two data sources — we’d have a high success rate if we tried to merge on the existing names, but would miss out on Prakasam (Source B) which is labeled Ongole (Prakasam) in Source A. This level of success is pretty magical, but not realistic – I’ve encountered situations where maybe 30% of the raw data merge correctly. This is bad for power and for representativeness. In Stata you might be satisfyced with 85+% \_merge == 3, but we should be striving for 100% since the dropped 15% could be dramatically different in composition than our 85% subsample.

Similarly, you may want to map your data using a related shapefile. If the administrative names (or associated identifier values) don’t match, you’ll have an incomplete map. The solution is to export the Attribute Table from your \*.SHP and use this as one of the input spreadsheets into the script I describe below.

As per b.), I recently opened a file where workers self-reported their primary occupation. Entries that *should’ve been* “construction” spanned “CONSTRUCTION”, “contruction”, “ccontrucion”, “construct”, “construciton.” You get the idea. If you wanted to calculate the share of workers by occupation, you could either manually edit each entry or adopt a FuzzyWuzzy-based approach described here. While there are other methods to achieve similar results, this is the workflow that has worked for me and I hope will also work for you.

[FuzzyWuzzy](https://github.com/seatgeek/fuzzywuzzy) is a fantastic Python package which uses a distance matching algorithm to calculate proximity measures between string entries. “CONSTRUCTION” and “CONSTRUCTION” would yield a 100% match, while “CONSTRUCTION” and “CANSTRICTION” would generate a lower score. FuzzyWuzzy can be installed via pip or other Python package management tools.

Take the Indian districts example with two distinct datasets each possessing unique entries. I would ideally like to match District D in Dataset B (e.g., Prakasam, Andhra Pradesh) to the best match in Dataset B. FuzzyWuzzy will generate those matching scores and provide you with N (user-selected) entries having the highest score. If all  
goes well, the entry with the highest score will indeed be our match, but we need to verify that first.

I’ve included a [sample Python script](https://www.dropbox.com/s/e9k5384k9mo1u2c/Fuzzy_Wuzzy.zip?dl=0) that’s worked for my purposes and can be easily retrofitted for a range of applications. I’ll briefly describe the steps I take from start to end.

1. Take two datasets and export the variables you wish to merge as distinct spreadsheets. In this example, I am interested in merging the names of district-state pairs from “ICRISAT\_Meso\_Names.csv” and “District\_Records\_1971\_Clean.csv.” They need not possess the same variable names since the function &lt;districtMatch&gt; takes as input arguments the district and state variable names from either input file. The former is a complete dataset, the latter a list of administrative names for a 1971 shapefile. Once these two are merged, I’ll be able to project variable values into a map.
2. I run the &lt;districtMatch&gt; function on these two files which generates “ICRI\_1971\_DistrictMatches.csv” into /Matched\_Results. The function’s default is to output the top 3 matches, but you can edit this.
3. If there’s a perfect match, the “perfect\_match” field in the DistrictMatches file is given a “True” value. You need not spend any more time on those entries since they will merge correctly. You’ll only need to focus on “False” matches and will need to do this manually. To save time, you may even sort on “perfect\_match”, rather than hunt-and-peck for Falses.
4. To address the False matches, I generate two new variables in ICRI\_1971\_DistrictMatches.csv and save it as an XLSX file to retain highlighting/cell formatting. 
    1. “best\_imperfect\_match” – identifies which match column is in fact correct. (i.e., 2 refers to ‘match2’, 3 to ‘match3’, etc.)
    2. “explicit\_match” – in the event the match is not captured in the top 3 match hits, I’ll have to manually identify the correct match and type it in.
5. Once I’ve made all the necessary edits, I can then import this XLSX file into Stata and run some basic code to get the two datasets merge properly. I provide example code in this [Stata “fuzzywuzzy\_Merge\_Sample.do”](https://www.dropbox.com/s/e9k5384k9mo1u2c/Fuzzy_Wuzzy.zip?dl=0).

If you’re interested in merging on a single variable (i.e., state names, without districts), you can easily modify the function to include only a single variable input name from each dataset and skip the joining of district and state names.

<style>.gist table { margin-bottom: 0; }</style><div class="gist" id="gist90649891" style="tab-size: 8"><div class="gist-file" translate="no"><div class="gist-data"><div class="js-gist-file-update-container js-task-list-container file-box"><div class="file my-2" id="file-districtnamematching-py"><div class="Box-body p-0 blob-wrapper data type-python  " itemprop="text"><div class="js-check-bidi js-blob-code-container blob-code-content"> <template class="js-file-alert-template"></template>

<div class="flash flash-warn flash-full d-flex flex-items-center" data-view-component="true"> <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg> <span>  
 This file contains bidirectional Unicode text that may be interpreted or compiled differently than what appears below. To review, open the file in an editor that reveals hidden Unicode characters.  
 [Learn more about bidirectional Unicode characters](https://github.co/hiddenchars)  
 </span>

<div class="flash-action" data-view-component="true"> [ Show hidden characters  ](<{{ revealButtonHref }}>)</div></div>  
<template class="js-line-alert-template">  
 <span aria-label="This line has hidden Unicode characters" class="line-alert tooltipped tooltipped-e" data-view-component="true">  
 <svg aria-hidden="true" class="octicon octicon-alert" data-view-component="true" height="16" version="1.1" viewbox="0 0 16 16" width="16"> <path d="M8.22 1.754a.25.25 0 00-.44 0L1.698 13.132a.25.25 0 00.22.368h12.164a.25.25 0 00.22-.368L8.22 1.754zm-1.763-.707c.659-1.234 2.427-1.234 3.086 0l6.082 11.378A1.75 1.75 0 0114.082 15H1.918a1.75 1.75 0 01-1.543-2.575L6.457 1.047zM9 11a1 1 0 11-2 0 1 1 0 012 0zm-.25-5.25a.75.75 0 00-1.5 0v2.5a.75.75 0 001.5 0v-2.5z" fill-rule="evenodd"></path></svg>  
</span></template>

|  | <span class="pl-c">\# — DistrictNameMatching.py </span> |
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

</div></div></div></div></div><div class="gist-meta"> [view raw](https://gist.github.com/a8dx/20caedaf942f1810e16994bfdb57e8dc/raw/ec412554285823b873e37d523142082d79cd36d5/DistrictNameMatching.py)  
 [  
 DistrictNameMatching.py  
 ](https://gist.github.com/a8dx/20caedaf942f1810e16994bfdb57e8dc#file-districtnamematching-py)  
 hosted with ❤ by [GitHub](https://github.com) </div></div></div>