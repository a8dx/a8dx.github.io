---
id: 120
title: 'Screen Sharing and Yosemite'
date: '2015-02-23T02:21:20+00:00'
author: admin
layout: post
guid: 'https://pathindependence.wordpress.com/?p=120'
permalink: /screen-sharing-and-yosemite/
geo_public:
    - '0'
categories:
    - Computing
---

[![Jumping from Yosemite?](https://pathindependence.files.wordpress.com/2015/02/os_x_yosemite_roundup.jpg?w=470&resize=470%2C258)](http://www.macrumors.com/roundup/os-x/)The Yosemite OS upgrade has been available since mid-October. Since it often takes weeks/months for developers to resolve bugs introduced by a new OS version, I held off on leaving Mavericks for fear of some vital Homebrew or Python functionality becoming a casualty of the shift and then being needlessly out of luck right before a presentation/deadline. I finally upgraded last week and the install process consumed some 9+ hours. That would be fine if the countdown clock said as much, but most of the time the install page declared the process would either be over in 20 minutes or 3 minutes (nothing ever in between, seemingly violating the intermediate value theorem).

That’s a one-time investment and so no big worry if the final goods stand up to light. While you can read about functionality changes and subjectively cleaner menus and spotlight windows elsewhere, here I’d like to talk about Screen Sharing, the wonderful, native VNC bundled in OS X. Yosemite has **destroyed** **it** (not in a positive way). The lag becomes absolutely unbearable after less than an hour of connecting to your remote system and oftentimes it’s difficult, nay impossible, for the connection to pass a handshake. You can relaunch Finder and restart your computer and still find no success.

This is a common problem, as described [here](https://discussions.apple.com/thread/6607624) and [here.](http://forums.macrumors.com/showthread.php?t=1758231) You may be able to recover some performance by closing and restarting the application, but I’ve not been able to squeeze out any speed-ups from this. Since Screen Sharing worked brilliantly on Mavericks (love the native shortcut keys when on the remote side), the poor performance with Yosemite marks a big setback for users. Plus, with no easy way to downgrade to recover that performance if you don’t either have a boot DVD of an earlier OS or a complete back-up of your drive prior to the Yosemite install, then either you’re stuck with this until Apple issues a fix or left to purchase a $20 copy of Mountain Lion or Snow Leopard. You might consider this either a [First World Problem](http://www.reddit.com/r/firstworldproblems/) or psychological torture, depending on how well you understand that trying to do work on a connection suffering a 10 second lag time is a worse experience than any 25cents/hour internet cafe I ever spent time in a decade ago.

Sure, changing to Adaptive Quality on the Screen Sharing options may provide you some temporary relief, but it’s not a first best.

Ultimately, if you haven’t upgraded to Yosemite and daily use Screen Sharing (or maybe even Back to My Mac, for which I can say nothing about), then for your health don’t upgrade. Monitor the threads I’ve linked above and I imagine that when a fix is issued, there’ll be some chatter about it on the same.

 **Update:** Just returned from the Grand Central Apple store where one of the Apple geniuses expertly identified a potential issue, trashed a relevant .plist file and then ran a fast install of Mavericks from a network drive (i.e., not through the App Store as I had previously done). Within 10 minutes we were up and running with Screen Sharing connected successfully to the remote machine on the first try. I’ll still be monitoring the setup for bugginess (reporting back if anything goes awry), but I’m optimistic this will be a successful setup which brings me exactly back to where I was about 3 weeks ago right before upgrading to Yosemite. I must give much-due respect to the two Apple employees who have been extremely helpful throughout this process. During my initial call I had no anticipation of success when the tech support on the other end, after sprinkling our ‘conversation’ with several 30-60 second long silences asked, “So is this thing like an app?” Thank god for Tier 2 support.