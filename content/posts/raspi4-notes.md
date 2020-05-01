---
title: "Notes for setting up a Raspberry Pi 4 for audio work"
date: 2020-04-30T16:07:22+02:00
draft: false
toc: true
tags:
- supercollider
- raspberrypi
---

![raspberry pi 4 setup with fireface ucx and quad speakers](/img/small/raspi4-quad2.jpg)

These are notes for setting up a Raspberry Pi 4 single board computer for pro audio work. Specifically for running [SuperCollider programs for interactive installations](supercollider.sourceforge.net/).

I used [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/) for this, since my intention is to run the Raspberry Pi in headless mode, meaning using no display and no desktop environment (to save resources). 

It is presumed that you have installed Raspbian on an sd card, with ssh enabled (allowing you to log in to the pi over the network and execute commands using a terminal), and that you are logged in. 

When logged in to the Pi via ssh, we can setup everything from the command line fairly quickly.

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
Jack is used to patch audio throughout your audio system on Linux. Mostly in this case we will use it to patch audio from software to the hardware output. 

Let's install and setup jack:

```bash
# Install jack
sudo apt-get install jackd2
```
Jack has a configuration file in `~/.jackdrc` that we will set up on installation, but you can edit this anytime to tune the system's settings using a text editor by running `nano ~/.jackdrc` for example or `vi ~/.jackdrc`. The config file consists of a `jackd` command which will be run by SuperCollider when we boot the audio server in there.

```bash
# Create config file in home folder called .jackdrc
echo /usr/bin/jackd -P75 -dalsa -dhw:1 -r44100 -p512 -n3 > ~/.jackdrc
```
Explanations of the flags used here:
- `-P75` - the realtime priority of the audio
- `-dhw:1` signifies the device number used to connect to jack. You can get a list of available devices to choose from using `aplay -l` then change the number `1` to whatever you like. 1 is usually the usb card.
- `-r44100` is the sample rate. 
- `-p512` is the number of frames. This can be tuned to achieve lower latency. Must be power of two.
- `-n3` - jack's buffer periods. Jack recommends using 3 here for usb devices.

Additionally, [Fredrik Olofsson](https://www.fredrikolofsson.com) pointed out to me that it can be useful to use the `-p`-flag to restrict the number of possible ports for jack to connect to, this reduces the amount of ram used, which is why the SuperCollider readme recommends adding the `-p16` flag to the jack command in `.jackdrc`.

### Changing audio interface

Never use the built in audio jack on a Rasbperry Pi for anything serious. Instead, you can use any class compliant usb audio interface. These range from cheap 1$ chinese ones to top-of-the-line RME Fireface UCX. I use and like both of these for different situations.

To see which audio interfaces are available on the pi:
`aplay -l`

With a USB-interface plugged in, the output should look something like this:
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

Compiling a kernel takes a lot of time, but my colleague at [Notam](https://notam.no), Thom Johansen, has built a [great RT-kernel that you can download here](https://users.notam.no/~thomj/rt-kernel.tar.gz). In the guide linked above you can skip to the last bit where you install the kernel.

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

Another useful tool for testing is [xruncounter](https://github.com/Gimmeapill/xruncounter) - this may be used to stress test your system to see when dropouts happen.

## Download, and compile latest version of SuperCollider
You can run many different kinds of software on the Raspberry Pi headless: Pure Data, Csound and SuperCollider are a few free and open source options.

I personally prefer SuperCollider so that's what I'm going to use here.

The easiest way to compile and install the latest version of SuperCollider is to use [lvm's build scripts](https://github.com/lvm/build-supercollider). I created [my own modified version](https://gist.github.com/madskjeldgaard/10ab025993d5b77d235819d56c4cc908) of them which are tailorfit to a headless Raspberry Pi install:

{{< gist madskjeldgaard 10ab025993d5b77d235819d56c4cc908 >}}

### Testing your SuperCollider installation

Let's create a simple SuperCollider test file which will boot the audio server (and automatically start and connect Jack) to see if the installation went well and we are getting sound out of the pi.

```bash
# Make a SuperCollider file for testing. 
echo "s.waitForBoot{ play{PinkNoise.ar(0.5)!2} }" >> test.scd

# This will boot the server and play white noise in 2 channels
# Stop it by hitting Ctrl+d on your keyboard
sclang test.scd
```
### Editing SuperCollider files

As shown above, you can run any SuperSollider file by passing it as an argument to `sclang` on the command line.

If you just have a finished SuperCollider-file that you want to run, then this is probably fine. You can use any text editor to make small edits to your files (`vi`, `nano`, `vim`/`nvim` or `emacs` are popular choices).

But, really, this isn't a great way of working. 

My preferred way of working (on any computer, single board or not) is using a combination of NeoVim and the [scnvim plugin](https://github.com/davidgranstrom/scnvim).

To get this combination to work properly, unfortunately you cannot just install neovim using raspbian's package manager since we need a fairly new version of NeoVim to get it to work (and raspbian/debian typically has pretty old "stable" versions in it's package manager).

Follow [the instructions here](https://github.com/neovim/neovim/wiki/Building-Neovim) (look for `debian` things, this should be the same on raspbian) to build NeoVim from source. The neovim wiki is the best place to find instructions on this, but I will condense the commands I used to build neovim on raspbian below:

``` bash
# Dependencies
sudo apt-get install ninja-build gettext libtool libtool-bin autoconf automake cmake g++ pkg-config unzip

# Get the neovim source code and switch to the stable branch
git clone https://github.com/neovim/neovim && cd neovim && git checkout stable

# Build and install
make -j4 && sudo make install

# Remove source files
cd ..
rm -rf neovim
```
Now, verify that NeoVim is installed by running the following command:

```bash
nvim --version

```
If you get any output from that command, you are good to go and can proceed to install [scnvim](https://github.com/davidgranstrom/scnvim) by following the instructions on the project's github repo.

```bash
# make a neovim config file 
mkdir ~/.config/nvim
echo "call plug#begin()
Plug 'davidgranstrom/scnvim'
call plug#end()" > ~/.config/nvim/init.vim

# Install the vim-plug plugin manager
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# Open nvim and install the scnvim plugin
nvim -c ":PlugInstall"
```

Exit neovim when it's done install the plugin, then finish the scnvim integration by symlinking the SuperCollider classes that came with the plugin to the extensions folder of SC:
```bash
mkdir -p $HOME/.local/share/SuperCollider/Extensions/scide_scvim
ln -sf /home/pi/.config/nvim/plugged/scnvim/sc /home/pi/.local/share/SuperCollider/Extensions/scide_scvim
```
## Links for further information
* [Raspberry Pi readme in the SuperCollider repo](https://github.com/supercollider/supercollider/blob/develop/README_RASPBERRY_PI.md)
* [Headless Raspberry Pi setup](https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0)
* [How to automatically connect to wifi on startup](https://raspberrypihq.com/how-to-connect-your-raspberry-pi-to-wifi/)
