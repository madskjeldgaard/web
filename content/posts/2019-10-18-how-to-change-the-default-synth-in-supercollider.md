---
title: How to change the default synth in SuperCollider
author: mads
type: post
date: 2019-10-18T12:42:12+00:00
url: /how-to-change-the-default-synth-in-supercollider/
enclosure:
  - |
    |
        https://www.madskjeldgaard.dk/wp-content/uploads/2019/10/new-default-synth.mp4
        19174104
        video/mp4
        

---
The default synth sound in SuperCollider is a cheesy old piano sound. If you have ever tried the event pattern examples in the documentation of SuperCollider or been in the process of testing some pattern specifics of your own, you will have heard this extremely unconvincing synthesizer:

<pre><code class="language-supercollider">&lt;br />().play

</code></pre>

## A nice alternative: A triangle wave synth with a low pass filter

Imagine a utopian world where the default cheese-piano-synth has been replaced by a nicer, kind of gameboy like synth. Well that world is here and now.

Overwriting the default is actually easy. All you have to do is write a new SynthDef called `\default`. Evaluate this piece of code:
```javascript
// A simple triangle wave synth in stereo 
(
SynthDef.new(\default, {
arg dur, attack=0.01, release=1.0,
t_gate=1, out, freq=442, cutoff=5500,
rq=1, pan=0.0, amp=0.5;

var env = EnvGen.kr(
	Env.perc(attack, release), 
	t_gate, 
	timeScale: dur, 
	doneAction: 2
);
var sig = DPW3Tri.ar(freq: freq, mul: env);
sig = RLPF.ar(sig, cutoff.clip(20.0, 20000.0), rq.clip(0.0,1.0));
sig = Pan2.ar(sig, pan);
Out.ar(out, sig * amp);
}).add;
)
```


Try this new default synth out by playing the default event again:

```smalltalk
().play;
```

Isn&#8217;t that much better?

Now, let us try it with patterns:

<pre><code class="language-supercollider">Pbind(\dur, 0.125, \degree, Pwhite(0,10)).play
</code></pre>

## Make this the default synth permanently

To make this change permanent we need to edit the startup file for SuperCollider. This is a regular SuperCollider file that is evaluated on startup. It is a good place to keep settings and defaults. Here is how to access it:

  1. Open the SuperCollider IDE</p> 
  2. In the top menu, click &#8220;File&#8221;

  3. In the drop down click &#8220;Open startup file&#8221;

Now, if we just paste the code from above here, it will still get overwritten by the default synth when booting the server. To get around this, we need to add our synth after the server has booted. We do this by wrapping our SynthDef in a server function called doWhenBooted (`s.doWhenBooted{/* paste code here*/}`):

<pre><code class="language-supercollider">// A simple triangle wave synth in stereo with panning and a simple low pass filter
s.doWhenBooted{
SynthDef.new(\default, {
arg dur, attack=0.01, release=1.0,
t_gate=1, out, freq=442, cutoff=5500,
rq=1, pan=0.0, amp=0.5;

var env = EnvGen.kr(Env.perc(attack, release), t_gate, timeScale: dur, doneAction: 2);
var sig = DPW3Tri.ar(freq: freq, mul: env);
sig = RLPF.ar(sig, cutoff.clip(20.0, 20000.0), rq.clip(0.0,1.0));
sig = Pan2.ar(sig, pan);
Out.ar(out, sig * amp);
}).add;
};
</code></pre>

Here is what the default synth should now sound like (with a bit of reverb of course)

<div style="width: 640px;" class="wp-video">
  <!--[if lt IE 9]><![endif]--><video class="wp-video-shortcode" id="video-742-1" width="640" height="360" preload="metadata" controls="controls"><source type="video/mp4" src="https://www.madskjeldgaard.dk/wp-content/uploads/2019/10/new-default-synth.mp4?_=1" />
  
  <a href="https://www.madskjeldgaard.dk/wp-content/uploads/2019/10/new-default-synth.mp4">https://www.madskjeldgaard.dk/wp-content/uploads/2019/10/new-default-synth.mp4</a></video>
</div>
