---
layout: post
title:  Naughty or Nice Santa Hat
date:   2016-01-06 09:46:53
categories: esp8266 iot junkdrawer lua python nodemcu flask
permalink: /santa-hat/
---

# This hat uses crime data to decide if your hood is naughty or nice.

![Hat Displaying Animated Colors]({{ site.baseurl }}/assets/santa-hat/IMG_6787.GIF)

The Sunday before Xmas I woke up with the great idea of building a blinky santa hat that would be a festive and silly thing to wear throughout the holiday.  Looking outside at the [record rains we've been getting](http://www.sfgate.com/news/article/Late-week-rain-to-open-Bay-Area-storm-door-until-6702165.php) I decided to see if I could build it out of components left over from other projects I already had lying around the house so I could spend my Sunday warm and dry.

## Hardware 

Digging through my storage boxes and junk drawer I managed to find the following:
	
* A strip of [APA102 RGB LEDs](https://cpldcpu.wordpress.com/2014/08/27/apa102/)
* An [EzSBC ESP8266-07 Breakout Board](http://www.ezsbc.com/index.php/featured-products-list-home-page/wifi01.html)
* A bunch of [Titanium Innovations CR123A Batteries](http://www.batteryjunction.com/tpen-tcr123a-.html) and holders

Using a couple bits of protoboard and random hookup wire I connected two of the CR123A batteries in series to provide a 6.4 volt power source which I used to run both the ESP8266 and the LEDs.

The LEDs are connected to the ESP8266's SPI (MOSI/CLK) pins.

![Electronic Components]({{ site.baseurl }}/assets/santa-hat/IMG_6772.JPG)

I was slightly concerned about putting 6.4v into the LEDs as the datasheet says not to give them more than 6 volts, but it seemed to work just fine. Your mileage may vary; there are a large number of APA102 clones on the market some of which are of dubious quality.

The ESP8266 is a 3.3v device, but the breakout board included a voltage regulator which can handle stepping the 6.4v down to a stable 3.3v. Overall this board is a good deal for a Made-in-the-USA full featured ESP8266 breakout, but it it does have a couple of problems.  It doesn't expose a pin for the VIN for the voltage regulator so I had to solder in a jump wire to power it from a non-USB source.  Additionally, it uses the older-style mini-USB connector rather than micro.

After soldering everything together, and checking the connections, I rolled it up into the brim of a santa hat and the hardware was complete!  

![Components Installed in Hat]({{ site.baseurl }}/assets/santa-hat/IMG_6779.JPG)

## Software

The ESP8266 shipped with [NodeMCU](https://github.com/nodemcu/nodemcu-firmware) pre-installed so I wrote a quick 10 lines of lua to light up each led to ensure everything was working as expected.  When I tested it out, I was pleasantly surprised with how well the faux-fur around the brim of the hat diffused the LEDs. Success!

![Hat with one LED lit at a time]({{ site.baseurl }}/assets/santa-hat/IMG_6749.GIF)

Initially this was all I had planned for this project, but the ESP8266 includes a wifi radio and TCP stack, so I might as well add a santa hat to the [Internet of Things](https://twitter.com/internetofshit).  Sticking with the theme, I decided I wanted the hat to use it's internet connection to act as a "naughty or nice" detector. 

After a few minutes of looking at available APIs, I stumbled across [SpotCrime](https://www.spotcrime.com/) and their API which aggregates crime data from many agencies and sources into a common format ([SpotCrime Open Crime Standard (SOCS)](http://blog.spotcrime.com/2014/03/the-spotcrime-open-crime-data-standard.html)).  TL;DR you make an HTTP GET providing your lat/lng and you get back a JSON document with all the crimes in area over the last 7 days.

Now I just needed to find out how to have the ESP8266 figure out where it was.  I planned to tether the hat to my phone while I was away from the house so I didn't want to trust IP geolocation; I needed something more accurate.  Thankfully Google provides the [Google Maps Geolocation API](https://developers.google.com/maps/documentation/geolocation/intro) which allows you to provide a list of BSSIDs nearby and it will approximate your location.

Now that "the cloud" was doing all the heavily lifting for me, I just had to write the code to tie it all together.  I decided to put as much of the code on the server as possible.  Given my [prior](https://en.wikipedia.org/wiki/NinjaTel_Van) [experience](https://en.wikipedia.org/wiki/Capture_(TV_series)) with wireless deployments, I don't trust a tethered connection to a mobile phone to be reliable or robust.  Limiting the amount of data I send/receive over the wireless link would be critical to having the hat actually function in the real world. 

### Server

I wrote a tiny webapp in Python using [Flask](http://flask.pocoo.org/) which accepts a simple string of data from the ESP8266, handles the API calls to SpotCrime and Google, processes the results to determine if an area is naughty or nice, and returns a simple naughty/nice/unknown response to the ESP8266. I hosted the app on [Google App Engine](https://cloud.google.com/appengine/docs) for free.


<script src="https://gist.github.com/cnelson/750202a0dc2c5b26b202.js?file=hat.py"></script>

### Client

I then wrote a very simple client in lua which attempts to connect to my phone's hotspot and update it's location and naughty/nice status every 5 minutes by talking to hat.py over the internet.  If the local area is deemed "naughty" the hat flashes red, if it's a "nice" hood the hat flashes green.  If it can't determine, or if the internet is unavailable, it defaults to a nice white/red/green fill loop.

This code was then flashed onto the ESP8266 using [nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader).

<script src="https://gist.github.com/cnelson/750202a0dc2c5b26b202.js?file=hat.lua"></script>

You can get [the code on Github](https://gist.github.com/cnelson/750202a0dc2c5b26b202) but please consider it POC quality at best!

## Real world performance

Knowing that the ESP8266 can pull upwards of 200ma when transmitting, and that the LEDs were using ~400ma; I was pleasantly surprised with the runtime of 3 hours I got out of a pair of [5 years old](http://ninjas.org/badges/defcon18.html) 1400mah batteries. I did notice that after extended use the batteries would become very warm to the touch.  I don't know what the maximum current draw from these cells is, but it's quite possible I was exceeding it when all the LEDs were lit full white.

I'd estimate the hat was able to update it's crime data successfully about 90% of the time.  Failures were mostly related to associating with my phone's hotspot.  When that did occur, a quick reset would fix it, which was simple thanks to the reset button on the breakout board, which I could easily press through the hat.

## Summary

Considering I only put about 6 hours of work into it, the hat turned out great, and I was very happy with the performance.  It was amazingly bright; I was asked to turn it off in a dimly lit restaurant, and I received dozens of compliments / questions while wearing it in public.

Santa even decided to wear it while doing Christmas Karaoke:

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">If you are in LA and not at Christmas Karaoke at <a href="https://twitter.com/SardosBar">@SardosBar</a> then you are missing out <a href="https://t.co/MfKSESwbFX">pic.twitter.com/MfKSESwbFX</a></p>&mdash; Physical Constant (@299792456nelson) <a href="https://twitter.com/299792456nelson/status/680617480998395904">December 26, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>



h0 h0 h0


