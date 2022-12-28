---
id: 162
title: 'Merging Innumerable Tables into LaTeX? (Mac OS/X)'
date: '2015-10-01T22:54:22+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=162'
permalink: /merging-innumerable-tables-into-latex-mac-osx/
sharing_disabled:
    - '1'
image: /wp-content/uploads/2015/10/bbgro.png
categories:
    - Agriculture
    - 'climate change'
    - Computing
    - Data
    - LaTeX
---

Sometimes you simply have to run models that test dozens of different hypotheses and therefore are left with a lot of output to work through. For example, if I’m interested in how crops respond to extreme heat, there’s a bevy of specifications to work through, but more importantly, numerous crops to test. I use the standard esttab machinery in Stata to output .tex tables which can easily be inputted into a master TeX file which generates attractive enough reports to scour and share with colleagues. What I’ll describe is generic enough to accommodate a range of inputs that would be included in your TeX file, but here I’m specifically interested in having page breaks followed by individual tables, which do not adhere to a consistent file-naming convention (e.g., table1.tex, table2.tex…).

The problem I’ve had is automating a process which captures all table filenames in a directory, loops through them, and then generates that desired, clean report. Manual entry is of course an option, but then each time a table gets added/removed you have to revisit the script and that would get unmanageable very quickly when dealing with dozens (or hundreds) of files.

Here’s the alternative, and one I think which greatly reduces your chances of errors. Use a single-line awk command in Terminal to generate a file which collects object-specific LaTeX code which can then be easily added to your master TeX file.

> ls \*.tex | awk ‘{printf “\\\\input{%s}}\\n”, $1}’ &gt; inputs.tex

In this example, since we search for \*.tex, we’re presuming you’ve already cd’ed to the path with the table files. Now I wanted a couple of extra options:

1. I should be able to keep the raw table files in a separate directory to where my TeX report files are
2. I want a page break to ensure tables don’t share space.

The revised code is then:

> ls \*.tex | awk ‘{printf “\\\\clearpage{\\\\input{\\\\tablePath/%s}}\\n”, $1}’ &gt; inputs.tex

where I add the following in my LaTeX master file preamble:

> \\newcommand\*{\\tablePath}{/Users/&lt;youruserid&gt;/&lt;yourtablepath&gt;}%

The double backslash ensures that a single instance remains in creating inputs.tex. Then in your master file, all your need is:

> \\input{\\tablePath/inputs.tex}

though if you’ve moved inputs.tex (since it has full directory information for all the files in that folder), then you might not need to include the \\tablePath directory in your input command.

Compile and watch your report assimilate those dozens of raw table files that would’ve been a pain to manually generate!

And for helping this process along, definitely want to h/t the appropriate [sources](http://tex.stackexchange.com/questions/13921/inputting-multiple-files-in-latex).