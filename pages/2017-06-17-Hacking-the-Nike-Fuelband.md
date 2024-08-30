---
title: "Hacking the Nike+ Fuelband....ok, maybe not"
date: YYYY-MM-DD
---

I know you might not be able to tell by my svelte physique, but I've recently become very into running; and being a technologist at heart, I am always fascinated with seeing all the new cool gadgets being used out there.

When Buster (the Boxer) and I first started back in March, I found this very cool Nike+ GPS app which will track your runs via the GPS in my iPhone, and give you feedback and cheers as you reach your goals. Tim Tebow himself told me how good I was doing! The app is very cool, and will sync all of the data it keeps about your run, to the Nike Running website. Probably all stored in a data warehouse somewhere, so Nike can sell you products catered to you, or maybe just make fun of how slow I am; but that's beside the point.

Nike seems to have a clear dominance in the running advertising space, so when I was browsing their website and stumbled across the Nike+ Fuleband, I was intrigued. This little "Livestrong" type wristband will track all of your movement throughout the day, so you can set goals, and track your movement day over day. Your movement is tracked in a arbitrary value known as "fuel". Since the band has data for your height, weight, and step-counts, it then is able to determine how much "fuel" you are burning in your various movements.

Now on to the techy stuff....

The band itself has a standard USB port at one end of it, and built in Bluetooth. You can choose to sync the band with a Bluetooth enabled device, or plug it into a Mac or Windows based PC; the PC must be used in order to charge the device.

In order to get the device operational, you must visit the Nike+ website and download the Fuelband (or at least I thought it was dedicated to the Fuelband) software in order to sync the device. Once the software is installed, simply enter your Nike+ login details, and the device will sync away.

Again, being a techy at heart, I must know what is going on in that sync process. How is my fuel data being sync'd to the Nike cloud!!

At a high level, after you plug in your device, the following occurs: 1) FuelBand software auto-opens 2) Connection is made from the software to the Nike website 3) Data is downloaded from the FuelBand 4) Data from Fuelband is uploaded to the website and 5) A new browser website is opened to the Nike+ website to show you all the data that was just uploaded

To start disassembling this, I thought, what better way to do so, then to decompile the Nike+ FuelBand software. Attempts were foiled when all decompilers I tried, failed miserably. Looking closer at the software, it looks like it was written in some variant of C (being cross-OS compatible), and without digging deeper into tools like IDA, I was going to get nowhere.

The software itself has some localization support in it for multiple languages - I believe the device is only available in the US...maybe Nike has plans to roll out elsewhere?

Using procmon (on my Windows 7 PC), I was also able to see that the app is modifying a file called "config.dat" - This is where my perspective on the software changed. The brilliant folks at Nike seem to have made this one application for use with multiple different devices. The program files directory also contains some DLL files, which I assume are various drivers for performing IO operations to the different devices. The config.dat was also a dead end; the file is just a very large XML files with URL's and locations to things like software updates for the devices.

So, what next? Wireshark. Having very little experience with the product, I wasn't able to do much immediately. After some fiddling, I was able to see the network operations that the software was performing, which would be the key to "hacking" the FuelBand.

From step 1 to step 5 (mentioned above) there were around 120 packets being sent/received. Right out of the gate, the software is performing DNS queries...ET phone home. These DNS queries (for nikerunning.nike.com) were just so the software can download the XML file referenced above - Following the TCP stream, we can see an HTTP GET issued to /nikeplus/connect5/config/config.xml on nikerunning.nike.com; the file looks to be cached locally, thus the "config.dat" file.Sometime after this XML file download is where I assumed the meat and potatoes to be. So, scrolling packet 65 in the capture, we again see some DNS operations being performed...this time to api.nike.com. Again, from the URL, it looks to be indicative of things to come in the future. Google tells me also that there does appear to be some movement on the FuelBand API front; no real details yet though. 

Also, if you, just as I, performed a dig against that api.nike.com domain, you will see the glorious amazonaws.com domain hosting API (assume its web services of some type). As a side note, I am very happy to see a brand as prestigious as Nike, link its brand to Amazon; it says a lot for the awesome work Amazon is doing in the cloud space.

So you all must be dying to know what data is being sent back and forth to the Amazon AWS/Nike API site! Well, the answer is....I can't tell. What I can tell, is that the network traffic is encrypted (using the Thawte SSL root CA). From some of (very little) clear text I do see inside the TLS v1 payload, there are references to api-preprod.nike.com, api-tie1.nike.com, developer.nike.com and api.stage.nike.com. I would assume that the data being sent up to the AWS website is a serialized and encrypted version of the object data being stored on the device. I can't really blame Nike for encrypting & serializing the data before sending to up to the API - best practices.

All in, the device is very cool - and has some very interesting technology behind it. Looking forward to the API, whenever that comes out...

Add some comments below if you got any further than I did :)
