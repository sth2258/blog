---
title: "The Singing Christmas House Part II"
layout: post
date: 2016-11-18
---

I'm still not certain if there are many other people out there that like Christmas more than Steph and I do.

Last year, we took our first steps into "next level" Christmas lights with the construction of our show that automatically turned on lights based upon the audio frequencies within a soundtrack. This had some severe drawbacks.
<!--more-->
##  It was too small!!

If you check back to the previous post, it showed us using some prebuilt software (LightshowPi) and an 8 channel solid state relay board. All this "8 channel board" means is that you're limited to turning on and off eight strands of lights during your show. I'm sure all you Christmas nuts out there know, 8 is good....but 16 is better.

![Dave and Steve]({{ site.baseurl }}/images/20161118_172956.jpg)

 There are so many channels, I am having a very hard time trying to find solutions to hold it all.

There was also a limitation to the types / sizes of lights we were able to install. The solid state relay board is really a hobbyist type, but lets be honest people....this is an industrial application. We were previously limited to 2 amps per channel - this limitation led to...well, see for yourself.

![Dave and Steve]({{ site.baseurl }}/images/12301481_10101531081592255_7575344281977432654_n.jpg)

Yes folks - those are melted relays. For some of the channels, we FAR exceeded that 2 ams (whoops). But don't worry folks - [Amazon's got a solution](https://www.amazon.com/uxcell-SSR-25-3-32V-24-380V-Solid/dp/B0087ZTN08/ref=sr_1_1?ie=UTF8&qid=1479508819&sr=8-1&keywords=solid+state+relay). I installed 2 of these guys in our new setup, though it took a bit of hacking to get there. 

`<nerd>`
These relays require a control voltage of min 3 volts in order to trigger. The trouble is, the PWM outputs on the Raspberry Pi output at 1.5v - so no good. The proper solution would have been to use an N-Channel MOSFET as a DC switch so that when the 1.5v hit the MOSFET, it opens up and lets pixes flow for another transformer that I would have to connect (say for something like an old phone charger). But - I wasn't about to solder or throw a breadboard into this setup, so the fun little workaround I found was to connect an AA battery in series with the GPIO output - so that way, a constant voltage of 1.5v hits the relay (and does nothing) but when combined with the other voltage it is enough to trigger the relay.
`</nerd>`

##  The software...

Just like I mentioned before, the software we were using was fairly dumb - It would:

1. Play a song
2. For each sampling, it would break that sample down into X frequency spectrum
3. From each of those spectrum in the sample, if the frequency was active, it would turn the lights on, and if the frequency was not active it would turn it off.

This is very clearly not ideal. We wanted complete control of the songs and which lights turned on when. This, my friends, is not an easy ask. Fortunately enough, there is plenty of open source software out there that will allow you to synchronize and choreograph your lights to music, but it takes a lot of work. In this 2016 iteration of our setup, we switched out the LightshowPi software for this option.

###  Welcome Falcon FPP and XLights

Falcon FPP is a pixel controller, which lets you put together the amazingly complicated LED Christmas shows you have likely already seen; but for us little guys just starting, it also lets you control single GPIO outputs and set those up as channels.

![Dave and Steve]({{ site.baseurl }}/images/Capture.PNG)

xLights, then gave us the ability do a few things. We set the software up so that it shows exactly where on the house the lights will lay. 

![Dave and Steve]({{ site.baseurl }}/images/Capture2.PNG)

With xLights now setup, you can then use it to choreograph your lights to music. Just setup a new sequence and turn on, and off whatever effect you want on! Again, needless to say, this is a big ask - with 16 different channel options to turn on/off throughout the course of a song, the show can get pretty complicated....

![Dave and Steve]({{ site.baseurl }}/images/Capture3.PNG)

When your done, just set it to play, and boom. It just works.
