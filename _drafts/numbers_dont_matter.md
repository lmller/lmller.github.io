---
layout: post
title:  "The numbers don't matter"
date:   2016-09-16 10:00:00
description: Google's Android Dashboards are the first address if you need to find out the distribution of a certain Android version. But how useful are these stats in the real world?
comments: true
categories:
- android
- non-technical
permalink: numbers-dont-matter
---
Google's [Android Dashboards][cac5a53b] are the first address if you need to find out the distribution of a certain Android version. But how useful are these stats in the real world?

To understand how we can use the numbers for app development, let's take a look on how the data is collected.  
Each month, some magic script on googles servers (hopefully, it _is_ a script and not a human employee) records all accesses to the Google Play Store from each Android device running any Version greater or equal than **2.1** ([Froyo][5bf955b9]).  
The recording usually starts one or two days before the end of that month.

Let's take a look at the distribution measured between August 30 and September 5 2016

Version  | Api | Distribution
--|--
2.2 (Froyo)                         | 8  | 0.1%
2.3.3 - 2.3.7 (Gingerbread)         | 10 | 1.5%
4.0.3 - 4.0.4 (Ice Cream Sandwich)  | 15 | 1.4%
4.1 (Jellybean)                     | 16 | 5.6%
4.2 (Jellybean)                     | 17 | 7.7%
4.3 (Jellybean)                     | 18 | 2.3%
4.4 (KitKat)                        | 19 | 27.7%
5.0 (Lollipop)                      | 21 | 13.1%
5.1 (Lollipop)                      | 22 | 21.9%
6.0 (Matshmallow)                   | 23 | 18.7%  


  [cac5a53b]: https://developer.android.com/about/dashboards/index.html "Android Dashboards"
  [5bf955b9]: https://en.wikipedia.org/wiki/Android_Froyo "Read about Froyo on wikipedia"
