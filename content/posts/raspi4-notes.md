---
title: "Notes for setting up a Raspberry Pi 4 for audio work"
date: 2020-04-30T16:07:22+02:00
draft: true
toc: true
tags:
- supercollider
- raspberrypi
---

These are notes for setting up a Raspberry Pi 4 single board computer for pro audio work. Specifically for running [SuperCollider programs for interactive installations](supercollider.sourceforge.net/).

I used [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/) for this, since my intention is to run the Raspberry Pi in headless mode, meaning using no display and no desktop environment (to save resources). 

It is presumed that you have installed Raspbian on an sd card, with ssh enabled (allowing you to log in to the pi over the network and execute commands using a terminal), and that you are logged in. 

This allows us to setup up and run everything from the command line.

## Tuning the system

The following commands are based on [the canonical instructions for tuning your linux system for audio work.](https://wiki.linuxaudio.org/wiki/system_configuration). *Warning:* These will alter config files on your system, use at your own risk and only on a fresh raspbian installation. 

```bash
# Install utils for cpu freq
sudo apt-get install cpufrequtils

sudo cpufreq-set -r -g performance

sudo echo "ENABLE="true"
GOVERNOR="performance"
MAX_SPEED="0"
MIN_SPEED="0" " | sudo tee -a /etc/default/cpufrequtils

# Set realtime priority and memlock
sudo echo "
@audio nice -15 
@audio - rtprio 90       # maximum realtime priority
@audio - memlock unlimited  # maximum locked-in-memory address space (KB)
" | sudo tee -a /etc/security/limits.conf

# Set swappiness
# This setting changes the so-called swappiness of your system, 
# or in other words, the moment when your system starts to use its swap partition. 
sudo echo "
vm.swappiness = 10
fs.inotify.max_user_watches = 524288
" | sudo tee /etc/sysctl.conf

# Install other useful tools
sudo apt-get install htop git perl
```
## Install and setup jack

```bash
# Install jack
sudo apt-get install jackd2

# Create config file in home folder called .jackdrc
echo /usr/bin/jackd -P75 -dalsa -dhw:1 -r44100 -p512 -n3 > ~/.jackdrc


```

[Fredrik Olofsson](https://www.fredrikolofsson.com) pointed out to me that it can be useful to use the `-p`-flag to restrict the number of possible ports for jack to connect to, this reduces the amount of ram used, which is why the SuperCollider readme recommends adding the `-p16` flag to the jack command in `.jackdrc`.

### Changing audio interface

Never use the built in audio jack on a Rasbperry Pi for anything serious. Instead, you can use any class compliant usb audio interface. These range from cheap 1$ chinese ones to top-of-the-line RME Fireface UCX. I use and like both of these for different situations.

To see which audio interfaces are available on the pi:
`aplay -l`

With a USB-interface plugged in, the output should something like this:
```
**** List of PLAYBACK Hardware Devices ****
card 0: ALSA [bcm2835 ALSA], device 0: bcm2835 ALSA [bcm2835 ALSA]  Subdevices: 7/7
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
card 0: ALSA [bcm2835 ALSA], device 1: bcm2835 IEC958/HDMI [bcm2835 IEC958/HDMI]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: ALSA [bcm2835 ALSA], device 2: bcm2835 IEC958/HDMI1 [bcm2835 IEC958/HDMI1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: UCX23906524 [Fireface UCX (23906524)], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

Note the card number. Card 0 is the trashy jack on the pi. Card 1 is the usb audio interface.

### Disable the onboard audio output
To disable the useless on board jack output on the raspberry pi open up the boot config file 
```bash
sudo nano /boot/config.txt
```

Find the line that says `dtparam=audio=on` and comment it out so that it looks like this:

```bash
#dtparam=audio=on
```

Then reboot, and run `aplay -l` to verify it isn't there anymore.

## Install a realtime kernel

One of the most important things you can do to improve any linux system's audio performance is installing a realtime-kernel. On the Raspberry Pi, you need to compile this kernel yourself. [There are instructions for doing so here](https://lemariva.com/blog/2018/07/raspberry-pi-preempt-rt-patching-tutorial-for-kernel-4-14-y).

Follow the instructions, then reboot the pi by executing `sudo reboot`, wait few minutes and then ssh into the pi again. 

If you then run `uname -a` you should get a result along these lines: 

``` 
Linux raspberrypi 4.19.71-rt24-v7l+ #2 SMP PREEMPT RT Sat Apr 25 22:04:08
UTC 2020 armv7l GNU/Linux 
```

## Check your system's configuration
[realTimeConfigQuickScan](https://github.com/raboof/realtimeconfigquickscan) is a really nice script that you can use to see if your system is setup correctly. Download and run it like this:

``` 
# Scan system 
git clone git://github.com/raboof/realtimeconfigquickscan.git
cd realtimeconfigquickscan
perl ./realTimeConfigQuickScan.pl
```

## Download, and compile latest version of SuperCollider
The easiest way to compile and install the latest version of SuperCollider is to use [lvm s build scripts](https://github.com/lvm/build-supercollider). I created [my own modified version](https://gist.github.com/madskjeldgaard/10ab025993d5b77d235819d56c4cc908) of them which are tailorfit to a headless Raspberry Pi install:

{{< gist madskjeldgaard 10ab025993d5b77d235819d56c4cc908 >}}

## Links for further information
* [Raspberry Pi readme in the SuperCollider repo](https://github.com/supercollider/supercollider/blob/develop/README_RASPBERRY_PI.md)
* [Headless Raspberry Pi setup](https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0)
* [How to automatically connect to wifi on startup](https://raspberrypihq.com/how-to-connect-your-raspberry-pi-to-wifi/)
