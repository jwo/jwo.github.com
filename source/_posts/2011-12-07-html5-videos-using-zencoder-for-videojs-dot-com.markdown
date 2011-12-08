---
layout: post
title: "The One with HTML5 Videos Using Zencoder for VideoJS.com"
date: 2011-12-07 20:31
comments: true
categories: HTML5
---

Things that are true:

* HTML5 is cool
* HTML5 Videos are Cool
* Getting HTML5 Video formats OMG all of them created ... _NOT_ cool

[VideoJS.com](http://videojs.com) will create an HTML5 embed for you that will render in iOS devices, all the modern browsers, and will fallback to flash for IE 6/7/8 types. However, to get all this goodness, you have to create your video file in 3 different formats:

* h.264
* web-m
* Theora

The [Voyeur gem](https://github.com/devthenet/voyeur) looks cool (in that you can use it to create all three formats at once), but it seems to only work for ubuntu. What's a mac fanboy to do?
<!--more-->

[ZenCoder](https://app.zencoder.com) is pretty dern cool as well -- pay as you go, no monthly fees, and it's who VideoJS.com recommends. Trouble is, you have to remember each of the formats and configure them and ZOMG it takes forever to build this API request.

So that you will not have to suffer as I have done, I created a shell GIST. Simply paste in your publicly accessible video URL (using S3 with an authenticated URL works spendidly here), paste in your API KEY, and configure the sizes you want. POST to https://app.zencoder.com/api/v2/jobs using HTTPClient and Bam. 

{% gist 1440139 %}

I have had good luck with the [VideoJS.com embed builder on VideoJS.com](http://videojs.com/embed-builder/). I've had bad luck converting to HAML for an as-yet undiscovered reason.
