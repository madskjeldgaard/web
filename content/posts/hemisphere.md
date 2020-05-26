---
title: "Control voltage and SuperCollider: Using the open source module Ornament & Crime as a midi-cv interface"
date: 2020-05-24T10:10:40+02:00
draft: false
toc: true
tags: 
- supercollider
- ornament-and-crime
- open-source
- modular-synthesis
- eurorack
---

![oc](/img/small/oc_frontside.jpg)
_My Ornament & Crime build (the Mini OC version). The USB connector on the Teensy on the back of the module has been rewired to a Neutrik USB connector attached to a panel next to it (the grey panel)._

I have tried quite a few different approaches to integrating my modular synthesizer with my SuperCollider workflow and I have found most them to be disappointing, extremely expensive and or inflexible.

But recently, I built the [Ornament & Crime](https://ornament-and-cri.me/) eurorack module and have been very satisfied using it as a midi-cv interface with four cv inputs and four cv outputs.

{{<vimeo 421912594>}}

This method has some advantages: The module and the firmware are both open source - this makes it a very sustainable choice because the community is constantly improving and expanding the possibilities with this module. This also means when you are not using it as a midi-cv interface you can pull up any of the many different applications it comes with and use it for something else. It's also fairly cheap to make yourself and the build process is - due to it's popularity and the great community - [very well documented](https://ornament-and-cri.me/build-it/). The disadvantage of this approach is that communication between the computer and the modular happens over midi with a value range limited to 0-127, but this may be smoothed at both ends of the signal.

## Ornament & Crime
Ornament & Crime is an open source eurorack module that serves as a DAC and ADC for [the Teensy microcontroller](https://www.pjrc.com/store/teensy32.html). The module has become highly popular in the DIY world and has inspired a range of derivatives in hardware (which even includes [versions for the Buchla synthesizer system](https://northernlightmodular.com/)). The hardware and firmware for it was largely inspired by the incredible work of Émilie Gillet at [Mutable Instruments](https://mutable-instruments.net/), a synthesizer company with a radical open source philosophy and very free licensing for both hardware and software.

The reason for the OC's popularity can probably be traced back to the core of the module: The Teensy. With this (which is basically a very nice sort of Arduino that works really well for audio tasks), anyone can plug a USB cable into the module and start hacking at the firmware as if it was any other Arduino.

The firmware for the OC consists of a range of small applications [serving purposes from signal quantization to low frequency oscillators](https://ornament-and-cri.me/firmware/), some of which are taken from the Mutable Instruments modules mentioned above. 

## Hemisphere Suite: Turning the module into a USB midi controller
However, recently an alternative to the "stock" firmware for the OC has appeared: [The Hemisphere Suite](https://github.com/Chysn/O_C-HemisphereSuite/wiki/Captain-MIDI). This firmware is an entirely different suite of applications.

One of these is [Captain MIDI](https://github.com/Chysn/O_C-HemisphereSuite/wiki/Captain-MIDI) which is what allows us to do midi-cv conversion between the module and our computer by plugging a USB cable into the Teensy itself on the back of the board. This turns the module into a USB midi interface recognized on your computer as any kind of USB midi controller. 

![oc](/img/small/oc_backside.jpg)

Note: Make sure you have at least v.1.8A installed - the previous versions of Captain MIDI had a nasty buffer overflow bug making the module crash at high load.

## Controlling the module with SuperCollider

Since the module is now recognized as a midi controller by the computer, we can address it as such in SuperCollider. I use [The Modality Toolkit](https://github.com/ModalityTeam/Modality-toolkit) for all of my SuperCollider work involving controllers. The Modality package is basically a large library and framework for doing amazing things with controllers by using them in a _modal_ way making interface setup and reconfiguration fairly easy. It comes with support for a bunch of controllers, and the list of supported ones is constantly growing.

I added support for the Ornament & Crime module with the Hemisphere Suite installed. 

To use it, install or update Modality in SuperCollider: 
``` 
Quarks.gui;
```

### Setting up Captain MIDI on the Hemisphere

On the module, open up the app "Captain MIDI" and enter the setup screen.

The Hemisphere/Captain MIDI description in Modality assumes that you are using it with 4 Mod midi signals going in and out. Set this up on the module like so:

In the screen that says "MIDI Assign" at the top, change parameters to this:

``` 
Screen: MIDI Assign
MIDI > A 	Mod
MIDI > B 	Mod
MIDI > C 	Mod
MIDI > D 	Mod
1 > MIDI 	Mod
2 > MIDI 	Mod
3 > MIDI 	Mod
4 > MIDI 	Mod
```

And on the setup page called "MIDI Channel":

```
Screen: MIDI Channel
MIDI > A 	1
MIDI > B 	2
MIDI > C 	3
MIDI > D 	4
1 > MIDI 	1
2 > MIDI 	2
3 > MIDI 	3
4 > MIDI 	4
```

### Making contact

Plug your USB cable into the back of the Teensy and in to your computer. 

Then, in SuperCollider you can init the module as a controller:
```javascript
m = MKtl('captain', "captain-midi")
```

One of the nice things about Modality is that it automatically creates GUI interfaces so that you can experiment with the controllers even though they are not present.

Open up the GUI (and press the labels button to make it clear what is what):
```javascript
m.gui;
```

Now you can either move the sliders for the outputs or run this code which will randomly change the four outputs on the module to random values

```javascript
// Test outputs
Tdef(\outTester, { 
	loop{
		var out = ['a', 'b', 'c', 'd'].choose;
		var val = rrand(0.0,1.0);
		"Setting output % to %".format(out, val).postln;
		m.elAt('out', out).value_(val);
		0.125.wait;
	}
}).play;
```

Alternatively, set them manually

```javascript
// Manually set outputs
m.elAt('out', 'a').value_(0.2);
m.elAt('out', 'b').value_(0.7);
m.elAt('out', 'c').value_(0.1);
m.elAt('out', 'd').value_(0.9);
```

Now, plug signals into the four inputs on the module and have them control four feedback sine wave synths in SuperCollider:

```javascript
// Map cv inputs 
['1', '2', '3', '4'].do{|in, in_num|
	Ndef(in, {|freq=424, feedback=0.65, amp=0.5, pan=0| 
		Pan2.ar(SinOscFB.ar(freq, feedback, amp), pan)
	}).play;
	
	m.elAt('in', in).action_({|el|
		var value = el.value;
		Ndef(in).set(\freq, value.linexp(0.0,1.0,40.0,5000.0))
	});
};
```
