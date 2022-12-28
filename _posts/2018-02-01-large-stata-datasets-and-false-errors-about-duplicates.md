---
id: 1299
title: 'Large Stata Datasets and False Errors about &#8216;Duplicates&#8217;'
date: '2018-02-01T01:49:42+00:00'
author: andagostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=1299'
permalink: /large-stata-datasets-and-false-errors-about-duplicates/
timeline_notification:
    - '1517449785'
image: /wp-content/uploads/2018/02/screen-shot-2018-01-31-at-5-49-02-pm.png
categories:
    - Computing
    - Data
---

Variable storage types exercise more importance when working with larger datasets, and variables with more digits. I’m reminded of this because of an error message Stata threw while trying to perform a long `reshape`, claiming duplicate entries of the ID variable. That was obviously not the case, since the `_n` id was uniquely created, and the value of each visibly corresponded to its row index.

The problem is Stata’s default type for numeric is as a `float`. Under many circumstances, that’s fine for either integers or decimal numeric objects. But with N≥ 20 million, the dataset that prompted the error is butting up against precision limits since a `float` [is accurate to 7 digits.](https://www.stata.com/manuals13/ddatatypes.pdf)

The solution is to use a `double` type instead, which can reliably hold up to 16 digits. Anything larger, you’re likely best off working with strings.

Specifying the storage type is straightforward, like so:

> `gen double id = _n`

So when you’re trying to reshape a large dataset and Stata quits, even though you *know* you’ve satisfied uniqueness in your identifying variable, **double**-check your ID’s storage type.