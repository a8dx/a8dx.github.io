---
id: 277
title: 'Stata: Union of Macros'
date: '2016-03-17T21:40:30+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=277'
permalink: /stata-union-of-macros/
image: /wp-content/uploads/2016/03/figure-8-the-ascii-text-stream-produced-when-the-binary-stream-in-is-decompressed-png.jpg
categories:
    - Computing
    - Data
    - Stata
---

![Figure-8-The-ASCII-text-stream-produced-when-the-binary-stream-in-is-decompressed.png](https://pathindependence.files.wordpress.com/2016/03/figure-8-the-ascii-text-stream-produced-when-the-binary-stream-in-is-decompressed-png.jpg?resize=508%2C196)

Wide data is ok, but I prefer long data any day of the week (at least most days of the week). Data creators/providers may disagree, and in such cases you may have to be creative about how you reshape the data.

Consider this common scenario: variable suffixes denote some index (e.g., time), but not every variable exists for each index value. For example, you can imagine Stata’s [auto](http://www.ats.ucla.edu/stat/stata/modules/usesave.htm) dataset formatted wide with fuel economy, weight, and price each indexed by years spanning 1990-2010 \[mpg90 weight90 price90 mpg91 weight91 price91 … mpg10 weight10 mpg10\]. To **reshape** this long, we need to identify all **stubs** which is easy enough in this case with 3 variables.

\[code language=”r”\]  
unab stubs: \*90  
reshape `stubs’, i(make model) j(year)  
\[/code\]

Unfortunately, variables not in the dataset for 1990 won’t enter stubs and you’ll be left with each instance of it (e.g., var91 var92 … var10) even after the reshape.

The fix is to identify all variables in the dataset with any valid index value, but to ensure you end up with a vector of unique variable names. This is relatively straightforward with a combination of **unab** and **uniq**.

The data I’m currently working with has denoted 61, 71, 81, and 91 with suffixes of 6, 7, 8, and 9 respectively. Some variables exist for 71 onwards, others only for 61. I therefore want to identify all the stubs with the following.

\[code language=”r”\]

unab stubs: \*6 \*7 \*8 \*9 // local list of all variables satisfying wildcard conditions

loc all\_vars ""  
foreach x in `stubs’ {  
loc x\_sub = substr("`x’", 1, length("`x’") – 1) // stub name is variable name less numeric suffix  
loc all\_vars "`all\_vars’" "`x\_sub’ " // ensure space follows `x\_sub’ to avoid smashing all variable names together when concatenating  
}

loc all\_vars: list uniq all\_vars // ensure stub names are not duplicated

reshape long `all\_vars’, i(district state) j(year)  
\[/code\]

Now things can go horrendously wrong if you have variables incorrectly ending in one of the stub suffixes. An easy fix is to ensure consistent endings, like renaming pertinent variables to \*\_6 instead of \*6.