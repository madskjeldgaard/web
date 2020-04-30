---
title: 'SuperCollider tutorial: Easily render generative compositions as sound files using NRT'
author: mads
type: post
date: 2019-08-05T17:46:09+00:00
url: /supercollider-how-to-render-patterns-as-sound-files-using-nrt/
featured_image: /wp-content/uploads/2019/08/supercollider-laptop-room-e1566211773173.jpg
draft: false
tags:
  - tutorial
  - supercollider
---

![alt](/wp-content/uploads/2019/08/supercollider-laptop-room-e1566211773173.jpg)

One of the many powerful features of SuperCollider is it&#8217;s ability to render sounds offline. This is called [Non-Realtime Synthesis (NRT)][1]. NRT is for example useful for fast, offline processing of sounds, doing sound analysis or rendering generative compositions.

NRT works like this (normally): First you write a list of server OSC messages (stored in a [Score][2] usually) which will tell the (offline) server what to do at what point in time when you decide to render it. These are in the format [beat, [osc\_command]]. An example: Making a Synth using synthdef \boring\_sine at beat number 2 looks like this in such a OSC form: [2.0, [\s\_new, \boring\_sine, 1001, 0, 0]]

All of the possible server osc message commands are documented here: [Server Command Reference][3]

Creating such a list manually is naturally time consuming and does not feel very smart. It is also a very awkward way of writing music.

Fortunately, there are ways of doing this in a more efficient and musical way.

# Recording event patterns

One of my favourite techniques is to simply convert Event Patterns (such as Pbind, Pmono, etc.) to scores using the .asScore method.

The process can be divided into the following steps:
  
1. Make a SynthDef and store it on your system using .store
  
2. Write an event pattern
  
3. Convert the event pattern to a Score object using .asScore
  
4. Render the Score to a sound file on your system

First step is to make a SynthDef. SynthDefs are sort of recipes for sound patches that the server uses to make sound. In this case, it will be a very boring sine, aptly named \boring_sine. Note the use of the .store method here. This will save the synthdef as a file on your system and make it available to the NRT process later on.

<pre><code class="language-supercollider">SynthDef.new(\boring_sine, {|freq, dur|
// A percussive envelope
var env = EnvGen.ar(Env.perc, gate: 1, timeScale:dur);

// A sine wave
var sig = SinOsc.ar(freq);

// Apply envelope to sine wave and output
Out.ar(0, env * sig)
}).store;
</code></pre>

We will keep the pattern super simple: Random scale degrees played using our \boring_sine synth, each of which a quarter of beat in duration.

The total duration of the pattern will be infinite for now (the length of this will automatically be truncated by the Score conversion process).

<pre><code class="language-supercollider">p = Pbind(
\instrument, \boring_sine,
\dur, 0.25,
\degree, Pwhite(0,10)
);
</code></pre>

And now, let us convert this to a score:

<pre><code class="language-supercollider">p = p.asScore(60); // Duration of 60 beats
</code></pre>

Finally, we render the score

<pre><code class="language-supercollider">(
// Destination path and file name
~outFile = "~/Desktop/yo.wav";

// Render the score as wav file
p.recordNRT(outputFilePath: ~outFile.asAbsolutePath, headerFormat: "WAV");
)
</code></pre>

# Lets make this interesting: Iteration

[<img class="alignnone size-large wp-image-527" src="https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2-1024x315.png" alt="" width="640" height="197" srcset="https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2-1024x315.png 1024w, https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2-300x92.png 300w, https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2-768x236.png 768w, https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2.png 1909w" sizes="(max-width: 640px) 100vw, 640px" />][4]

Rendering one random melody is quite nice, but let us exploit the fact that our pattern chooses random scale degrees every time we play it and combine that functionality with iteration to make 10 (or any number) of rendered random melodies.

First, let us wrap what we wrote up until this point in a function that we can call as often as we want.

<pre><code class="language-supercollider">~renderMelody = { |pattern, destinationPath|

// Convert pattern to score
var patscore = pattern.asScore(60); // Duration of 60 beats

// Render the score as a wav file
patscore.recordNRT(outputFilePath: destinationPath.asAbsolutePath, headerFormat: "WAV");
};
</code></pre>

Let us keep the pattern as is

<pre><code class="language-supercollider">p = Pbind(
\instrument, \boring_sine,
\dur, 0.25,
\degree, Pwhite(0,10)
);
</code></pre>

And then render 10 versions of it

<pre><code class="language-supercollider">// Now make 10 different melodies and save them to your desktop as wave files
10.do{|index|
~renderMelody.value(p, "~/Desktop/boringMelody" ++ index ++ ".wav");
};
</code></pre>

 [1]: http://doc.sccode.org/Guides/Non-Realtime-Synthesis.html
 [2]: http://doc.sccode.org/Classes/Score.html
 [3]: http://doc.sccode.org/Reference/Server-Command-Reference.html
 [4]: https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/sc-render-chopped2.png
