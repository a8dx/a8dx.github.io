---
id: 190
title: 'Stata: Reghdfe and factor interactions'
date: '2016-01-14T22:08:04+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=190'
permalink: /stata-reghdfe-and-factor-interactions/
geo_public:
    - '0'
image: /wp-content/uploads/2016/01/screen-shot-2016-01-14-at-5-09-25-pm.png
categories:
    - Computing
    - Data
    - econometrics
---

If you don’t know about the [reghdfe](http://www.statalist.org/forums/forum/general-stata-discussion/general/118137-new-command-reghdfe-available-on-ssc) function in Stata, you are likely missing out, especially if you run ‘high dimensional fixed effects’ models — i.e., your model includes 3+ dimensions of FE, perhaps 2 in time and 1 in space-time. I’ve been encountering a situation which raises this unhelpful error message:

`(null assertion)<br></br>Empty sample, check for missing values or an always-false if statement<br></br>r(2000);`

This is problematic because for the particular data I’m working with (less than 50K obs.), a ‘reg’ command even on 24GB of RAM would take an hour+ to run. Using reghdfe (when it worked), would give the same results in less than 1 minute.

Possible explanation? **Check to see whether the variable you’re clustering your standard errors on is a string.** If so, then use an egen/group command or convert your string to numeric and then retry — there’s a good chance it’ll work. I suppose ‘reg’ allows us to be lazy with ‘vce(cluster &lt;stringvar&gt;)’, but ‘reghdfe’ \[Homey\] “[don’t play that](https://www.youtube.com/watch?v=_QhuBIkPXn0).”