---
title: "Christmas-Lights 2024"
layout: post
date: 2024-09-27
---

If you've been following my various technical adventures, you will know how mich we've done to invest in our Christmas decoration game. 2024 marks the first year we will be introducing our signing Christmas house since we've moved. 
<!--more-->

### Why. 
In [our first year doing this](https://sth2258.github.io/blog/The-Singing-Christmas-House/), we've done purely high (mains) voltage elements. This worked well enough, but had [it's own challenges](https://sth2258.github.io/blog/The-Singing-Christmas-House-Part-II/). 

Then we graduated to using pixel lights in conjunction with pixels. These worked good as well, but the big challenge with this is two fold:

1. The amount of setup involved is insane. 

We used some hacks such as attaching [pixel node netting](https://www.holidaycoro.com/PixNode-Net-RGB-Pixel-Node-Mounting-Net-p/775.htm) affixed to PVC pipe to mount these elements. You have to have permanent mounting points on the house, but only used for the Christmas season. 

2. During the day, it looks atrocious. 

It honestly looks like trash was thrown all over the lawn. Half of the elements are slapped together with CPVC, HDPE and hot glue.

With these two items in mind, I aimed to step up this game. 

### What.

All the rage these days are items like [Govee's permanent outdoor lights](https://us.govee.com/collections/outdoor-lights), but come with a bit of a downside. The default implementation uses a proprietary controller that only works using their app. If we wanted to keep the musical animations that we've had in previous years, this would not do but we can work around this. 

We'll leave these permanent lights up year around, but allow also some other models to be put up and taken down with ease. My thoughts are:
- The permanent lights, mounted to the bottom of the rake board, providing an outline and straight line strings
- A 2ft wreath, wrapped with pixels
- 8x (or fewer) spotlights - these provide really great up-lighting and are a very cool dynamic element
- Pixels wrapped around the bushes - tbd if I end up doing this, we'll see

### How.

![Layout]({{ site.baseurl }}/images/2024-09-27_11-01-23.jpg)

These kinds of things are always a bit awkward to depict, so I'll try to explain a bit. 

These controllers all work over standard TCP/IP, but the protocols they use are a bit chatty (DDP and E1.31). To make sure my primary network is not effected, I'm using a dedicated "Christmas" VLAN (listed as VLAN3 above) to segregate this traffic.

Inside my primary network closet will house one port dedicated for the primary [Falcon Pi Player](https://github.com/FalconChristmas/fpp/). This is the open source software responsible for controlling the show. It will have the 7W FM transmitter connected directly to it allowing you to listen to the music that is playing during the show. The FPP software is installed on a Raspberry Pi device and connected to VLAN3. Inside the configuration of this primary player, you configure the IP targets and channels for all of the downstreams that it will communicate. 

![FPP]({{ site.baseurl }}/images/2024-09-27_11-57-16.jpg)

Inside the above configuration it's configured to use the E1.31 protocol to communicate with the spotlights (solid state relays) controller, which actually also runs FPP (in LED Enclosure 3)

The primary network closet is also physically interconnected with a 2nd network closet upstairs. This 2nd rack will have 3x vlan ports running to LED Enclosure 2, as well as the trunk ports to the basement switch. 

#### LED Enclosure 1

![FPP]({{ site.baseurl }}/images/PXL_20240927_160853043.jpg)

This will be the year-round controller that will provide primary power and signal for the Govee permanent lights. The LED output 1 from the DigiQuad controller will be connected directly to the signal wire of the Govee pixels. Since these pixels are 36v, they cannot use the same PSU that powers the controller (which uses 12v), but the grounds there do need to be tied together. Instead, we will use the PSU that comes with the kit, and splice these in directly to the pixels. 

Note: it's quite important that these lights do not constantly receive 36v without a corresponding signal. As such the power to this box is switched using a TP-Link Kasa device. 


#### LED Enclosure 2

![FPP]({{ site.baseurl }}/images/PXL_20240927_160725470.jpg)

The pixel run of Govee lights is a long one, with some rather large gaps in the connection because of the various elevation changes on the front of my house. This is the primary reason for this box. It provides both 36v power injection as well as powers the [F-Amp](https://pixelcontroller.com/store/accessories/53-famp.html) that will be used to boost the signal along those longer distances. Similar to the voltage changes that happen in LED Enclosure 1, in order for the signal to process properly, the 12v PSU that powers the F-Amp also needs to be bonded (grounds interconnected) to the 36v on the Govee pixels. 

This box will also host a DigiQuad controller which will be the output location for the pixel wreath that I'm planning on building later this year. 

#### LED Enclosure 3

![FPP]({{ site.baseurl }}/images/PXL_20240927_160757022.jpg)

This isn't exactly "LED" related, but you get the idea. This is also probably the box that takes the longest to build....but closest to my heart. These 'dumb' switches can power anything that we want. In the past I've had traditional light trees connected there, spotlights and traditional trees. 

To build one of these it takes: 
- the FPP controller, using GPIO outputs to trigger the....
- solid state relays - I use SSRs because they are faster than the traditional ones, without the loud clicking noises too (and less prone to failure too). Downside is they only support a few amps through them, but since these only power one light it's not a problem. The rest of the stuff is just some basic voltage protection, bus bars and various splices to make the whole thing work safely. 

![FPP]({{ site.baseurl }}/images/2024-09-27_12-42-12.jpg)

![FPP]({{ site.baseurl }}/images/2024-09-27_12-41-05.jpg)

This box also has the FFP running in remote mode. When it receives a signal over the E1.31 protocol for one of the channels it's responsible for, it will send the HIGH command to the GPIO output, which sends the required voltage (2.5V - 20V) to the relay, which then allows the (normally closed) relays to open, allowing the magical pixies to flow. When the GPIO signal is set to LOW (0V - 0.5V), the relay closes.

### When.

Welp, the plan is to have most of this setup for Halloween. We won't need to sequence (lights dancing to music) for Halloween but we'll use the WLED software that runs the Digiquad controllers to have some cool lighting for the season. Stay tuned for the adventures in that arena!