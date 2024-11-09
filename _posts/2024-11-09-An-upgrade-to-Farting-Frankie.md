---
title: "An upgrade to Farting Frankie"
layout: post
date: 2024-11-09
---

One of the most amazing lawn inflatables of the Halloween season comes with one crucial flaw. It works terribly at night!
<!--more-->
This is of course very unfortunate because they are designed to run **at night**.

This, is farting frankie, and a neighbor of mine approached me with an interesting challenge. Farting Frankie is supposed to fart at a potentiometer-controlled volume, based on motion of people passing in front of him. The way it's triggered is via photoeye, which detects differences in visible light, and as people pass by, if it detects enough change in light, it assumes this is motion and triggers the fart. 
![Layout]({{ site.baseurl }}/images/th.jpg)

The big issue however is this simple photoeye. It's built inside a simple 5v circuit which; if the voltage is high enough (enough light difference) it triggers; which of course is a challenge, unless the sensor is under constant light, it won't be able to trigger. 

# The Solution

[Passive infrared sensors](https://en.wikipedia.org/wiki/Passive_infrared_sensor), as compared to photoeyes, are much more complex sensors and in-turn, much more sensitive to changes in motion. The negative however, as already mentioned, is that the existing sensor is built into a 5v circuit, which is not the way these sensors work. 

[Sensors such as these](https://www.amazon.com/Weewooday-Adjustable-Infrared-Controller-Weewooday-62301/dp/B09P3LK1B8?crid=25GYG25ZMINJS&dib=eyJ2IjoiMSJ9.SXunZP8zHjrpS_CFwA-n2MRt-t5VtpzqMwCHF-pDceK0v7FZhTYL7kV1ydBjP-Dxo0MhqZ-jFd5YEM6vJm6W_To1YbuWi7_qsVroqNPoWtjFbYlt1ymBN6uFFjJEsgi2J92FW7mYY7mHybjXdJn7NiyT9-WDN5tJKv9SSAFnSxSES0E3RVZ6FAos-Lrl7pJti4YnWY1iN-UDdFuiGXOy5kn3Sf6v1OAlfCUYKnSfEZgfClsBJU3A813m5U4U6J_OxDLwzBNGjIQOT63QuWntQ80i2pXfQWDRwc98N3qsU1A.kD_GzKk5jkocpgcLOzacGI2XwEhZAAVpdEoglbc6j7w&dib_tag=se&keywords=12v+PIR+sensor&qid=1731123699&sprefix=12v+pir+sensor%2Caps%2C111&sr=8-5) require a 12v input, and once IR is detected, allows the same 12v to pass through to the output. We get around this using a [simple relay](https://www.amazon.com/AEDIKO-Channel-Optocoupler-Isolation-Support/dp/B095YFJ69T?crid=RTO67A82Q5AX&dib=eyJ2IjoiMSJ9.KlO8_9d6gyhk-wUDIy19YX1i_fzeDghqzyVBAwR0tw4ezAShVSOz5GpcG6lzFSI5_gI3XumIzghSD0f98sA-_KBE6BSgnNKleAGWq44Va0Kpww_GChu6VSq88VM4gWeXrlE1AodIxSnuweJ3up1mYzRSJPUVDreAq0luxs5KpCSPuT2ric6xyqS9YcKFMbFdYsfYCnT0U4zLSNXFuvVcf24wJCv-RT3aulMY6m3si_Q.xPqU-uGOrHcE55lw0_yhbj_lVN1GGIceFxJjVRiGQkg&dib_tag=se&keywords=12v%2Brelay&qid=1731120511&sprefix=12v%2Brelay%2Caps%2C133&sr=8-5&th=1). 

![Layout]({{ site.baseurl }}/images/2024-11-08_22-25-22.png)

We luck out in that there is an existing 12v circuit for this prop, which is used to power the fans that inflate the device. We tap into this 12v circut to power our new PIR sensor, as well as our relay (labeled `DC+` and `DC-` on the relay). The +12v **output** of the PIR sensor is connected to the `IN` on the relay, which is used as our relay trigger. The specs of these relays specifically allow for triggering of (HIGH, aka closed circuit) for 4.5 through 12VDC, which of course is what our PIR sensor is putting out. 

The existing photoeye sensor is on a 5vdc circuit. We simply cut the existing connection, and connect the +5vdc line into the `COM` port on the relay, and the photoeye load wire into the `NO` port on the relay. 

A few notes here:
- Yes, it's likely possible to swap the +5vdc and load wires of the photoeye, however lots of relays have built in diodes which allow voltage to flow in only one direction, so much safer to test which side is **line** and splice accordingly. 
- `NO` on the relay stands for normally open; meaning the circuit is 'off' until the trigger closes it. 
  - If you screw up and connect it to the `NC` (normally closed), it would mean constant farts until the PIR detects motion. Quite the opposite of what we want, but would most certainly be a fun outcome :P
- There is a high/low signal jumper on the relay - This lets you control whether the relay is triggered by a high or low (aka on and off in human speak) in the circuit. Our sensor will always be low unless the sensor detects motion, which then sends high (on, +12v). High is the default, and yes of course you can offset this by using `NO` / `NC` - but lets k.i.s.s. - `HIGH` plus `NO` will do just fine.