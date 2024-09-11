---
title: "The Singing Christmas House"
layout: post
date: 2015-11-28
---

For starters, its Christmas season - which for me means all sorts of woodworking / crafty / electrical based projects, with my wife as a lead visionary. Some our projects last year included the "[North Poles](http://haberhomestead.blogspot.com/2014/12/crafting-north-poles.html)" and "[Merry and Bright](http://haberhomestead.blogspot.com/2014/12/merry-and-bright.html)", but this year is bigger in many ways.
<!--more-->
I always enjoy these sort of arts meet sciences projects that Steph and I get to do - the last one of this scale was the "Tablecarder" application which served as the table place cards for our wedding; she has the vision, and I make it happen :)

Steph has always had very vivid, amazing memories of visiting the singing Christmas House in her hometown of Fairport, NY which synchronized lights and music (broadcast over an FM radio station) - so naturally, when we though of how to expand our external Christmas light display, this was our goal. 

So with this as my mission, I immediately thought of my Raspberry Pi. Over the past few years, the amount of micro-controllers (such as Pi and Arduino) available on the market has skyrocketed, leading to many innovative projects.

With some quick googling it seems like this is an easy project to take on. Ultimately all you need is:

- Raspberry Pi
- Male/Female jumpers (for carrying the signals)
- Relay board (for handling the GPIO signals sent from Pi)
- #12 / #14 AWG
- 4 outlets

All of the above components combined creates and 8 channel system that will flash along with the music it plays.

For starters, I connected up the outlets, making sure to break the tab so it allowed two different power sources to be connected to them.

Next, I connected up the Pi to the solid state relay board.

![Dave and Steve]({{ site.baseurl }}/images/blogger-image--1645300972.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image-1628147368.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image-340316581.jpg)

Next, it was just a matter of splicing together the neutral wires, and ground wires. I used a few larger wire nuts, but have seen some good implementations that re-purposed grouding bars from circuit breaker panels

![Dave and Steve]({{ site.baseurl }}/images/blogger-image--690280103.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image-1666331868.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image-978282789.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image-1062371081.jpg)
![Dave and Steve]({{ site.baseurl }}/images/blogger-image--57454194.jpg)

Then it was just a matter of the software, which is really just an open source implementation of LightShowPi, and a PHP web interface that executes Python scripts.

And now its just a matter of picking out music, which is harder than you might think! The whole interface is managed using a web interface (or using a playlist which is fully hands off).

We also got a FM transmitter which is capable of broadcasting up to 7 miles so we can transmit our songs over the radio for our passers by.

Stay tuned (har har har!) for some videos of our light display. 