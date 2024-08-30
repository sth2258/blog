---
title: "Hockey Statistics & Technology -- Piggy backing off the big guy."
layout: post
date: 2013-04-13
---

There are quite a few things that I am passionate about in this world. Two of them, are sports and technology. I've recently become very interested in some of the mathematics behind the statistics in sports -- most specifically, hockey.

I, by no means, fancy myself a statistician; but this does not mean I can't play one on the internet :) In my quest to derive to develop a relationship between some of the statistics out there, I ran into a brick wall; these stats ain't free. Companies pay quite a bit of money for these sorts of data feeds.

I immediately looked at one of the big boys. The biggest, in fact...but who will remain nameless for the duration of this article.

I wasn't quite sure what I was looking for at first, but it had something to do with hockey. Given there are already quite a advanced statistics sites out there to calculate things like a player/team's Corsi or Fenwick rating, I was looking for something with more of a direct graphical relationship.

What I set out to do, was gather statistical relationships between the quadrants of the ice (graphed using standard Cartesian coordinates) and the percentage of goals that happen as a result.

When I looked at the available data that my target data source has, this seemed fully available. There is a game cast section that streams and stores all of the goals, shots, hits, blocked shots and penalties all graphed on an ice overlay (which is just a big Cartesian plane).

This page (wrapped in a Flash interface) was just sending simple AJAX calls, parsing the results, and mapping the events on the graph. The problem here was reading this data.
![AJAX Data Trace]({{ site.baseurl }}/images/2013-04-13-1.png)

It took a few nights of looking at the data before realizing how to parse it, but after you get the hang of how it's formatted, its quite simple. The issue I had was with my X,Y mapping. I wrote a quick app to download and parse the data, but when I tried to map the data how it was presented, it didn't make much sense when you compare it to what actually happened in the game.
![Cartegian plane]({{ site.baseurl }}/images/2013-04-13-2.png)

It should have looked like this.
![ESPN Rendering]({{ site.baseurl }}/images/2013-04-13-3.png)

Silly oversight on my part - X and Y coordinates are relative to where the goal/shot/whatever took place, and not absolute like they would appear to be in the above depiction. Simply reflect the X points over the X axis, and the Y points over the Y (depending upon the team) to make them absolute with respect to a certain area of the ice.

My initial idea was just focused on gathering data for the Pittsburgh Penguins (my team of preference...Let's go Pens!). I'm still in the process of deriving some tangible data from the data that's out there, but at first glance, this stood out to me.
![Cartegian populated]({{ site.baseurl }}/images/2013-04-13-4.png)

The above is a graph of all shots scored in the 2013 season that our data source has available. I have to preface this as not all data was available on the site - there were probably around 8 games where there was no XML data available for us to parse. The goal mark is around the (-95, -5 through 5). You can immediately see, the the large majority of goals coming from within the -75 X coordinate (which equates to the 10ft mark).

Be on the lookout for another post in the near future where I'll (attempt) to develop some trends from this seasons data.

As a footnote, I wasn't able to find anything on the internet about this when I was developing it, so I figure I'd share my success. These are my notes from the data I was able to gather from the XML file that the Flash player is downloading. Happy to answer questions about it, as it took me a few looks to get it just right. I should also mention that these lines should be split using the '~' character; each field will mean something, but tracking down exactly what that something is for each entry would be very tough, and frankly, not needed.

- The "Game" XML node has some high level data about the game being played and the teams involved such as the date/time, teams playing, home/away tream. Sample node as follows:
    - `<Game id="400443012"> <![CDATA[ 3~1~6~3~0~7:30 PM~Apr 3~April 3, 2013~Madison Square Garden~New York~New York~27~39~0~1~0~ ~3~2~1~ ~13~New York~Rangers~nyr~0x0B3D91~16~Pittsburgh~Penguins~pit~0xC3B263~2~(28-10-0, 56 pts)~(18-15-3, 39 pts)~Series starts 1/20 ]]> </Game>`
- The "Player" node has data about some of the players where a "Play" node has been referenced. This is key as the "Play" nodes reference these internal player ID's for instances where a reference is needed. For example, if the below player, Arron Asham, were to score a goal (however unlikely :p ), he would have a "Play" entry and a "Player" entry as well so there is no need for external API (or whatever) calls to pull things like his head shot.
    - `<Player id="f24"> <![CDATA[ 24~1822~Arron Asham~45~13~RW~http://somewebsite.com/i/headshots/nhl/players/35/24.jpg~0~0~0~0~531~0~1~0~0~7~2~0~2~2~7875~11~30~0~0~24 ]]> </Player>`
- The one where most of the magic happens is the "Play" node. For each NHL hockey play (eg., shot, goal, blocked shot, stoppage, etc.) one entry is logged into this file. Breakdown of the entry is as follows. Numbers referenced are the zero-based index of the parameter in question.
    - `<Play id="4004430120000681"> <![CDATA[ 58~8~506~19:55~3~2179~0~0~Shot on goal by Chris Kunitz saved by Henrik Lundqvist(Snap 32 ft)~0~6~1~0~701~16~802~901~20~0~0 ]]>`
    - Item[0] - X Coordinate for where the Play took place- remember that this is absolute and would change depending upon the period of play (as teams switch sides)
    - Item[1] - Y Coordinate of where the Play took place - same footnote as [0]
    - Item[2] - What type of play was it? An `enum` value for the type of play. I didn't look for all of them, but these are the ones I cared about
        - 502 = faceoff win
        - 503 = hit
        - 505 = goal
        - 506 = shot
        - 507 = missed shot
        - 508 = shot blocked
        - 512 = penalty
        - 502 = faceoff
        - 516 = stoppage (iceing)
        - 516 = goal stoppage
        - 1401 = takeaway
        - 1402 = giveaway
    - Item[3] - How far into the period did this Play happen?
    - Item[4] - What period did it happen?
    - Item[5] - What was the primary player ID for the player performing the action?
    - Item[6] - The secondary action on the Play (eg., 1st assist on a goal)
    - Item[7] - The tertiary action on the Play (eg., 2nd assist)
    - Item[14] - The internal team ID value - Can be cross referenced against the Game node.