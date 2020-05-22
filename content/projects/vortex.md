---
title: "Vortex"
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

This resonated deeply with me along with a want for a system that would be able to programmatically generate such small and large scale structures in composition and performance from simple gestures in sound and control. Or even better: To have a system which seems to have a will of it's own, a vibrating structure which may be affected or disturbed by the performer (me) but never fully controlled, leading to a range of surprises. In my piece [States of Emergency](https://shop.conditional.club/album/states-of-emergency) I achieved this somewhat by manually performing recursive sound effect transformations controlled by complex low frequency oscillators - iterating over the results for days on end in a DAW, this process became laborous and I started dreaming of automating this process to be able to setup and define such systems more easily. 

A few years later I participated in a workshop with [Jérôme Noetinger who works with reel to reel tape machines as instruments](https://www.youtube.com/watch?v=pnZ55jQe8jA) for improvisation by setting up tape loops, (re)injecting sounds into them and varying their playback speed. Working with the materiality of the sound in this way really fascinated me and so this core concept of a digital varispeed tape loop is at the center of the Vortex system. Other inspirations for this part of the project are [Eliane Radigue's feedback works](https://youtu.be/C_3Fu8YfSdI), [Giovanni Lami's outdoors tape loop sessions](https://vimeo.com/238351530) as well as [Valerio Tricoli's tape loop live performances](https://www.youtube.com/watch?v=J7nDhTk705s). 


## Design goals

- **You will never master this instrument**: Each time the system is used, it is reconfigured in a new and slightly random way making it necessary to relearn it and find new sweet spots.
- **Gestures have unforeseable consequences**: Whatever gesture you put into the system (through code or controllers) should have consequences that are neither random nor predictable, like a [Lorenz system](https://en.wikipedia.org/wiki/Lorenz_system). 
- **Multiple time scales**: The same input gestures cause reactions in sound and control now, in the near future and on a massive time scale.
- **No sound source**: The system only processes sounds, forcing the user to input sound.
- **No control, only influence**: The system cannot be fully controlled, only influenced.
- **Channel agnostic**: No usage differences between stereo and multichannel outputs.
- **Interface agnostic**: May be used using code, patterns and or any physical controller. The system/instrument as an API.
- **Extremely dynamic**: The whole system may be reconfigured and repatched by executing a command or pushing a button. 

## Technical details
![controllers](https://raw.githubusercontent.com/madskjeldgaard/vortex/master/documentation/vortex_controllers.JPG)

Vortex itself is an interface for controlling several VortexVoices. It allows to set these voices up in feedback loops, interconnect them and maintain a common control input for them all. 

In other words: It is designed as an API.

### Controlling vortex
Vortex may be controlled by live coding with it or using physical controllers like MIDI controllers, OSC devices and HID devices. 

{{<mermaid>}}
graph TB
G{sound in} --> E

subgraph GESTURES
A[performer] --> B[keyboard/code]

A --> C[controllers]
end
B --> D(multiplexer)
C --> D

D --> E(vortex)
D --> E
D --> E
D --> E
D --> E
D --> E

style E fill:#BD1E2E
style G fill:#8BCEF0
style F fill:#8BCEF0

E --> F{sound out}
{{</mermaid>}}
### Multiplexing control data: VortexFlux
Input data is multiplexed by an algorithm called VortexFlux (based on Alberto de Campo's Influx package) which makes one or a few inputs become many outputs (the exact number may be decided upon initialization). 

It then passes those many outputs through further data warping algorithms to increase the complexity of the control data and further distance them from the gesture put in to the system. 

The steps are :
1. Weighting: Multiply the input with a random weight
2. Envelopes: Use the input of the previous as an index into a random envelope. This works sort of like a control rate wavetable.
3. LFO: The input of the previous is used to control a network of lfos. This step is optional. Additionally, the lfo network may be configured in a feedback mode with the individual lfo's in the network controlling eachother's frequency alongside the control input.
4. Map the mulitplexed control data to the parameters of a VortexVoice.


{{<mermaid>}}
graph TB

F((control)) --> A
A((multiplexer)) --> B((in * w))
A --> C((in * w))
A --> D((in * w))
A --> E((in * w))

B --> G((env))
C --> H((env))
D --> I((env))
E --> J((env))


G -.-> K((lfo))
H -.-> L((lfo))
I -.-> M((lfo))
J -.-> N((lfo))

K -.-> O{voice}
L -.-> O{voice}
M -.-> O{voice}
N -.-> O{voice}
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

W[control input]
W --> I

I[multiplexer]
I -.-> B
I -.-> C
I -.-> D
I -.-> E
I -.-> F
I -.-> G


A(SoundIn)
A --> B

subgraph FXCHAIN
B((Delay))
C((Pitchshift))
D((Reverb))
E((FreqShift))

B --> C
C --> D
D --> E
end

subgraph MIXING
H((Other voice)) --> F
Z((Other voice)) --> F
E --> F(Mixer)
end

G --> H
F --> G(Varispeed Loop)
G --> Z
G --> SoundOut

{{</mermaid>}}


