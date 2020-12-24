---
layout: post
title:  "Baseline for My Tutorials"
date:   2020-04-05 18:30:34 -0600
categories: tech tutorials
---

I have two Raspberry Pi 3’s running PiHole, because redundancy, right? And of course in the home lab I’m interested in what metrics I can find and display. That’s also part of my primary role at work, so at home I get to play and explore, and then I get to go to work and succeed and impress people.

Over the past couple of days while thinking of what & how I wanted to write posts about how-to or step by step things, I developed the idea of what I visualize something similar to how Git works. I don’t want to repeat content. I want to be able to have a post that is very specific, and if it requires some prior knowledge or configuration then that should be a separate post linked from the more advanced one.

Take this one for example: Getting metrics from PiHole into Grafana. What needs to be in place first?

* Grafana
  * How to install and configure Grafana
  * InfluxDB
    * How to install and configure InfluxDB
* PiHole
  * How to install and configure PiHole
  * How to install Raspian OS
    * How to load Raspian onto a new micro SD card.

Each one of those should be a stand alone post. Then later on, when I write about monitoring VPN sessions on a Palo Alto with Grafana, that baseline for InfluxDB and Grafana are already there!

Right, so what’s first? Well, I’ve got my primary PiHole – LIT-Pi1 – in a happy state. It has a 32GB SD card, the latest Raspian OS, RetroPi, and not much else. My secondary PiHole – LIT-Pi2 – is still on an older 16GB SD card, but it’s doing the metrics collection for both of them.

So first off, I’m going to write down everything I remember about what I did to get that all working in its current state. Then I’ll swap the cards and build it back up, get that write-up perfect, and then it’ll be up here.

Stay tuned!

-Kevin
