---
title: Pattern workshop, Herlev Bibliotek 2019
author: mads
type: post
date: 2019-11-06T11:22:54+00:00
url: /pattern-workshop-herlev-bibliotek-2019/
featured_image: /wp-content/uploads/2019/11/IMG_20191107_190242_369.jpg
toc: true
tags:
  - workshop-material
  - supercollider
---

![alt](/img/small/workshop-herlev-2019.jpg)

Here are the materials for the SuperCollider workshop at Herlev Bibliotek, Denmark, November 2019.

The SynthDef used for the workshop [can be downloaded here.][1]

[Download slides here.](/pdf/herlev-2019.pdf)

---

SuperCollider workshop
----

### Herlev Bibliotek, Denmark, 2019

------------------------------------------------------------------------

About me
----

-   Name: Mads Kjeldgaard
-   Occupation: Composer and developer
-   Work: The Norwegian Center for Technology and Art (Notam)

------------------------------------------------------------------------

![studio3](studio3.jpg)

------------------------------------------------------------------------

Notam
----

-   Development for art projects (hardware, software, tech and artistic
    guidance)
-   Communities / meetups (SC meetup among others)
-   Studios / 3D sound / VR / Visuals
-   Courses

------------------------------------------------------------------------

My practice
----

-   Computer music / livecoding
-   Field recording
-   3D sound composition
-   Sound environments

------------------------------------------------------------------------

Contact info
----

