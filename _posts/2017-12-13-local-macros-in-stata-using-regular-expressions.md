---
id: 1286
title: 'Local Macros in Stata Using Regular Expressions'
date: '2017-12-13T19:21:12+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=1286'
permalink: /local-macros-in-stata-using-regular-expressions/
image: /wp-content/uploads/2016/03/figure-8-the-ascii-text-stream-produced-when-the-binary-stream-in-is-decompressed-png.jpg
categories:
    - 'climate change'
    - Computing
---

Regular expressions can dramatically make your scripting simpler, more automated, and enable you to embed systematically-important information in filenames, variables, dictionaries, and paths. With enough practice, xkcd reminds us that regexp can also [make you a superhero](https://xkcd.com/208/).

Stata provides a [very nice table](https://www.stata.com/support/faqs/data-management/regular-expressions/) of their regular expressions and offers some helpful examples, but these seem more geared towards creating derivative variables, like extracting the area code from a telephone number string variable. Other objectives require a different tack.

I often want to use regular expressions to make a local that can be manipulated on the fly. Consider scanning through a vector of variable names and extracting some key feature, like a two-letter state abbreviation or a four digit year. In such a case, I don’t want to create a new variable (it’d hold the same value across all rows anyway), so the standard `gen newvar = something` approach isn’t going to apply here.

When exporting weather data, I’ll save temperature-specific values in a unique variable. For example, the total number of days in a season that a given pixel is exposed to temperatures between 23.5ºC and 24.5ºC might be named `tempExpos_24`.

We could then run something like,

```
if regexm("`x'", "([0-9][0-9])") { 
    loc degree = "`=regexs(1)'"
    }
```

to extract the temperature, and then do anything from constructing new variables that use that value, renaming the variable (e.g., converting ºC to K), or exporting to file with a meaningful name.

Another possibility doesn’t directly require an if statement, but follows immediately after a display. Given the `fn` local which takes a filename as a string, a regexp local can be made with syntax like,

```
di regexm("`fn'", "(_ppt_)([0-9][0-9][0-9][0-9])(_)([a-z0-9]*)(_)")  
    loc year = regexs(2)
```

These are departures from the more usual regexp application of generating new variables from other variables, like in this example,

```
gen dum_val = regexs(3) if regexm(parm, "(_)(tmaxpop_)([0-9]*)(_)")
```

which followed a `parmest` dump of model estimates.

And lastly, a more complete example of how multiple locals can be used in an iterative process with regular expressions. What this code snippet does is scan through a locally-defined range of variables, and constructs an aggregate degree day measure relative to an adjustable lower bound (`y`). Since this is a composite function requiring many adjacent variables, it is continuously redefining itself until it reaches an upper value, which are the dataset-specified `*_max` variables that precede the snippet.

```
// manually-identified data-specific season temperature max 
loc ERA_Kh_max = 44
loc ERA_Rb_max = 36 

loc IMD_Kh_max = 40
loc IMD_Rb_max = 34 

loc APHRO_Kh_max = 42
loc APHRO_Rb_max = 35


** Construct cumulative harmful degree day totals for a range of potential cut-off points 
foreach sea in "Kh" "Rb" {  
    foreach z in  "ERA" "IMD" "APHRO" { 
        forval y = 26(1)31 { 
            gen `z'_`sea'_`y'plus_tmeanDD_Sum = 0 
                lab var `z'_`sea'_`y'plus_tmeanDD_Sum "Cumulative degree days >= `y' for `sea' Season, using `z' Temp Data [tmean DD approach]"
            
            // collect all variables for the specified temperature range 
            unab `z'`y'plus: `z'_tmeandd_`sea'_`y' - `z'_tmeandd_`sea'_``z'_`sea'_max' 
                foreach x in ``z'`y'plus' { 
                    if regexm("`x'", "([0-9][0-9])") { 
                        loc degree = "`=regexs(1)'"
                    }
                    
                    loc degint = int(`degree')
                    replace `z'_`sea'_`y'plus_tmeanDD_Sum = `z'_`sea'_`y'plus_tmeanDD_Sum + ((`degint' - `y') *  `x')               
                } 
        } 
    }   
}   
          
```