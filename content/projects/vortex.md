---
title: "Vortex: A system for turbulent tape music"
date: 2020-05-22T10:56:16+02:00
draft: false
toc: true
---

Vortex is an open source computer music system by [Mads Kjeldgaard](https://madskjeldgaard.dk) for improvisation and composition. It is written as a package for the [SuperCollider programming environment](https://supercollider.github.io/).

It is inspired by reel to reel tape recorders, cybernetic and feedback music as well as more contemporary generative systems.

At it's core it has a complex web of sound processes divided in voices, all of them interconnected in feedback paths and outputting their sound to digital looping varispeed tape reels.

The name is meant to illustrate the fact that there is a continuous, _turbulent_ flow and that the sounds and movements of the performer disappear in this fluid motion.

![candle flow](https://upload.wikimedia.org/wikipedia/commons/0/03/Laminar-turbulent_transition.jpg)_"the thermal convection plume rising from an ordinary candle in still air. The plume is initially laminar, but transition to turbulence occurs in the upper 1/3". [Image by Dr. Gary Settles, CC-by-sa-3.0.](https://commons.wikimedia.org/wiki/File:Laminar-turbulent_transition.jpg)_

## Background

This project builds on a conglomeration of inspiration. 
For years I've been fascinated with the music and ideas of cybernetic music composers such as Jaap Vink and Roland Kayn which is something that has lingered in me since discovering [this particular record by Vink](http://editionsmego.com/release/REGRM018EXT) during a heat wave in Paris: I would sit inside, dripping with sweat and in a sort of haze from the heat, completely dazzled by the slowly mutating sounds of Vink and the way they sort of seemed to move through my cranium. 

Vink and Kayn worked with [cybernetics](https://en.wikipedia.org/wiki/Cybernetics): They would set up enormous (physical) systems with feedback as a central idea; feedback in both sound and control (the latter by patching cords in synthesizers and sound modules) to create complexities in sound on a microscopic, granular scale and on larger time scales. 

{{< bandcamp-album 1070203104 >}}

This resonated deeply with me along with a want for a system that would be able to programmatically generate such small and large scale structures in composition and performance from simple gestures in sound and control. Or even better: To have a system which seems to have a will of it's own, a vibrating structure which may be affected or disturbed by the performer (me) but never fully controlled, leading to a range of surprises. In my piece [States of Emergency](https://shop.conditional.club/album/states-of-emergency) I achieved this somewhat by manually performing recursive sound effect transformations controlled by complex low frequency oscillators - iterating over the results for days on end in a DAW, this process became laborous and I started dreaming of automating this process to be able to setup and define such systems more easily. 

{{< vimeo 238351530 >}}

A few years later I participated in a workshop with [Jérôme Noetinger who works with reel to reel tape machines as instruments](https://www.youtube.com/watch?v=pnZ55jQe8jA) for improvisation by setting up tape loops, (re)injecting sounds into them and varying their playback speed. Working with the materiality of the sound in this way really fascinated me and so this core concept of a digital varispeed tape loop is at the center of the Vortex system. Other inspirations for this part of the project are [Eliane Radigue's feedback works](https://youtu.be/C_3Fu8YfSdI), [Giovanni Lami's outdoors tape loop sessions](https://vimeo.com/238351530) as well as [Valerio Tricoli's tape loop live performances](https://www.youtube.com/watch?v=J7nDhTk705s). 

With these inspirations in mind, the question becomes: Is it possible to design a system which allows to work on both small and large scale sonic structures, incorporating extreme sound processing that is easily influenced but impossible to control? 

## Design goals
![controllers](https://raw.githubusercontent.com/madskjeldgaard/vortex/master/documentation/vortex_controllers.JPG)

- **You will never master this instrument**: Each time the system is used, it is reconfigured in a new and slightly random way making it necessary to relearn it and find new sweet spots.
- **Gestures have unforeseable consequences**: Whatever gesture you put into the system (through code or controllers) should have consequences that are neither random nor predictable, like a [Lorenz system](https://en.wikipedia.org/wiki/Lorenz_system). 
- **Multiple time scales**: The same input gestures cause reactions in sound and control now, in the near future and on a massive time scale.
- **No sound source**: The system only processes sounds, forcing the user to input sound.
- **No control, only influence**: The system cannot be fully controlled, only influenced.
- **Channel agnostic**: No usage differences between stereo and multichannel outputs.
- **Interface agnostic**: May be used using code, patterns and or any physical controller. The system/instrument as an API.
- **Extremely dynamic**: The whole system may be reconfigured and repatched by executing a command or pushing a button. 

## Technical details

{{<mermaid>}}
graph TB

subgraph vortex
Voice1 --> Voice2
Voice1 --> Voice3

Voice2 --> Voice1
Voice2 --> Voice3

Voice3 --> Voice1
Voice3 --> Voice2
end

SI{SoundIn}

SI --> Voice1
SI --> Voice2
SI --> Voice3

C[control input]
C -.-> Voice1
C -.-> Voice2
C -.-> Voice3

SO{SoundOut}
Voice1 --> SO
Voice2 --> SO
Voice3 --> SO

{{</mermaid>}}

Vortex itself is an interface for controlling several VortexVoices. It allows to set these voices up in feedback loops, interconnect them and maintain a common control interface for them all. 

Each VortexVoice can be thought of as a varispeed tape loop with extreme processing, feedback and an esoteric control interface.


### Source code

Vortex is designed as an API so it is easy to interact with.

It is completely open source. It is written using the SuperCollider programming language and may easily be forked and instrospected. You can [find the source code here](github.com/madskjeldgaard/vortex).

It is [licensed under GPL3.0 like SuperCollider itself](https://github.com/madskjeldgaard/vortex/blob/master/LICENSE.md).

### Controlling vortex
Vortex may be controlled by live coding with it or using physical controllers like MIDI controllers, OSC devices and HID devices. 

Controlling it using controllers is made easy thanks to the [Modality-toolkit project](https://github.com/ModalityTeam/Modality-toolkit).

Vortex may also be controlled by livecoding the voices or using custom event types for SuperCollider patterns (work in progress).

{{<mermaid>}}
graph TB
G{sound in} --> E

subgraph GESTURES
A[performer] --> B[keyboard/code]

A --> C[controllers]
end
B --> D(multiplexer)
C --> D

D --> E(vortex voice)
D --> E
D --> E
D --> E
D --> E
D --> E

E --> F{sound out}
{{</mermaid>}}

### Multiplexing control data: VortexFlux
Input data is multiplexed by an algorithm called VortexFlux (based on Alberto de Campo's Influx package) which makes one or a few inputs become many outputs (the exact number may be decided upon initialization). 

For more info on this approach and Influx, [this article explains the idea nicely](https://www.3dmin.org/research/open-development-and-design/influx/).

It then passes those many outputs through further data warping algorithms to increase the complexity of the control data and further distance them from the gesture put in to the system. 

The steps are :
1. Weighting: Multiply the input with a random weight
2. Envelopes: Use the input of the previous as an index into a random envelope. This works sort of like a control rate wavetable.
3. LFO: The input of the previous is used to control a network of lfos. This step is optional. Additionally, the lfo network may be configured in a feedback mode with the individual lfos in the network controlling each other's frequency alongside the control input.
4. Scale the data using presets to center around sweet spots.
5. Map the multiplexed control data to the parameters of a VortexVoice.

{{<mermaid>}}
graph TB

M((multiplexer)) 
C((control)) 

W1((in * w))
W2((in * w))
W3((in * w))
W4((in * w))

E1((env))
E2((env))
E3((env))
E4((env))

L1((lfo))
L2((lfo))
L3((lfo))
L4((lfo))

P((Preset))
SC1((scale))
SC2((scale))
SC3((scale))
SC4((scale))

O{voice}

C --> M
M --> W1 --> E1 --> L1
M --> W2 --> E2 --> L2
M --> W3 --> E3 --> L3
M --> W4 --> E4 --> L4

subgraph "optional: lfo network"
L1 -.-> L4
L2 -.-> L1
L3 -.-> L2
L4 -.-> L3
end

subgraph "Scale using preset center points and bandwidth"
P -.-> SC1
P -.-> SC2
P -.-> SC3
P -.-> SC4
end

L1 --> SC1 --> O
L2 --> SC2 --> O
L3 --> SC3 --> O
L4 --> SC4 --> O

{{</mermaid>}}

### Closeup of LFO mapping
{{<mermaid>}}
graph TB
IN[Control]
INX[Xfade amount]
X[Crossfade control]
FB[LFO Feedback]
OUT
OSC[VarSaw]
SC[Scale]

IN -- 0.0-1.0 --> SC -- 0.001-100.0hz: Frequency --> OSC
INX -. 0.0-1.0 .-> X
FB -. 0.0-1.0: Shape .-> OSC 
OSC -- 0.0-1.0 --> X
IN --> X --> OUT
{{</mermaid>}}

### TODO: preset scaling
{{<mermaid>}}
graph TD
IN[Control]
OUT

P(Preset)

C[ctrl: Center point]
BW[ctrl: Bandwidth]
S[Scaler]
P -.-> C
P -.-> BW
C-- 0.33 --> S
BW-- 0.1 --> S

S -- range: 0.23-0.43 --> OUT
IN -- range:0.0-1.0 --> S
{{</mermaid>}}

### Example of LFO feedback network

This example illustrates an lfo feedback network in a VortexVoice. In this example there are 32 lfos, each matching up with the output of a VortexFlux instance.

{{<mermaid>}}
flowchart TB
LFO0 -. width .-> LFO17
LFO1 -. width .-> LFO5
LFO2 -. width .-> LFO20
LFO3 -. width .-> LFO8
LFO4 -. width .-> LFO17
LFO5 -. width .-> LFO28
LFO6 -. width .-> LFO21
LFO7 -. width .-> LFO29
LFO8 -. width .-> LFO9
LFO9 -. width .-> LFO23
LFO10 -. width .-> LFO20
LFO11 -. width .-> LFO29
LFO12 -. width .-> LFO11
LFO13 -. width .-> LFO14
LFO14 -. width .-> LFO13
LFO15 -. width .-> LFO1
LFO16 -. width .-> LFO10
LFO17 -. width .-> LFO24
LFO18 -. width .-> LFO26
LFO19 -. width .-> LFO22
LFO20 -. width .-> LFO5
LFO21 -. width .-> LFO5
LFO22 -. width .-> LFO23
LFO23 -. width .-> LFO10
LFO24 -. width .-> LFO9
LFO25 -. width .-> LFO13
LFO26 -. width .-> LFO15
LFO27 -. width .-> LFO2
LFO28 -. width .-> LFO2
LFO29 -. width .-> LFO23
LFO30 -. width .-> LFO5
LFO31 -. width .-> LFO23
{{</mermaid>}}

###  The tape machine: VortexVoice
The Vortex system consists of several interconnected voices. 
These voices are controlled using the multiplexer described above.

Each voice takes a sound input, passes it through an sound effect chain and through a varispeed loop before reaching the output.

On it's route to the output, the sound is mixed with the input of other voices (and is itself sent to other voices to be mixed with them). 

Features:
* Simple control input => complex effect on sound process
* Output agnostic: Number of output channels is defined at init
* Livecodable: The guts of the voice are organized in a way to be dynamically changed and reorganized after initialization.

#### FX chain organization
Sound effects in a VortexVoice may be changed dynamically.
This and the specific sound effect algorithms are organized externally in the [Sleet package](github.com/madskjeldgaard/sleet).

##### Example of a VortexVoice

{{<mermaid>}}
graph TB

C[control input]
M[multiplexer]
C --> M

M -.-> FX1
M -.-> FX2
M -.-> FX3
M -.-> FX4
M -.-> FX5
M -.-> VAR
M -.-> MIX

SI(SoundIn)
SI --> MIX


FX1[Delay]
FX2[Reverb]
FX3[PitchShift]
FX4[FreqShift]
FX5[Ring Modulation]

subgraph FXCHAIN
FX1 --> FX2 --> FX3 --> FX4 --> FX5
end

MIX((Mixer))
OV1((Other voice))
OV2((Other voice))

subgraph MIXING
OV1 --> MIX
OV2 --> MIX
MIX --> FX1
end

VAR[Varispeed looper]
SO(SoundOut)

FX5 --> VAR --> SO
VAR --> MIX

{{</mermaid>}}

