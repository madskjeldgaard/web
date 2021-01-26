---
title: "Mutable Instruments dev environment on Arch Linux"
date: 2021-01-26T13:39:45+01:00
featured_image: /img/full/mutabledev.jpg
tags:
- cpp
- arch
- mutable-instruments
draft: false
---

![mutable instruments dev environment](/img/full/mutabledev.jpg)

[Mutable Instruments](https://mutable-instruments.net/) is an absolutely amazing synthesizer company that produces open source hardware with open source firmware on it. I have several of these and I love poking around in the firmware.

The company has even open sourced it's [development environment which on most systems may be setup in a virtual machine](https://github.com/pichenettes/mutable-dev-environment). 

But this is not so easy on Arch Linux, since these virtual machines rely on old(er) kernels, and I am always on bleeding edge kernel versions on my development laptop, so I decided to setup my own little development environment for hacking Mutable Instruments (and similar) firmwares directly in Arch with no virtual machines used and it seems to work quite nicely.

Below are some notes on how to replicate this - but if you are on anything other than Arch I would seriously recommend you to just use the Mutable Instruments dev environment in a virtual machine. 

Messing up here might do weird things to your installation, so proceed with caution.


## Install arm 4.8-3
The MI modules need a very old version of `gcc-arm-none-eabi` to work, so old in fact that there currently isn't an arch package for it. You can create your own though:

```bash
mkdir tmp
cd tmp
echo "gccver=48
pkgname=gcc-arm-none-eabi-bin-$gccver
pkgver=4.8_2013_q4_major
pkgrel=1
pkgdesc="GNU Tools for ARM Embedded Processors 4.8.3"
arch=('x86_64')
depends=('lib32-glibc')
url="https://launchpad.net/gcc-arm-embedded/4.8/4.8-2013-q4-major/+download/"
source=("https://launchpad.net/gcc-arm-embedded/4.8/4.8-2013-q4-major/+download/gcc-arm-none-eabi-4_8-2013q4-20131204-linux.tar.bz2")
license=('custom')
options=(!strip staticlibs)
sha256sums=('SKIP')

package() {
  mkdir -p $pkgdir/usr/local/arm-4.8.3
  cd $srcdir
  cp -a gcc-arm-none-eabi-4_8-2013q4 $pkgdir/usr/local/arm-4.8.3
}" > PKGBUILD
```

Then, to install you run `makepkg` from the directory of your new PKGBUILD

```bash
makepkg -si --clean
```

## Install prerequisites

Now to install the rest of the packages:

```bash
yay -S  autoconf pkgconf libusb libftdi git ncurses cowsay figlet arm-none-eabi-gcc openocd python2-numpy python2-scipy python2-matplotlib avr-gcc avr-binutils avr-libc avrdude stlink arm-none-eabi-newlib arm-eabi-4.8
```


## Symlinking
Make expects arm g++ to available in a non traditional place:

```bash
sudo ln -s /usr/bin/arm-none-eabi-g++ /usr/local/arm/bin/arm-none-eabi-g++
sudo ln -s /usr/bin/arm-none-eabi-gcc /usr/local/arm/bin/arm-none-eabi-gcc
sudo ln -s /usr/local/arm/bin/objcopy /usr/local/arm/bin/arm-none-eabi-objcopy
sudo ln -s /opt/toolchains/arm-eabi-4.8/ /usr/local/arm
```

Then, to compile you need to specify the toolchain path before the make command

```bash
TOOLCHAIN_PATH=/usr/local/arm-4.8.3/gcc-arm-none-eabi-4_8-2013q4/ make -f stages/bootloader/makefile hex
make -f stages/makefile
```

To make the command shorter and more readable you could make an alias for it:

```bash
alias mimake="TOOLCHAIN_PATH=/usr/local/arm-4.8.3/gcc-arm-none-eabi-4_8-2013q4/ make"
```


The steps above should be enough for you to compile the firmware. Now to connecting it to your computer.


## Connecting to the modules: Setup stm stuff
I connect to my MI modules using an stlink jtag USB thingy. If you aliased the make command like I recommended above, you should be good to go with just this:

```bash
mimake -f clouds/makefile upload
```
