---
title: "SoX tutorial: Batch processing audio on the command line"
date: 2020-04-26T10:07:25+02:00
draft: false
toc: true
images:
tags:
 - tutorial
 - sox
---

To make full use of SoX' potential for batch processing we will be using a bit of command line wizardry.

The idea is to put our sox command inside of a for-loop which iterates over all audio files in the folder you are currently in. If you are unsure of what folder your terminal is executing from, you can write `pwd` to see it's full path and `ls` to see the folder's contents.

# For-loop

The structure of our for-loop-command will look something like this:
```bash
for file in *.wav; do command $file; done
```
This needs a bit of explanation. What we see here is shell scripting where commands (or lines of code you could say) are seperated by semi colons.

The first bit of this command (`for file in *.wav`) will find all files in the current directory containing the suffix `.wav`. Note that this is case-sensitive, so if you want WAV files to be converted change it to `*.WAV`, and so on. The smart thing about this is that each file inside of the for-loop will accessible as the variable `$file`.

The second bit of the command is the meat of it. Here we execute our sox command like we have done previously in this tutorial, with the main difference being we put `do` in front of it - this is a way to tell our for loop that this is supposed to happen on each file we find.

The third bit is self-explanatory: `done`.

We will be operating on .wav-files here, but you can easily change the commands to target .aiff files or some other supported file format.

# Batch processing examples

## Normalize
To normalize a file in SoX we need to apply the `norm` effect which only takes one parameter which is the sound level to normalize to. A reasonable normalization level is -0.1 dB so let us use that in our conversion process.

```bash
for file in *.wav; do sox "$file" "n_$file" norm -0.1; done
```

Notice that what we do here is non-destructive. The normalized files produced by this process have the same file names as the input files but with a "n_" at the beginning to signify that it has been normalized.

## Channel conversion

Another useful effect included with SoX is `channels`, this makes it possible for us to specify the number of channels in the output file. This is useful if for example you need to convert a folder of files to mono, this can be done in a manner similar to the above:

```bash
for file in *.wav; do sox "$file" "mono_$file" channels 1; done
```

## Convert to 48khz sample rate

```bash
for file in *.wav; do sox "$file" "48khz_$file" rate 48k; done
```

## Convert to 16bit

Now, this is slightly different because when converting bit-depth we need to use a command line flag instead of an effect. Normally, SoX will use the detected bit-depth of the input file as the bit-depth of the output file, but you can force SoX to change it to something else (like 16 bit) by adding a `-b 16` in between the file names (or `-b 24` for 24 bit).

```bash
for file in *.wav; do sox "$file" -b 16 "16bit_$file"; done
```

# Where to go from here?
I would highly encourage you to go back and read the SoX manual (`man sox`, remember?) because there really is a plethora of fun and useful things you can do with SoX in a for-loop.
