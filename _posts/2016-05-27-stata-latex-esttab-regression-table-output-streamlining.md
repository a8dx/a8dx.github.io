---
id: 353
title: 'Stata-Latex esttab Regression Table Output Streamlining'
date: '2016-05-27T21:36:45+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=353'
permalink: /stata-latex-esttab-regression-table-output-streamlining/
image: /wp-content/uploads/2016/05/screen-shot-2016-05-27-at-6-13-42-pm.png
categories:
    - Computing
    - Data
    - econometrics
    - LaTeX
---

Researchers spend an excessive amount of time getting up to speed with a field’s chosen tools and methods, excessive because there is often a consensus on best practice and yet those best practices are not made common knowledge. I think the CS and statistics communities have this right in their pushing for open data, transparency, and reproducibility in a way that economics, for example, has been late to the game on. As a result, early-stage PhD students can emulate and save those wasted hours tinkering with multicolumns in Latex or some user unfriendly Stata syntax. I have personally benefited greatly from the likes of [Jorg Weber](http://www.jwe.cc/2012/03/stata-latex-tables-estout/) and [UCLA IDRE](http://www.ats.ucla.edu/stat/stata/faq/estout.htm), among the numerous Stack Overflow posts on publishing regression output, and have finally developed a satisfycing Stata-Latex [esttab](http://repec.org/bocode/e/estout/esttab.html) workflow which doesn’t require an excessive amount of post-processing in order to be usable. [Eyal Frank](http://www.eyalfrank.com/) deserves a hat-tip for helping inspire this process.

[First check out the sample .do](https://github.com/a8dx/Stata-Tools/blob/master/esttab_Latex_Sample.do) to follow along, downloadable from Github. It’s a basic Stata function which enables me to change dependent variables (`dv’), but otherwise retain the same specifications across all columns. You’ll notice a few things:

- Fixed effects are invoked using estadd and local macros. I follow them with replace to ensure that values correspond to the model that’s just been run, and not a previous model by accident.
- The row order of fixed effects is specified in estout’s <span style="text-decoration:underline;">s</span>tats parameters (“s(…)”) and need not adhere to the sequence in which they appear following a model.
- Some manual work is needed to ensure the correct number of columns is used if you want additional header rows, likely when multiple columns have a common dependent variable. In this instance, Columns 1-3 have unconditional days worked as the DV, while Columns 4-6 are conditioned on non-zero outcomes. I’m sure there’s a way to relate {\*M} to those values, but I’m fine with manually counting and correcting if you generate a multicolumn width error.
- If you don’t want this additional header, you can remove the “&amp;multicolumn{3}…” line without problem.
- I really like \\cmidrule, but to prevent those underlines from bleeding into each other, include the (l) argument as seen in \\cmidrule(l){2-4}. Tip: \\cmidrule can also be a real pain – if you want to use it on a single column, you’ll have to repeat the column value as the second argument, e.g. \\cmidrule(l){2-2}, NOT \\cmidrule(l){2}. I wasted too much time figuring that out. In order to get midrule to work, load both the booktabs and multirow packages, e.g., \\usepackage{booktabs} \\usepackage{multirow}.

What I really like about the prehead posthead syntax affixed to the bottom of the esttab command is the seamless multicolumn centering of footnote text, which otherwise is likely to misalign at least one column of regression output. All that nonsense is avoided here.

Of course there’s additional flexibility to add horizontal lines, remove them, insert line breaks, etc., but I think this approach provides a streamlined solution to the problem of efficiently exporting Stata results and still retaining some control over header specifics.

Lastly, this isn’t possible without esttab, do definitely spend some time investigating the numerous formatting options and flexibility built into the various est\* programs.

![Screen Shot 2016-05-27 at 6.13.42 PM](https://pathindependence.files.wordpress.com/2016/05/screen-shot-2016-05-27-at-6-13-42-pm.png?resize=712%2C571)