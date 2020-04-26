---
title: "SoX tutorial: SoX on Android"
date: 2020-04-26T10:21:42+02:00
draft: false
toc: true
images:
tags:
  - sox
---

![sox on android](/img/small/sox-android-2.jpg)

In this tutorial, I will cover how to install and setup SoX on android devices using Termux.

Termux is a free &#8220;Android terminal emulator and Linux environment app that works directly with no rooting or setup required&#8221;.

Basically it is a command line interface for your Android device and works like a small linux distribution. It even includes a package management system. And if you get something like an OTG-dongle you can even connect a keyboard and/or a class compliant sound interface.

## Setup

The first think you need to do is install [termux][1] on your phone. It can be installed either via PlayStore or F-Droid.

Before continuing, it may be a good idea to make sure Termux has the proper storage permissions.

According to the [termux wiki][2]:
  
&#8220;It is necessary to grant storage permission for Termux on Android 6 and higher. Use &#8216;Settings>Apps>Termux>Permissions>Storage&#8217; and set to true.&#8221;

## Installing sox

To install a package, first update the information available about the package repos

    pkg update
    

And then install SoX by executing

    pkg install sox
    

Hopefully, you now have sox installed on your Android device.
  
You can verify this using the `which` command, if it returns nothing, something is wrong, if it returns a path, you should be good.

    which sox
    

## Audio input

To use your Android device&#8217;s internal microphone, you need to tweak a few things.

By default, Sox uses the pulseaudio driver to get sound in and out of Termux but the audio input is not activated by default.

Let&#8217;s fix this.

First, kill pulseaudio if you have it running.

    pulseaudio -k
    

Then, restart pulseaudio and load the &#8220;module-sles-source&#8221; module.

The `-D` flag will daemonise/background pulseaudio.

    pulseaudio -L "module-sles-source" -D
    

That&#8217;s it, now you can record sound using sox.

Let&#8217;s test it by recording 5 seconds of audio, you should see the &#8220;meter&#8221; on the right react when you make noise.

    rec yay.wav trim 00:00 00:05
    

And then play it back

    play yay.wav
    

[<img class="alignnone size-large wp-image-847" src="https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430-1024x768.jpg" alt="" width="640" height="480" srcset="https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430-1024x768.jpg 1024w, https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430-300x225.jpg 300w, https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430-768x576.jpg 768w, https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430-1536x1152.jpg 1536w, https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430.jpg 2000w" sizes="(max-width: 640px) 100vw, 640px" />][3]

* * *

This tutorial is part of series of tutorials:

<div class="display-posts-listing">
  <div class="listing-item">
    <span class="dps-portfolio-list-item"><a class="dps-title entry-title" id="no-decoration" href="https://www.madskjeldgaard.dk/sox-tutorial-command-line-tape-music-an-introduction/">SoX tutorial: Command line tape music (an introduction)</a></span>
  </div>
  
  <div class="listing-item">
    <span class="dps-portfolio-list-item"><a class="dps-title entry-title" id="no-decoration" href="https://www.madskjeldgaard.dk/sox-tutorial-split-by-silence/">SoX tutorial: Split by silence</a></span>
  </div>
  
  <div class="listing-item">
    <span class="dps-portfolio-list-item"><a class="dps-title entry-title" id="no-decoration" href="https://www.madskjeldgaard.dk/sox-tutorial-batch-processing-audio-on-the-command-line/">SoX tutorial: Batch processing audio on the command line</a></span>
  </div>
  
  <div class="listing-item">
    <span class="dps-portfolio-list-item"><a class="dps-title entry-title" id="no-decoration" href="https://www.madskjeldgaard.dk/sox-tutorial-using-sox-on-android-using-termux/">SoX tutorial: SoX on Android (using Termux)</a></span>
  </div>
</div>

&nbsp;

 [1]: https://termux.com/
 [2]: https://wiki.termux.com/wiki/Termux-setup-storage
 [3]: https://www.madskjeldgaard.dk/wp-content/uploads/2020/03/IMG_2430.jpg
