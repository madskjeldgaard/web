---
title: "How to install Windows VST Plugins on Arch Linux"
date: 2020-10-02T12:20:40+02:00
draft: false
toc: true
tags: 
- arch
- vst
- windows
images:
- /img/small/linvstprefs.png
---

One of the big hurdles for me when switching from a Mac based audio setup was the potential loss of access to my expensive collection of VST plugins (because a lot of developers still ignore the Linux platform as if it was 1995 even though it has never been easier to make and release cross platform plugins). Little did I know that almost all of them would run just fine on Linux using the `wine` Windows compatibility layer and the `linvst` VST plugin bridging software. 

The workflow for getting set up though was quite difficult before I switched to Arch based distributions than on other Linux distributions (thanks to the Arch User Repository (AUR)), and even though it is significantly easier on Arch, it still has some quirks in the setup process which I have tried to collect here as a set of instructions and notes for getting started.

__Disclaimer__
There may be licensing problems in doing this operation on some plugins. I talked to a commercial plugin developer about using their plugins on Linux and they said that it sounded interesting (because they developed everything on Linux anyway (but did not make releases for Linux)) but that I was probably in conflict with their licensing when doing so. I don't take responsibility for anyone breaking any laws here.

## Preparations

### Enable 32 bit repositories

To install `wine` (and enable 32 bit support) you need to do enable the `multilib` repositories in your pacman configuration.

By doing this, you get access to a range of 32 bit software that can be run on a 64 bit machine. 

Open up `/etc/pacman.conf` in your favourite text editor. E.g. in vim:

```bash
sudo vim /etc/pacman.conf
```
And then uncomment the multilib section:

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

And then upgrade your system:

```bash
sudo pacman -Syu
```
See [the arch wiki for more information](https://wiki.archlinux.org/index.php/Official_repositories#multilib).

### wine

To be able to run Windows software on Linux, you need to install `wine`. Wine is a huge subject in itself (and I definitely do not master it at all), but it basically runs as a compatibility layer between windows software and your linux system.

If you followed the previous step by enabling multilib, you should now be able to install wine like this:

```bash
sudo pacman -S wine
```

## Setting up linux VST compatibility

The main software used to enable windows VST plugins is [linvst](https://github.com/osxmidi/LinVst). It is used to create bridges between the windows `.dll` files that VST plugins are delivered in and creates a Linux equivalent `.so` file. Managing this process can be complicated and a bit of a pain, but thankfully there is a program to do it all for us: [https://github.com/Goli4thus/linvstmanager](linvstmanager).

Linvstmanager is the main program we use to keep track of vst plugins, where they are and if they are supported on our system.

There are packages for linvstmanager and all of the linvst bridges on [the AUR](aur.archlinux.org) that can be installed using `yay` or another aur helper system:

```bash
yay -S linvstmanager-git linvst-bin linvst3-bin linvst-x-bin linvst3-x-bin
```

### Setting linvstmanager preferences
![linvstmanager preferences](/img/small/linvstprefs.png)

Before we are able to use linvstmanager, we need to tell it where the vst bridges we installed along with it are in our system.

If you used the packages from the AUR, they are probably in `/usr/share/LinVst`. Select and find them in your preferences.

Next up you need to set a folder that will contain your plugins. This is called the __Link folder__. This is the folder your DAW needs to read from when scanning for VST Plugins. I have put mine in a hidden folder called `.vst-win` in my user's home directory but you can put it anywhere.

## Installing a plugin

This is the workflow I use for downloading and installing a plugin. See the [linvstmanager documentation](https://github.com/Goli4thus/linvstmanager) for a more thorough explanation.

### Part 1: Install the plugin to your wine drive
1. Go to the developer's website and download the `.exe` file for installing the plugin on a Windows system.
2. Open up a terminal and execute `wine path/to/plugin.exe`
3. Follow the install instructions of the vst plugins you are installing. Note where you are putting the `.dll` and/or `.vst3` files at the end of the installation.

This will leave the plugin files in your wine installation's "C" drive which is probably in your home folder: `~/.wine/drive_c`

### Part 2: Find and enable the plugins in vstmanager
![scan process](/img/small/linvstscan.png)

1. Open `linvstmanager` to create a bridge to your new plugin.
2. Scan for your plugins (press `ALT+s` or go to the menu `Edit->Scan`)
3. Choose the scan folder by clicking `select`. If you don't see any hidden folders (starting with a `.`), you can right click and choose `show hidden files`. Then go to `.wine` and `drive_c`.
4. Once the scan is finished you should get a list of files. Find the ones you want to use and press `s` to select. 
5. Click `Add`

### Part 3: Enable the plugin

In the main window of linvstmanager you should now see your plugin listed with a `no so` warning next to it.

1. Update by pressing `u` or going to the menu `Edit->Update`
2. Right click on the plugin you just added and click enable. It should now turn green.

### Part 4: Save settings

Before finishing off, save the settings.

Go to menu `File->Save` or press `Ctrl+s`

## Scan for the plugins in your DAW
![linvst settings in Reaper](/img/small/linvst-reaper.png)

You should now be able to use your newly bridged Windows VST plugins in your DAW. Point it to the link folder you set in your linvstmanager preferences and scan for the plugins.

In [Reaper](http://reaper.fm/) (which, by the way, has an amazing Linux native installation) for example, you can add the Link folder as a path in your Preferences. 
Find the `VST` tab and add the __Link folder__ path you set in linvstmanager. 
The syntax for this consists of full path names with semicolons (`;`) in between: `/a/full/path;/another/full/path`

After adding the path, click `re-scan`.

## An alternative approach: Using Carla
An alternative approach is to use [carla](https://kx.studio/Applications:Carla), which has been [outlined nicely by kindohm / Mike Hodnick in this post](http://blog.kindohm.com/posts/2020/2020-06-06-vsts-on-linux/).
