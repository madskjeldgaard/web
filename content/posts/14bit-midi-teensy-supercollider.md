---
title: "Working with 14bit Midi using a Teensy microcontroller and Supercollider"
date: 2020-10-21T15:48:28+02:00
draft: false
images:
- /img/full/14bitteensybb.jpg
tags:
- teensy
- supercollider
- microcontrollers
- midi
- 14bitmidi
---
![Teensy on a breadboard](/img/full/14bitteensybb.jpg)

Lately I have been experimenting with 14bit MIDI and found it to be a huge revelation for my work with SuperCollider. Often, the regular data range of MIDI (0-127) is way too choppy and coarse for nice interactions with your systems and instruments in SuperCollider, but 14 bit MIDI provides a resolution of 16384 steps which is great. This is actually achieved quite cleverly by combining two MIDI CC signals in to one.

There are probably some commercial MIDI controllers out there that provide 14 bit MIDI by now but I have experimented with it using the trusty [Teensy 3.2 microcontroller](https://www.pjrc.com/store/teensy32.html). Among the many nice features of this is the fact that it has native support for USB MIDI which works great out of the box and so with this and a couple of analog potentiometers you have yourself a cheap and simple high-resolution 14bit MIDI controller. The only problem with the Teensy is that it only has a 13 bit ADC (Analog to digital converter) meaning the resulting readings will only be in a resolution of 8192 steps but I find that it is more than adequate and you just have to shift it up (see the code below) to fill out the whole spectrum available.

## Microcontroller code

The code is well documented so please read the comments for details. 

The core concept here is that the 14 bit MIDI is achieved by sending two MIDI cc values at the same time. One "low" MIDI cc and one "high" MIDI cc value which is 32 MIDI CC's above it. These are then combined at the receiving software to become a 14 bit MIDI signal.

[Source code is here](https://github.com/madskjeldgaard/14bit-midi-teensy-supercollider).

```cpp 
// main.cpp
#include "Arduino.h"
#include <ResponsiveAnalogRead.h>

#define LEDPIN 13
#define POT_PIN1 14
#define POT_PIN2 15

/* Set analog resolution */
#define RESOLUTION 8192

// 14 bit midi is achieved by sending two midicc signals, one "low" combined
// with one 32 midicc's later called high
// Here we just define the lower ones
#define LOWCC1 16
#define LOWCC2 17

#define HIGHCC1 LOWCC1 + 32
#define HIGHCC2 LOWCC2 + 32

/* Pot value storage */
int pot1val = 0;
int pot2val = 0;

/* Set up read pins */
ResponsiveAnalogRead pot1(POT_PIN1, true);
ResponsiveAnalogRead pot2(POT_PIN2, true);

void setup() {
  /* resolution */
  analogReadResolution(13);

  /* smooth input values */
  analogReadAveraging(16);

  /* Set up smooth potentiometer readings */
  pot1.setAnalogResolution(RESOLUTION);

  // Enabling sleep will cause values to take less time to stop changing and
  // potentially stop changing more abruptly, where as disabling sleep will
  // cause values to ease into their correct position smoothly.
  // On by default
  /* pot1.disableSleep(); */

  // edge snap ensures that values at the edges of the spectrum (0 and 8192) can
  // be easily reached when sleep is enabled
  // On by default
  /* pot1.disableEdgeSnap(); */

  pot2.setAnalogResolution(RESOLUTION);

  /* Turn internal led on on boot */
  pinMode(LEDPIN, OUTPUT);
  digitalWrite(LEDPIN, HIGH);
}

void readPots() {
  pot1.update();

  if (pot1.hasChanged()) {
    // Shift up from ADC's 13 bits to Midi 14 bit
    pot1val = pot1.getValue() << 1;

    usbMIDI.sendControlChange(HIGHCC1, pot1val & 0x7F, 1);
    usbMIDI.sendControlChange(LOWCC1, (pot1val >> 7) & 0x7F, 1);
  }

  pot2.update();
  if (pot2.hasChanged()) {
    pot2val = pot2.getValue() << 1;

    usbMIDI.sendControlChange(HIGHCC2, pot2val & 0x7F, 1);
    usbMIDI.sendControlChange(LOWCC2, (pot2val >> 7) & 0x7F, 1);
  }
}

void loop() {
  readPots();
  delay(10);
}
```

## Testing it out in SuperCollider

SuperCollider does not at the time of writing have built in support for 14 bit MIDI, but there are several external libraries you may install to achieve such a result.  One of the simplest solutions is Carl Testa's `FourteenBitCC` class which may be [downloaded here](https://gist.github.com/carltesta/bb5065a7b92bab7673237e9cc1c9a612) and placed in your SuperCollider extensions folder.

Here is an example of how to use it:

```javascript
/*********************************/
/* Using the FourteenbitCC class */
/*********************************/
// Get it here: https://gist.github.com/carltesta/bb5065a7b92bab7673237e9cc1c9a612
(
MIDIIn.connectAll;

~x = FourteenBitCC.new("x", 16, 48);
~x.func = {|val| 
	var bits = 13,
	maxval = (2**bits-1);

	Ndef(\s).set(\carfreq, val.linexp(0, 2**13, 50.0,15000.0));
	("x: "++val).postln
};

~y = FourteenBitCC.new("y", 17, 49, 0);
~y.func = {|val| 
	var bits = 13,
		maxval = (2**bits-1);

	Ndef(\s).set(\freq, val.linexp(0, maxval, 50.0,5000.0));
	("y: "++val).postln;
};

Ndef(\s, {|freq=444, carfreq=919| SinOsc.ar(freq * SinOsc.ar(carfreq))*0.5!2}).mold(2).play;
)
```

Another reasonable choice is the [Modality Toolkit](https://github.com/ModalityTeam/Modality-toolkit) library which is normally my go to for anything controller based in SuperCollider. It has 14bit midi support but it is at the time of writing this not documented and not 100% implemented, but the following example should work fine:

```javascript
/*********************************/
/* Using Modality 				 */
/*********************************/
(
~descInput = (
	idInfo: "Teensy MIDI",
	deviceName: "Teensy MIDI",
    protocol: \midi,
    elementsDesc: (
        elements: (16..20).collect{|lowerMidiCC|
			(
                key: "kn%".format(lowerMidiCC).asSymbol,
                type: 'knob',
                spec: [0,16383,\lin,1,0], // Custom spec needed for the moment since a midiCC14 spec doesn't exist yet
                midiMsgType: \cc14,
                midiChan: 0,
                midiNum: lowerMidiCC,
                ioType: \in
            )
		}
    )
);

m = MKtl( \testMIDI, ~descInput);
m.elAt('kn16').action = {|el| "Touched kn16: %".format(el.value).postln};
m.elAt('kn17').action = {|el| "Touched kn17: %".format(el.value).postln};
);
```



