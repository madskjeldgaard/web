---
title: 'SuperCollider tutorial: Mass producing SynthDefs'
author: mads
date: 2019-08-19T10:36:35+00:00
tags:
  - tutorial
  - supercollider
images:
- /img/small/supercollider-laptop-room.jpg
- 

---
In SuperCollider, one of the most common ways of making sounds is by first defining a sort of recipe for a UGEN patch in a SynthDef and then from that recipe produce Synths that make sounds.

But when you write a SynthDef, the patch architecture cannot change after the definition (as opposed to changing arguments in the patch).

This becomes annoying when working with UGens that want to know the exact number of channels used, eg. [PlayBuf][1], when defining the Synth.

These kinds of UGEN arguments cannot be changed from the outside like other UGEN arguments, and so if you want to make a synth based on the PlayBuf buffer player UGEN, you have to make seperate versions for mono and stereo buffers because the **numChannels** argument is fixed on definition.

In the following, you will see how to mass producing SynthDefs in two different ways: One for simple multichannel enumeration (which can be seen used in the wild in projects such as [SuperDirt][2]]) and another for more complex variations in patches.

These are techniques that I use extensively myself to help me organize my synth library (see [KModules][3]) and they can help you unclutter and shrink your own library as well.

# First technique: Multichannel enumeration

When you need to create synths and you want it to be flexible in terms of the number of channels involved, a nice way of doing it involves the almighty _do_ function.

The trick here is basically to put the SynthDef inside of a do function, which will repeat 64 times. We will then use the index from the do process to append to the SynthDef’s name and set the ugens to the appropriate amount of channels.

Let’s start by making the function to be used inside of the SynthDef:

```javascript 
// Function for buffer player synth defs
~bufplayerfunc = {|numchans=1|
   {|rate=1, buffer, trigger=1, start=0, loop=0, amp=1, out=0|

      // Buffer player
      var sig = PlayBuf.ar(
         numchans, // Number of channels passed into the function from the outer function
         buffer, 
         rate * BufRateScale.kr(buffer),  
         trigger,  
         start * BufDur.kr(buffer),  
         loop
         );

      // Output
      Out.ar(out, sig * amp);
   }
};
```

As you can see, the synth function is wrapped in an outer function which takes one argument: the number of channels. The function returns the actual synth function we need to put inside of a SynthDef.

Using the numchans argument, the PlayBuf UGEN is set to appropriate channel number (and the Out UGEN is smart enough to adjust to this).

The next step is to put this function inside of a SynthDef and call it 64 times. One time for each number of channels we want to have:

```javascript
(1..64).do{|chanNum|
   var name = "bufplayer" ++ chanNum;
   SynthDef.new(name, ~bufplayerfunc.value(chanNum)).add;
};
```

Now, whenever you need to use this synthdef, you can call it by it’s basename (“bufplayer” in this case) plus the number of channels. For example: A 33 channel buffer player would then look like `Synth(\bufplayer33, [\buffer, b])`.

# Second technique: SynthDef.wrap

In the beginning of this blog post I mentioned that the architecture of a SynthDef needs to be fixed upon definition. But there is a way around this which involves an amazing method in the SynthDef class called [wrap][4].

It may seem a bit hard to understand how it works at first, but once you have gotten the hold of it, wrap has mind blowing potential for quasi-dynamically making SynthDefs. In other words: semi-automatic sound patching.

Let us say we want to build a Synth which is a basic sawtooth based oscillator with a filter at the end. Now, SuperCollider contains a lot of different filters. Let us make a few different versions of this synth, all containing different filters.

We will organize the filter functions in an Event (which is a sort of Dictionary). When putting them in a data structure like this, we can easily get all of them using iteration.

Then, we will use a sort of do-function called `keysValuesDo` to get the filter names and functions and then for each of them create a SynthDef containing that particular filter.

## The wrap function and it&#8217;s arguments

When you add a function to your synthdef using .wrap like this, the outer SynthDef gets the arguments you defined in the filter functions. You do not have to define them with the freq argument of the synthdef itself because **they will automatically be added to your synthdef**. So our SynthDef will get a `cutoff` argument when we use SynthDef.wrap inside of it with our functions.

The signal of our Saw oscillator is passed into the filter using the wrap-argument _prependArgs_.

This part of SynthDef.wrap is very important and a bit confusing too. Anything you pass in to the prependArgs argument will be put into the first argument(s) of the function used in the wrap-method.

The argument in question (in our case the _in_ argument) will then be removed from the outer function&#8217;s list of arguments. The prepended argument is in other words overwritten and becomes unavailable to the outside SynthDef argument list.

```javascript
// Filter functions organized in a dictionary (Event)
// The signal of our synth will be passed in as the first argument
f = (
    hpf: { |in, cutoff=1000, rq=1|
        RHPF.ar(in, cutoff, rq)
    },
    bpf: { |in, cutoff=1000, rq=1|
        BPF.ar(in, cutoff, rq)
    },
    lpf: { |in, cutoff=1000, rq=1|
        RLPF.ar(in, cutoff, rq)
    }
);

// Iterate over all the filters we defined above and use them in a SynthDef
f.keysValuesDo{|filtername, filterfunction| 
    var synthdefname = "saw" ++ filtername.asString;

    SynthDef.new(synthdefname, { |freq=220, out=0|
        var sig = Saw.ar(freq, mul:0.1);

        sig = SynthDef.wrap(
            filterfunction,  
            prependArgs: [sig] // Pass signal in to the filter
            // NOTE: prependArgs HAVE to be inside of []
        ); 

        Out.ar(out, sig)
    }).add;
};

)
```
Now, let us test these synths:

```javascript
// Test low pass version
Synth("sawlpf", [\freq, 222, \cutoff, 100]);

// Test high pass version
Synth("sawhpf", [\freq, 831, \cutoff, 1000]);

// Test band pass version
Synth("sawbpf", [\freq, 323, \cutoff, 1000]);
```

Once you have gotten into the habit of using SynthDef.wrap it really is a flexible and powerful way of making Synths which takes care of a lot of the plumbing you otherwise need to do whenever you write a SynthDef, and it allows you to really experiment with different patching ideas.

Note that, in the example above, whenever you add a filter function to the dictionary at the top, it will automatically be added as another SynthDef.

Another cool thing about SynthDef.wrap is that you can actually use it inside of NodeProxies and Ndefs as well when livecoding.

 [1]: http://doc.sccode.org/Classes/PlayBuf.html
 [2]: https://github.com/musikinformatik/SuperDirt
 [3]: https://github.com/madskjeldgaard/kmodules
 [4]: http://doc.sccode.org/Overviews/Methods.html#wrap
