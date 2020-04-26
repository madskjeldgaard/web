---
title: "SoX tutorial: Command line tape music (an introduction)"
date: 2020-04-26T00:00:44+02:00
draft: false
toc: true
images:
tags:
  - sox
---

SoX is a very powerful command line audio processing tool. You can think of it as a sort of command line equivalent of Audacity but with a text based interface that let's you perform powerful audio operations by typing just a few words in your computer's terminal.

I came across SoX via the live coding community where it is a popular tool for chopping sound files (by detecting silence) and batch processing large quantities of audio files (eg. normalizing and entire folder of drum sounds in a matter of seconds). It is a very trustworhy and flexible solution ̣̣- and can even be used for DJ'ing.

But the commands for SoX are dense and sometimes slightly obscure so getting started is not always easy.

Hopefully, this tutorial will sort out those problems.

## Installing SoX

### MacOS
Using the Homebrew package manager:
```bash
brew install sox
```
### Windows

Using the Chocolatey package manager

```bash
choco install sox.portable
```

### Ubuntu

```bash
sudo apt install sox
```

### Arch / Manjaro

```bash
sudo pacman -S sox
```

## What's included

### The manual
![sox manual](/img/small/sox-man.png)

To read the manual, open up a terminal and type `man sox`.

The manual is the best place to find information and example usages for SoX.

To search the manual press `/` followed by your search query. By pressing `n` while searching, the cursor will jump to the next instance of the query you searched for. By pressing `N` the cursor will jump to the previous instance.

Scroll forward one page by pressing `ctrl-f`

And backwards one page by pressing `ctrl-b`

Exit the manual by pressing `q`

### Audio recorder

SoX includes a very handy way of recording audio using the `rec` command.

The simplest use is to type `rec filename` which will start recording from the default input until you stop it by pressing `ctrl-c` in the terminal window.

Example use:

```bash
rec hello.wav
```

You can specify a predefined length of the recording like this: `rec hello.wav trim 0 30:00` which will record for 30 minutes and then automatically stop

### Audio player

Similarly, playing audio is also possible using the command `play`

Example use:
```bash
play hello.wav
```

Note that both play and rec can be used with SoX's many included effects.

Playing the above example at half speed with a flanger at the end is as simple as

```bash
play hello.wav speed 0.5 flanger
```
## Command line basics

You do not have to have a lot of command line experience to use SoX but there is a few basic commands that will make it easier for you to navigate your computer in the command line.

• `pwd` - see path to directory you are currently in
• `ls` - see files in current directory
• `cd /some/path` - move to __/some/path__
• `cd ..` - move up one folder
• `cd ~` - move to home folder

As well as commands, here are some essential keyboard shortcuts:
• up/down - scroll through previous commands (easy way to see / reuse previous work)
• ctrl-c - cancel/abort the program (a sort of panic button)

Sometimes, a simple way of using `cd` is to drag and drop a folder from your computer onto your terminal, this will in most cases paste the full path.

## The sox command

Our main interface for sox in this tutorial will be the `sox` command. The basic usage of this includes specifying an input file path, an output file path and then optionally some effects followed by an optional series of parameters.

The basic command we will use will thus look like this:

```bash
sox inputfile outputfile effect parameters
```

If your input or output file contains spaces in the file name, you should wrap the path to it in quotation marks like this:

```bash
sox "/some folder/containg/a sound.wav" "outputfolder/a new sound.wav" effect param1
```

Like other command line tools, sox executes from the context of the folder that you are currently in (which can be found by typing `pwd`) and as such you do not need to type out the entire path to a file if it is in the same folder as the one you are executing sox from.

## Command line tape music

Tape music artists of the 60's and 70's had very rudimentary tools at their disposal - mostly they did their work using reel-to-reel tape recorders and simple effects. But you would be surprised by the incredible sonic possibilities available in a tool as simple as this, using basic techniques of recording, reversing, adding effects, changing playback speed, etc. you can get a long way towards making interesting music.

Let us explore some of SoX' basic commands a bit by doing some command line audio manipulations reminiscent of classic tape music techniques.

First of all, we need some audio to operate on. I would recommend recording a quick bit from your computer's microphone. If you have an instrument around, maybe use that for this exercise or simply (like me) whistle like an idiot in front of your computer.

Record to the file idiot.wav for 10 seconds:

```bash
rec idiot.wav trim 0:0 0:10
```

Once SoX is done recording, it will post a "done" message.

Moving on, let us test the file we just recorded

```bash
play idiot.wav
```

You should now hear yourself doing something silly in front of your computer a few seconds ago.

To convert this to something else, we need to invoke the `sox` command now, providing it with an input file name, an output file name and a chain of effects. In this example, I will add a silly effects chain consisting of reversing the audio -> flanger (2ms delay) -> playback speed 50% (0.5) -> reverb. The output of this operation will be saved in the file `art.wav`

```bash
sox idiot.wav art.wav reverse flanger 2 speed 0.5 reverb
```

If you execute the command `ls` now, you should in your directory see both the files idiot.wav (the original) and the manipulated file art.wav.

Just to be sure, we can test our output file.

```bash
play art.wav
```

Now, we would not be proper command line tape musicians if we felt satisfied after 1 manipulation to the original recording, so let us continue our sonic journey by transforming the `art.wav` file further, this time we will time stretch to twice the length (factor 2), reverse the audio again and add some reverb. Just to make sure we do not lose too much of our audio level, we will normalize the output to -0.1db as well finally saving the result in the file `art2.wav`

```bash
sox art.wav art2.wav stretch 2 reverse reverb norm -0.1
```
