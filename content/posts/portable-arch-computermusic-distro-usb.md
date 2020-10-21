---
title: "A portable Arch Linux Computer Music system on a USB drive"
date: 2020-10-10T17:17:22+02:00
draft: false
images:
- /img/full/archlive.jpg
- /img/small/archlogo.png
tags:
- arch
- linux
---
![my usb](/img/full/archlive.jpg)

[Inspired by this blog post](http://jamesmcm.github.io/blog/2020/09/09/alma/#en), I have been toying with a silly idea lately: A completely portable Arch Linux distribution set up properly for computer music, but flashed onto a USB thumb drive so that I may pop it into any computer, boot from the thumb drive and start creating music on whatever hardware is in front of me.

I have achieved this now using the awesome [alma](https://github.com/r-darwish/alma) project. Alma is a tool for creating persistent USB installations of Arch Linux using a series of configuration files and bash scripts called _presets_. See [my alma presets here](https://github.com/madskjeldgaard/arch-linux-computermusic-usb).

The LiveUSB environment runs exactly like a native installation on your hard drive and it comes with `yay` installed, making it super easy to install new software from the [aur](https://aur.archlinux.org).

The goal with this is to have a solid base for people to flash onto their drives and then let them install their own dotfiles and additional software as needed.

## Features
My [alma](https://github.com/r-darwish/alma) preset installs a fully usable and persistent Arch Linux system on a LiveUSB. 

It is targeted at users of [tiling window managers](https://en.wikipedia.org/wiki/Tiling_window_manager) and is set up with two of the most popular choices for these.

Further, the preset contains a range of software preconfigured for computer music production and audio work including:

- NeoVim
- [SuperCollider (with sc3-plugins)](https://supercollider.github.io/) (with [scnvim](https://github.com/davidgranstrom/scnvim))
- [Csound](http://csound.org/)
- [tidal](https://tidalcycles.org/) (with [tidalvim](https://github.com/tidalcycles/vim-tidal))
- [Pure Data](https://puredata.info/)
- [Reaper](https://reaper.fm/) (not including in the .img file below for licensing reasons but easily installed using `yay -S reaper-bin`)
- [Sox](http://sox.sourceforge.net/)
- [Flucoma](https://flucoma.org/)
- Both the Sway and i3 tiling window managers (for Wayland and X, selectable on login)
- Jack (with both the GUI jack manager `qjackctl` and the terminal based `njconnect` patcher)
- Realtime kernel (and added realtime priviliges for your user)

## Usage
There are two ways of installing and using this.

### The easy way: Flash my preconfigured image onto your drive

Using something like [balena etcher](https://www.balena.io/etcher/) or `dd` you can flash a `.img` file to a drive.

I have made a preconfigured image file that you can download for that purpose.

The image comes with a default user:

```
User: computer
Password: music
```

Flash it on to your drive, choose the drive as the boot device in your BIOS and start making noise.

[Download the preconfigured image here](http://users.notam02.no/~mads/archstuff/arch-computermusic-usb.img.gz) (the deflated image file is 10GB so should fit on anything larger than that).

### The hard way
[The full alma preset is available here including instructions for creating an image file](https://github.com/madskjeldgaard/arch-linux-computermusic-usb).

See [the alma project](https://github.com/r-darwish/alma) for information about installing it.

## Logging in for the first time
At the login screen, in the top right corner you may choose between `i3` (X) and `sway` (Wayland) as your window manager.

After choosing your window manager, type your credentials and log in.

On first boot, open a terminal and run `topgrade` to upgrade everything on the system to latest version.

#### When running tidal the first time
Before running tidal cycles, open up a .scd file and run `Quarks.install("SuperDirt")` to get SuperDirt installed. The rest of the setup should work out of the box.


