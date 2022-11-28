---
layout: "../../layouts/Project.astro"
title: "Smart Vents"
description: "ESPHome based smart ventilation system with manual and automatic targeted boost"
pubDate: "2022-11-22T12:33:04Z"
updatedDate: "2022-11-19T12:33:04Z"
---

1. [Overview](#overview)
1. Features
1. Ducts
1. Smart Vents
1. Controller
1. Conclusion

## Overview

__Automatic, boosted, targeted, smart ventilation system for whole home ERV__

We started our major home renovation just prior to pandemic. So my "pandemic project" ended up being much more substaintial than anticipated. I did all sorts of modern building science upgrades during the reno. One such upgrade involved adding ductwork and vents to all major rooms in the house. Although I had my HVAC company quote the installation, I took the effort on myself.

My goal was an ERV with boost capability with targeted stale air exhaust and fresh air supply. I landed on the following vents:

<div style="font-size:0.9rem">

| Room                 | Type               | Goal
| ---------------------| -------------------| -------------------------------------
| Kitchen              | Stale Exhaust      | Automatic boost when cooking
| Living               | Fresh Supply       | -
| Office               | Stale Exhaust      | Automatic boost when 3D printing
| Master Bath          | Stale Exhaust      | Automatic boost when showering
| Powder Toilet        | Stale Exhaust      | Automatic boost when in use
| Master Toilet        | Stale Exhaust      | Automatic boost when in use
| Master Bedroom       | Fresh Supply       | -
| Guestroom            | Fresh Supply       | -
| Bedroom South        | Fresh Supply       | -
| Bedroom North        | Fresh Supply       | -
| Upstairs Bath        | Stale Exhaust      | Automatic boost when occupied
| Crawlspace           | Stale Exhaust      | Automatic boost when poor AQ

</div>

## Features

* Central control module
* Stale-air monitoring
* Local-only control
* Home-assistant ready (but not required)
* Multi-boost capable
* Manual & auto boosting

## Smart Vent

The smart vents are for stale air returns so you can target bad air and get it out of the house. Although targeting fresh-air vents could be useful in certain scenarios, it isn't as important.

<img style="max-width:340px;float:right" src="/img/smart-vents/vent-elbow-example.webp" />

During construction, I ran a 4" duct to all locations listed above. I utilized a combination of:

* 4" Flexible duct commonly used for dryer venting (less efficient)
* 4" Straight ducts for longer runs (more efficient)
* 4" Low profile platic elbow (Fits in 2x4 Wall)
* Cat5e cable to end of each opening
* Custom 3D printed cable holder

At the end of each duct is a wall boot

### ESPHome DIY Module
