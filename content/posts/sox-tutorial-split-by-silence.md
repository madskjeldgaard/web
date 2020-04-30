---
title: "SoX tutorial: Split by silence"
date: 2020-04-26T01:07:33+02:00
draft: false
toc: true
images:
tags:
  - sox
  - tutorial
---

SoX has a very effective and rather precise way of semi-automatically chopping a sound file into smaller sound files.

Let us say you have a sound file containing many different sounds seperated by a bit of silence in between. It could be a series of drum hits that you have recorded off of a drum machine. To make these sounds easy to use, you most probably need them as seperate sound files so you can load them into a sampler or other software as a sample bank of sorts.

In SoX we can approach this problem quite simply: Split the input file (the long file containing many different sounds in sequence) by detecting the silence in between the sounds.

To do this we need to use the `silence` effect in SoX, which I will explain in a bit more detail since it is an important one and it's syntax is a bit esoteric to say the least.

## What is silence

In the manual, `silence` is defined like this:

"Removes silence from the beginning, middle, or end of the audio. 'Silence' is determined by a specified threshold."

`silence` takes a range of optional arguments but we will only use the first three of them:

• above-periods - indicate if audio should be trimmed at the begnning of the audio. 0 = no silence trimmed from beginning, 1 = trim silence from beginning
• duration - amount of time in seconds that non-silence must be detected before it stops trimming audio
• threshold - audio threshold, we will indicate this in percentages

The parameters are stringed together after the `silence` keyword in the sox command like this:
```bash
sox infile.wav outfile.wav silence above-periods duration threshold
```

## Trimming silence from beginning and end of one file
To trim the beginning of a file until the audio is above 1% in volume for more than 0.1 seconds, you would write a command like this:
```bash
sox infile.wav outfile.wav silence 1 0.1 1%
```

To trim the ending as well, we basically repeat the parameters like this:
```bash
sox infile.wav outfile.wav silence 1 0.1 1% 1 0.1 1%
```

## Chaining (pseudo) effects

This is all good and well, but we want to produce a sample bank from one input audio file. To do this we need to make use of SoX' ability to chain effects chains after eachother and enter into "multiple output file mode".

From the manual: "In multiple output mode, a new file is created when the effects prior to the 'newfile' indicate they are done. The effects chain listed after 'newfile' is then started up and its output is saved to the new file."

An effects chain can thus be chained after another using a colon `:`. Now instead of manually writing out the silence effect and it's parameters for each bit we want to extract from the sound file, we can make the process automatically restart each time it has detected a bit of sound by silence

To do this we need to chain the `restart` pseudoeffect at the end of our command. This will make the process create a new file from the bit it detected by silence, then restart the process from where it left off and repeat until it reaches the end of the file. Kind of like slicing off bits of a (sound) sausage from left to right.

Our final command for chopping files by silence will then end up looking like this:
```bash
sox infile.wav outfile.wav silence 1 0.1 1% 1 0.1 1% : newfile : restart
```
## Chopping three bursts

![alt](/img/small/threebursts.png)

As an example of the above, let us have a look at a sound file containing three short noise bursts.

The sound file is called threebursts.wav and can be [downloaded here](/audio/threebursts.wav).

To split the soundfile into three seperate files containing the bursts (without the silence in between), we simply execute the command

```bash
sox threebursts.wav burst_num.wav silence 1 0.1 1% 1 0.1 1% : newfile : restart
```

which will produce sound files called "burst_num001.wav", "burst_num002.wav" etc.

Now this works very well for our very unnatural example here, but I encourage you to mess around with the parameters when you do this on your own with your own files. Change the threshold to 5% for example if it's noisy or set the duration to something higher if it results in too many small files.

Note: Sometimes on some systems this command will produce an extra audio file containing nothing. I honestly have no idea why. Just delete the file (or send me an email if you have a solution to this problem)
