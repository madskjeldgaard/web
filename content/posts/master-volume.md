---
title: "A midi controller in a box of screws"
date: 06-02-2021
draft: false
images:
- /img/full/screws1.jpg 
- /img/full/screws2.jpg
tags:
- supercollider
- cpp
- teensy
---

![box of screws controller](/img/full/screws1.jpg)
![box of screws controller](/img/full/screws2.jpg)


I recently built version 2 of a "master volume control" midi controller that I had made before. This version is slightly improved on the former. 

It is very cheap and simple. It revolves around the wonderful and cheap (11$) Teensy LC.

Bill of materials includes:
- A 10k potentiometer
- A button
- A [Teensy LC](https://www.pjrc.com/teensy/teensyLC.html)
- A box (I used a leftover box that had contained some wood screws - perfect size!)
- A few wires

All [code and notes for building may be found here](https://github.com/madskjeldgaard/MasterVolumeController).

## SuperCollider example

To use this controller as the master volume controller in SuperCollider you may copy and paste the following code to your startup file in SuperCollider.

```
//------------------------------------------
// Server volume set by external controller
//------------------------------------------

Server.local.waitForBoot{

	// Adjust these to match the Teensy code
	var potentiometerCC = 7;
	var muteNoteNum = 44;
	var midichan = 0;

	// Adjust this if device sourceid varies
	var deviceSrcID = 1572864; 

	// ----------------

	// Connect midi
	MIDIIn.connectAll();

	// Setup responder for master volume
	MIDIdef.cc(\masterVolume, {|...args| 
		var value = args[0];
		var volume = (value / 128.0).ampdb;

		"Server volume: % db".format(volume.round(0.01)).postln;

		Server.local.volume = volume;
	}, 
		chan: midichan,
		ccNum: potentiometerCC, 
		srcID: deviceSrcID
	).fix; // Make permanent between hardstops

	// Mute button
	MIDIdef.noteOn(\masterMute, {|...args| 

		if(Server.local.volume.isMuted, {
			"Unmuting...".postln; 
			Server.local.unmute 
		}, {
			"Muting...".postln; 
			Server.local.mute 
		})

	}, 
		noteNum: muteNoteNum, 
		chan: midichan,
		srcID: deviceSrcID
	).fix; // Make permanent between hardstops

};

```
