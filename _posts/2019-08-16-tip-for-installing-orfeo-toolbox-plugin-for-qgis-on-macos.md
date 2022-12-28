---
id: 1467
title: 'Tip for Installing Orfeo Toolbox Plugin for QGIS on MacOS'
date: '2019-08-16T02:47:08+00:00'
author: Anthony Louis D'Agostino
layout: post
guid: 'https://anthonylouisdagostino.com/?p=1467'
permalink: /tip-for-installing-orfeo-toolbox-plugin-for-qgis-on-macos/
image: wp-content/uploads/2019/08/logo.png
categories:
    - Computing
    - GIS
---

![image](/wp-content/uploads/2019/08/logo.png)

Last weekend I spent longer than I care to admit trying to get [Orfeo Toolbox](https://www.orfeo-toolbox.org/CookBook/index_TOC.html) (OTB) to play nicely with QGIS on MacOS High Sierra. I tried QGIS 3.4, the stable version, and I tried 3.8, the latest version, and scoured every forum and installation tutorial I could find for activating the tools and having them appear in QGIS’ Processing Toolbox. Once they appear, ensuring they actually work is a separate issue. One piece of advice seems to have made all the difference.

So, I want to shout out Juan Ramón Selva for pulling me from the morass with what appears to be the most important detail that’s not adequately emphasized in any OTB installation instructions.

When you extract the OTB-6.6.x-Darwin64.run file that was downloaded from the OTB site in Terminal, it’ll create a folder entitled “OTB-6.6.x-Darwin64.” You may have read in some fora that OTB acts up if the target folder includes non-alphanumeric characters like hyphens or spaces, and have been tempted to manually rename the folder to something innocuous like “OTB.” As Juan points out [here](https://gitlab.orfeo-toolbox.org/orfeotoolbox/otb/issues/1745), that’s a no-go. Instead, you need to specify any differently-named target folder like “OTB”, as explained in his Oct 19, 2018 post.

Once that was sorted out, I went back and revised the remaining installation steps:

- Add the plugin and [specify the Name and URL](https://gitlab.orfeo-toolbox.org/orfeotoolbox/qgis-otb-plugin/blob/master/README.md) in QGIS’ plugin manager.
- Properly identify the OTB and OTB application folders, as described in the Open processing settings section [here](https://www.orfeo-toolbox.org/CookBook/QGIS-interface.html).
