---
id: 153
title: 'On Working with India&#8217;s NSS Data in an OS/X Environment'
date: '2015-06-03T17:49:26+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=153'
permalink: /on-working-with-indias-nss-data-in-an-osx-environment/
sharing_disabled:
    - '1'
geo_latitude:
    - '40.7127837'
geo_longitude:
    - '-74.0059413'
geo_address:
    - 'New York, NY, USA'
geo_public:
    - '1'
image: /wp-content/uploads/2015/06/link_nesstar.gif
categories:
    - Computing
    - Data
---

[![link_nesstar](https://pathindependence.files.wordpress.com/2015/06/link_nesstar.gif?resize=300%2C65)](https://pathindependence.files.wordpress.com/2015/06/link_nesstar.gif?resize=300%2C65)

I’ve shifted gears after spending the last few months with district-level data and begun getting my hands dirty with the National Sample Survey data the Indian government regularly compiles. If you’re familiar with the flat text/ASCII approach to digging out your data, you understand the unwieldiness of this format in which variables are created from specifying byte lengths and column positions. This seems reasonable in the age of punch cards, but…

However, NSS has begun rolling out “new format” data which can be unpacked and converted into SAS, Stata, etc. formats. This is not entirely straightforward, especially for Mac users since the data uses a PC-native Nesstar executable. After some time fiddling with this, I recommend the following:

\* Download a .rar-specific unzipper. I originally used Ez7z, but encountered several crashes so switched to the no-fuss [StuffIt Expander](http://www.macupdate.com/app/mac/20954/stuffit-expander). This will generate a parent folder with nicely organized paths that include webpages with variable descriptions and other helpful information. Assuming there’s uniformity across datasets, you can look into the “/survey0/data/Document/” directory for PDF versions of all the necessary documentation on region/occupation/etc. codes and copies of the original survey in English.

\* Download [PlayOnMac](https://www.playonmac.com/en/) to run Windows executables on your Mac. If you don’t already have [WineBottler](http://winebottler.kronenberg.org/), now would be a good time to pick that up. My \[incomplete\] understanding is that PlayOnMac provides the interface to use Wine services, so I believe they’re both required.

\* Now when you view your NSS “new format” directory, the “AutorunPro.exe” file to run Nesstar should feature the PlayOnMac clover icon. Running that file will open PlayOnMac and the first time around you’ll be able to install NesstarExplorer. If in PlayOnMac you create a shortcut icon to NesstarExplorer, you can easily navigate to that installer which is how you’ll unpack your actual data files.

\* Locate the \*.Nesstar file in the “survey0/data/” directory. Ideally you can “File/Export All Datasets,” specify your preferred file format and path, and be done. However, these filenames tend to be long and may exceed filename limits. You’ll then have to identify how far you got in the unpacking process, return to the Export All Datasets option, and then unclick the offending file(s). When you manually select individual files, you control the filename and can trim that down to avoid the length violation.

<div class="geo geo-post" id="geo-post-153" style="display: none"><span class="latitude">40.7127837</span><span class="longitude">-74.0059413</span></div>