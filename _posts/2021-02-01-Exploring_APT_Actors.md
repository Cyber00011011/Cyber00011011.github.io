---
layout: post
title: Exploring APT Actors
categories: [Malware, APT]
---
(Estimated Reading Time: 6 minutes)

- [APT Actors](#apt-actors)
- [Who are APT Actors](#who-are-apt-actors)
- [References](#references)

## APT Actors

The term [APT or Advanced Persistance Threat](https://en.wikipedia.org/wiki/Advanced_persistent_threat) is used to refer mainly to Nation States but more recently can also include well funded organized crime groups conducting large scale intrusions. In this blog post we are going to look at the [list of APT Groups published by Mitre](https://attack.mitre.org/groups/) to understand what countries are most associated with APT Groups, as of Feb-2, 2021.  

## Who are APT Actors

I took data from the Mitra list of APT Groups and dropped that into excel. With a little clean up of the data I had 123 groups list. I then added a new colum for most likely country of origin and populated the column from reading the threat Actor description. I validated my thinking by also checking the Malpedia list of actors, and Fireeye's list of APT actors. When I was all said and done, I did a simple GroupBy Country to see the top countries associated with APT Threat actors. There is a large number of groups where the Country of origin is unknown. When I was looking over the unknown groups they largely seemed to be organized crime groups, verses a nation state.

![APT Actors](/images/apt_image.PNG)

> Top Countries Associated with Malicous activity; China, Iran, Russia and North Korea.  

## References
* Link to the [Excel file](/files/APT_Stats.xlsx) I created from the Mitre data. 
* Link to [Mitre APT Group page](https://attack.mitre.org/groups) - 123 Actors tracked, as of Feb-2, 2021
* Link to [Malpedia APT Group page](https://malpedia.caad.fkie.fraunhofer.de/actors) -  332 actors tracked, as of Feb-2, 2021
* Link to [Fireeye APT Group page](https://www.fireeye.com/current-threats/apt-groups.html)