-   mail: <mail@madskjeldgaard.dk>
-   web: [madskjeldgaard.dk](http://madskjeldgaard.dk)
-   github:
    [github.com/madskjeldgaard](http://github.com/madskjeldgaard)

------------------------------------------------------------------------

Design
----

------------------------------------------------------------------------

Short history of SuperCollider
----

SC was designed by James McCartney as closed source proprietary software

Version 1 [came out in 1996 based on a Max
object](https://groups.google.com/forum/#!topic/comp.music.research/g2f9EcL1mUw)
called Pyrite. Cost 250\$+shipping and could only run on PowerMacs.

Became free open source software in 2002 and is now cross platform.

------------------------------------------------------------------------

Overview
----

When you download SuperCollider, you get an application that consists of
3 separate programs:

1.  The IDE, a smart text editor
2.  The SuperCollider language / client (**sclang**)
3.  The SuperCollider sound server (**scsynth**)

------------------------------------------------------------------------

Architecture
----

![alt](https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/client-server.png)

The client (language and interpreter) communicates with the server
(signal processing)

This happens over the network using Open Sound Control

------------------------------------------------------------------------

Multiple servers
----

![alt](https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/client-multiple-servers.png)

This modular / networked design means one client can control many
servers

------------------------------------------------------------------------

Consequences of this modular design
----

Each of SuperCollider's components are replacable
-------------------------------------------------

IDE \<---\> Atom, Vim, or Visual Studio

language \<---\> Python, CLisp, Javascript

server \<---\> Max/MSP, Ableton Live, Reaper

------------------------------------------------------------------------

Extending SuperCollider
----

The functionality of SuperCollider can be extended using external
packages

These are called Quarks and can be installed using SuperCollider itself

    // Install packages via GUI (does not contain all packages)
    Quarks.gui;

    // Install package outside of gui using URL
    Quarks.install("https://github.com/madskjeldgaard/KModules");

------------------------------------------------------------------------

SC Plugins
----

[SC3 Plugins](https://supercollider.github.io/sc3-plugins/) is a
collection of user contributed code, mostly for making sound

The plugins are quite essential (and of varying quality / maintenance)

------------------------------------------------------------------------

IDE
===

------------------------------------------------------------------------

![the
ide](https://www.madskjeldgaard.dk/wp-content/uploads/2019/09/ide.png)

------------------------------------------------------------------------

Important keyboard shortcuts
----

-   Open help file for thing under cursor: **Ctrl/cmd + d**
-   Evaluate code block: **Ctrl/cmd + enter**
-   Stop all running code: **Ctrl/cmd + .**
-   Start audio server: **Ctrl/cmd + b**
-   Recompile: **Ctrl/cmd + shift + l**
-   Clear post window: **Ctrl/cmd + shift + p**

------------------------------------------------------------------------

The IDE as a calculator
----

SuperCollider is an interpreted language

This means we can "live code" it without waiting for it to compile

A good example of this is using it as a calculator

------------------------------------------------------------------------

Autocompletion
----

Start typing and see a menu pop up with suggestions (and help files)

------------------------------------------------------------------------

The status line
----

Shows information about system usage

Right click to see server options + volume slider

------------------------------------------------------------------------

About livecoding
----

------------------------------------------------------------------------

What is it?
----

The act of using a piece of software, while you write/modify it

------------------------------------------------------------------------

A quote
----

"In live coding the performance is the process of soware development,
rather than its outcome." - Alex McLean, Artist-Programmers and
Programming Languages for the Arts, 2011

------------------------------------------------------------------------

Livecoding isn't special
----

It is used all the time now in web design, science and software
development to prototype / finetune ideas interactively.

------------------------------------------------------------------------

Livecoding SuperCollider
----

Since SuperCollider is an interactive programming language and not a
compiled one (as opposed to Csound for example), you are in effect
always livecoding in SuperCollider.

------------------------------------------------------------------------

Livecoding music: Kind of special after all
----

Musical livecoding is intrinsically connected to improvisation.

It can be compared to playing a jazz concert on a guitar...

------------------------------------------------------------------------

Except
----

... You are actually building the guitar itself, while playing it.

------------------------------------------------------------------------

Quick history of live coding music
----

-   Took off in the early 2000's
-   Around 2000: Slub started playing with screen projections
-   Around same time: TSpawn trick -\> Hot swapping code in SC

------------------------------------------------------------------------

General strategies
----

1.  Building / reworking an instrument (using NodeProxy/Ndef)
2.  Writing or modifying patterns (using ProxySpace/Pdef)
3.  A mix of the above

------------------------------------------------------------------------

What are the rules of live coding?
----

-   There are none!
-   Do whatever you like!
-   Crush all conventions!

------------------------------------------------------------------------

About patterns
----

------------------------------------------------------------------------

From the [Pattern help
file](http://doc.sccode.org/Classes/Pattern.html):

### "\[The Pattern\] classes form a rich and concise score language for music"

------------------------------------------------------------------------

In other words:
----

Patterns are used to sequence and compose music

------------------------------------------------------------------------

Abstracting the composition process
----

the conditions for a composition vs. a fixed composition

------------------------------------------------------------------------

It's just data
----

Easily transpose, stretch and warp the composition

------------------------------------------------------------------------

Duration is not an issue
----

Composing a 4 bar loop is not necessarily any more or less work than a 4
hour one

------------------------------------------------------------------------

Guides in the help system
----

Patterns are pretty well documented in the help system:

-   [A practical
    guide](http://doc.sccode.org/Browse.html#Streams-Patterns-Events%3EA-Practical-Guide)
-   [Understanding Streams, Events and
    Patterns](http://doc.sccode.org/Browse.html#Tutorials%3EStreams-Patterns-Events)

------------------------------------------------------------------------

Event patterns
----

------------------------------------------------------------------------

Like pressing the key of a piano
----

What data does that involve?

------------------------------------------------------------------------

-   Duration of key press
-   Pitch of the key
-   Sustain (are you holding the foot pedal?)
-   etc. etc.

------------------------------------------------------------------------

What an Event looks like
----

    // See the post window when evaluating these
    ().play; // Default event
    (freq:999).play; 
    (freq:123, sustain: 8).play;

------------------------------------------------------------------------

Changing the default synth
----

The default synth sucks

You can change it by defining a new synth called \default

More info on [my
website](https://www.madskjeldgaard.dk/how-to-change-the-default-synth-in-supercollider/)

------------------------------------------------------------------------

Introducing the allmighty Pbind
----

Arguably the most important pattern class in SuperCollider

------------------------------------------------------------------------

Pbind data
----

Pbind simply consists of a list of key/value pairs

------------------------------------------------------------------------

Keys correspond to Synth arguments
----

Most often, keys correspond to a Synth's arguments.

Example: If a SynthDef has the argument cutoff, we can access that
argument in a Pbind using \cutoff.

------------------------------------------------------------------------

Some keys are special
----

------------------------------------------------------------------------

dur
===

\dur is used in most SynthDef's to specify the duration of a note/event.

Make sure this key never gets the value 0.

------------------------------------------------------------------------

stretch
----

\stretch is used to stretch or shrink the timing of a Pbind

------------------------------------------------------------------------

When does a Pbind end?
----

If one of the keys of a Pbind are supplied with a fixed length value
pattern, the one running out of values first, will make the Pbind end.

------------------------------------------------------------------------

Livecoding: Pdef
----

Livecoding patterns is easy. All you have to do is wrap your event
pattern (Pbind) in a Pdef:

    Pdef('myCoolPattern', Pbind(...)).play;

------------------------------------------------------------------------

What this means
----

The Pdef has a name ('myCoolPattern') which is a kind of data slot
accessible throughout your system

Everytime you evaluate this code, it overwrites that data slot
(maintaining only one copy)

------------------------------------------------------------------------

Value patterns
----

------------------------------------------------------------------------

The building blocks of compositions
----

-   List patterns
-   Random patterns
-   Envelope patterns
-   Rests
-   Data sharing between event parameters
-   Patterns in patterns

------------------------------------------------------------------------

List patterns
----

See [all of them
here](http://doc.sccode.org/Browse.html#Streams-Patterns-Events%3EPatterns%3EList)

------------------------------------------------------------------------

Pseq: Classic sequencer
----

    // Play values 1 then 2 then 3
    Pseq([1,2,3]);

    // 4 to the floor
    Pseq([1,1,1,1]);

------------------------------------------------------------------------

Testing value patterns: asStream
----

You will see the .asStream method a lot in the documentation for value
patterns.

    // Pattern
    p = Pseq([1,2,3]);

    // Convert to stream
    p = p.asStream;

    // See what values the pattern produces
    p.next; // 1, 2, 3, nil

------------------------------------------------------------------------

Random value patterns: Pwhite and Pbrown
----

    // (Pseudo) random values
    Pwhite(lo: 0.0, hi: 1.0, length: inf);

    // Drunk walk
    Pbrown(lo: 0.0, hi: 1.0, step: 0.125, length: inf);

------------------------------------------------------------------------

Random sequence patterns: Prand and Pxrand
----

    // Randomly choose from a list
    Prand([1,2,3],inf);

    // Randomly choose from a list (no repeating elements)
    Pxrand([1,2,3],inf);

------------------------------------------------------------------------

Probability: Pwrand
----

Choose items in a list depending on probability

    // 50/50 chance of either 1 or 10
    Pwrand([1, 10], [0.5, 0.5])

    // 25% chance of 1, 25% change of 3, 50% chance of 7
    Pwrand([1, 3, 7], [0.25, 0.25, 0.5])

    // 30% chance of 3, 40% change of 2, 30% chance of 5
    Pwrand([4, 2, 5], [0.3, 0.4, 0.3])

------------------------------------------------------------------------

Envelope pattern: Pseg
----

    // Linear envelope from 1 to 5 in 4 beats
    Pseg( levels: [1, 5], durs: 4, curves: \linear);

    // Exponential envelope from 10 to 10000 in 8 beats 
    Pseg( levels: [10, 10000], durs: 8, curves: \exp);

------------------------------------------------------------------------

Rest
----

Skip/sleep a pattern using Rest. If used in the \dur key of a Pbind, the
value in the parenthesis is the sleep time

    // One beat, two beats, rest 1 beat, 3 beats
    Pbind(\dur, Pseq([1,2,Rest(1),3])).play;

------------------------------------------------------------------------

Pkey: Share data between event keys
----

Using Pkey we can make an event's parameters interact with eachother

    // The higher the scale degree
    // ... the shorter the sound
    Pbind(
        \degree, Pwhite(1,10),
        \dur, 1 / Pkey(\degree)
    ).play

More info about data sharing in patterns:
[here](http://doc.sccode.org/Tutorials/A-Practical-Guide/PG_06g_Data_Sharing.html)

------------------------------------------------------------------------

patterns in patterns: The computer music inception
----

You can put patterns in almost all parts of patterns.

This may lead to interesting results:

    // A sequence with 3 random values at the end
    Pseq([1,2,Pwhite(1,10,3)]);

    // An exponential envelope of random length
    Pseg(levels: [10, 10000], durs: Pwhite(1,10), curves: \exp);

------------------------------------------------------------------------

Working with pitches and Pbinds
----

------------------------------------------------------------------------

Pitch model
----

![pitch model](http://doc.sccode.org/Classes/Event-default-note.png)

[Pitch model is described
here](http://doc.sccode.org/Classes/Event.html)

------------------------------------------------------------------------

Changing scales
----

    // Use the \scale key, pass in a Scale object
    Pbind(\scale, Scale.minor, \degree, Pseq((1..10))).play;
    Pbind(\scale, Scale.major, \degree, Pseq((1..10))).play;
    Pbind(\scale, Scale.bhairav, \degree, Pseq((1..10))).play;

------------------------------------------------------------------------

Available scales
----

    // See all available scales
    Scale.directory.postln

------------------------------------------------------------------------

Changing root note
----

    // Use the \root key to transpose root note (halftones)
    Pbind(\root, 0, \degree, Pseq((1..10))).play;
    Pbind(\root, 1, \degree, Pseq((1..10))).play;
    Pbind(\root, 2, \degree, Pseq((1..10))).play;

------------------------------------------------------------------------

Changing octaves
----

    // Use the \octave key
    Pbind(\octave, Pseq([2,4,5],inf), \degree, Pseq((1..10))).play;
    Pbind(\octave, Pwhite(3,6), \degree, Pseq((1..10))).play;
    Pbind(\octave, 7, \degree, Pseq((1..10))).play;

------------------------------------------------------------------------

Playing chords
----

    // Add an array of numbers to the degree parameter 
    // to play several synths at the same time (as a chord)
    Pbind(\degree, [0,2,5] + Pseq([2,4,5],inf), \dur, 0.25).play;

------------------------------------------------------------------------

Changing tempo
----

The tempo of patterns are controlled by the TempoClock class You can
either create your own TempoClock or modify the default clock like below

    TempoClock.default.tempo_(0.5) // Half tempo
    TempoClock.default.tempo_(0.25) // quarter tempo
    TempoClock.default.tempo_(1) // normal tempo

------------------------------------------------------------------------

Learning resources
----

------------------------------------------------------------------------

Videos
----

Tutorials by Eli Fieldsteel covering a range of subjects: [SuperCollider
Tutorials](https://www.youtube.com/watch?v=yRzsOOiJ_p4&list=PLPYzvS8A_rTaNDweXe6PX4CXSGq4iEWYC)

------------------------------------------------------------------------

Books
----

E-books
-------

-   [A gentle introduction to
    SuperCollider](https://ccrma.stanford.edu/~ruviaro/texts/A_Gentle_Introduction_To_SuperCollider.pdf)
-   [Thor Magnussons Scoring Sound](https://leanpub.com/ScoringSound)

Paper books
-----------

-   [Introduction to SuperCollider, Andrea
    Valle](https://www.logos-verlag.de/cgi-bin/engbuchmid?isbn=4017&lng=eng&id=)
-   [The SuperCollider
    Book](https://mitpress.mit.edu/books/supercollider-book)

------------------------------------------------------------------------

Community
----

-   [scsynth.org](http://scsynth.org/)
-   [sccode.org](http://sccode.org/)
-   [Slack](https://scsynth.slack.com/)
-   [Lurk](https://talk.lurk.org/channel/supercollider)
-   [Mailing
    list](https://www.birmingham.ac.uk/facilities/ea-studios/research/supercollider/mailinglist.aspx)
-   [Telegram](https://t.me/supercollider_en)
-   [Telegram ES](https://t.me/supercollider_es)
-   [Facebook](https://www.facebook.com/groups/supercollider/)

------------------------------------------------------------------------

Awesome SuperCollider
----

A curated list of SuperCollider stuff

Find inspiration and (a lot more) more resources here:

[Awesome
Supercollider](https://github.com/madskjeldgaard/awesome-supercollider)

------------------------------------------------------------------------

Learning to code: Advice
----

-   Practice 5 minutes every day
-   Set yourself goals: Make (small) projects
-   Use the community

------------------------------------------------------------------------
