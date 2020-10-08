---
title: "Raspberry Pi 4: Simple Button to OSC example in Rust"
date: 2020-06-11T16:49:29+02:00
draft: false
tags:
- rust
- raspberrypi
- osc
---

Today I did a small experiment with my Raspberry Pi 4: I wanted to poll the GPIO data pins and use data from them in SuperCollider. This is best done by polling the pins in a separate program and then sending that data to SuperCollider via OSC.

I have done this in Python before but I much prefer doing this kind of thing in Rust since the latter is fast and safe (and just generally: I love Rust!).

I concocted a small example of this. 

[https://github.com/madskjeldgaard/gpio-osc-button](https://github.com/madskjeldgaard/gpio-osc-button)
