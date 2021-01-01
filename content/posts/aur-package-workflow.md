---
title: "Notes on creating packages for the Arch User Repository (AUR)"
date: 2020-09-19T22:51:04+02:00
draft: false
toc: true
tags: 
- arch
- linux
images:
- /img/small/archlogo.png

---

![arch logo](/img/small/archlogo.png)

These are notes I have been taking while making packages for the [Arch User Repository](https://aur.archlinux.org), a community based package repository for users of Arch Linux and it's derivative operating systems. This isn't a complete guide and certainly not beginner friendly but should help some people with some linux experience get started.

If you have a tolerance for YouTube videos, then maybe [this video will be helpful to you](https://youtu.be/crnGzF43aoc).

It is also recommend to read the official guide on how to create packages: [Creating packages](https://wiki.archlinux.org/index.php/Creating_packages).

The manual for `PKGBUILD` is a nice resource for information as well:

```
man PKGBUILD
```

## Creating a repository
This was one of the steps of creating an AUR package that confused me the most. How do you "create a repository" like you would on Github by logging in to the web interface and clicking a button that would create a repo?

Well, the way to do it is: 

1. clone an empty repo from the AUR's git repository named after your package
2. then adding your PKGBUILD and .SRCINFO to that and 
3. pushing it back to AUR.

## Step 0: Setup AUR user account + ssh.
Before continuing, you need to create an account on [aur.archlinux.org](https://aur.archlinux.org).

Then, you need to set up your ssh key pairs. 

See this [link](https://wiki.archlinux.org/index.php/AUR_submission_guidelines#Authentication) on how to make the key pairs.

Then add your public key to your aur user [here](https://aur.archlinux.org/account).

## Step 1: Clone the AUR repo
The cloning/creating of the repository looks like this (change the `PKG_NAME` variable to contain the name of your package):

```bash
PKG_NAME="wowzers"
AUR_GIT_URL="ssh://aur@aur.archlinux.org/$PKG_NAME.git"
git clone $AUR_GIT_URL
git remote add origin $AUR_GIT_URL
git fetch
cd $PKG_NAME
```

## Step 1.5: Avoid messing up your git stuff
Inside of the folder that is about to contain your package, create a `.gitignore` file setup to ignore everything. This way you have to explicitly add and commit files to your package's git history to have them take effect.

```bash
# Only add files explicitly using -f flag with git, eg git -f add <file>
echo "*" > .gitignore
```

## Step 2: Write a PKGBUILD 
Write a [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) file. This is a script describing the name of your package, it's source, dependencies and how to install it among other things.

I won't cover how to write a PKGBUILD in detail here but you can copy a full PKGBUILD from your system like so:

```bash
cp /usr/share/pacman/PKGBUILD.proto PKGBUILD
```
Be warned though that this is a full PKGBUILD containing tons of things you don't actually need to worry about. 

[Here you can find an explanation](https://wiki.archlinux.org/index.php/PKGBUILD) of all the PKGBUILD's keys and functions.

But honestly, the best way to learn about the necessary things in a PKGBUILD is to search [aur.archlinux.org](https://aur.archlinux.org) for a package similar to yours. Then click it and then find a link to "VIEW PKGBUILD".

Don't forget to add your PKGBUILD to your git repo:


```bash
git add -f PKGBUILD
```

### An important trick: Using the install command
In the `package(){}` function of your PKGBUILD, you can use the `install` command to copy things into the system.

A lot of packages are simply copying files (binaries, shared libraries and or documentation for example)  like this, and it is best done using the `install` command because it can copy the files and set it's permissions in one go.

Some examples:

```bash
# Copy package binary to system
install -Dm755 "$srcdir/package/amazingbinary" "$pkgdir/usr/bin/amazingbinary"

# Copy shared library to system
install -Dm755 "$srcdir/package/somesharedlibrary.so" "$pkgdir/usr/share/somesharedlibrary.so"

# Copy package license to system
install -Dm644 "$srcdir/package/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

# Copy some documentation to the system
install -Dm644 "$srcdir/docs/explanatory-documentation.html" "$pkgdir/usr/share/doc/$pkgname/explanatory-documentation.html"
```

Also: If the system directory you are copying to does not exist, `install` will create it if you use the `-D` flag like the example above.

For a full list of what to install where, [see this wiki page](https://wiki.archlinux.org/index.php/Arch_package_guidelines#Directories).

## Step 2.5: Prepare for release

### Making the package
In the folder containing your PKGBUILD you can run the following command to make the package and install it:

```bash
makepkg -s
```

This will download the sources, create a .tar file and a file/folder structure like in this example for [linvst3-bin](https://aur.archlinux.org/packages/linvst3-bin/):

```bash
linvst3-bin
├── LinVst3-2.1-Debian-rz.zip
├── linvst3-bin-2.1-1-x86_64.pkg.tar.xz
├── pkg
│   └── linvst3-bin
│       └── usr
│           ├── bin
│           │   ├── linvst3convert
│           │   ├── linvst3converttree
│           │   ├── lin-vst3-servertrack.exe
│           │   └── lin-vst3-servertrack.exe.so
│           └── share
│               └── LinVst
│                   └── linvst3.so
├── PKGBUILD
└── src
    ├── LinVst3-2.1-Debian-rz
    │   ├── convert
    │   │   ├── linvst3convert
    │   │   ├── linvst3converttree
    │   │   └── ReadMe
    │   ├── embedded
    │   │   ├── lin-vst3-servertrack.exe
    │   │   ├── lin-vst3-servertrack.exe.so
    │   │   └── linvst3.so
    │   ├── License
    │   └── ReadMe
    └── LinVst3-2.1-Debian-rz.zip -> /home/mads/code/arch-packages/linvst3-bin/LinVst3-2.1-Debian-rz.zip

10 directories, 17 files
```

Note: The `src` directory is available in your PKGBUILD script as the `$srcdir` variable.

For more info on [the makepkg duties, see this.](https://wiki.archlinux.org/index.php/Arch_package_guidelines#Makepkg_duties)

### Installing the test package

By adding the `i` flag to the `makepkg` command your package will be installed on your system using pacman.

From the package directory, run:
`makepkg -si`

### Clean up your mess
When using `makepkg -si`, the package will actually be installed on your system. Make sure to clean them up afterwards using pacman: 

```bash
sudo pacman -R <pkgname>
```

### Test

Check sanity of package using the `namcap` tool (which you will need to install if you haven't already):

```bash
namcap PKGBUILD # If this returns nothing, it is okay
namcap name-of-pkg.pkg.tar.xz
```

### Update checksums
Update package checksums. This will be inserted into the PKGBUILD file.
`updpkgsums`

### Create .SRCINFO
Make the `.SRCINFO` file
```bash
makepkg --printsrcinfo > .SRCINFO
git add -f .SRCINFO
```

## Step 3: Publish

Do the final git stuff:
```bash 
git add -f .SRCINFO PKGBUILD
git commit .SRCINFO PKGBUILD -m "first commit"
```

Then push:
`git push`

## Step 3.5: Updating a package
After updating a PKGBUILD, you need to regenerate the `.SRCINFO` file and then commit it with the PKGBUILD that was changed:

```bash
makepkg --printsrcinfo > .SRCINFO
git add -f .SRCINFO PKGBUILD
git commit .SRCINFO PKGBUILD -m "important change"
```



---

## Random notes

Here are some random things I learned by doing them the wrong way on the AUR (and getting feedback from more experienced contributors).

## Naming packages based on version control sources (like Github)
[... is covered here](https://wiki.archlinux.org/index.php/VCS_package_guidelines)

### Make-flags should be avoided

You shouldn't use flags when `make`'ing software in an AUR package. A common way to speed up the compilation process is to add the `-j4` flag for example to utilize 4 cores. On Arch that is done globally for packages in the `/etc/makepkg.conf` configuration file, eg.:
```
MAKEFLAGS="-j4"
```
### No need to unzip/untar sources
This is handled by the package system for you. 

### An AUR package cannot touch a user's home directory
Some software needs to go into the home directory of the user. This is not allowed on the AUR.

### Do not include dependencies covered in the `base-devel` group
Make sure the `makedepends=()` array does not contain things already covered in the `base-devel` group (like `gcc` and `make`):

[https://wiki.archlinux.org/index.php/PKGBUILD#makedepends](https://wiki.archlinux.org/index.php/PKGBUILD#makedepends)

```bash
# List all packages in base-devel group
pacman -Sg base-devel
```

### Using git hash as version number

For packages based on version control systems such as git (these packages normally have a name ending in `-git`), a trick to update the package version is to create a function that extracts the git hash automatically.

Add this to your PKGBUILD:
```bash
pkgver() {
	cd "$srcdir/${pkgname}"

	printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}
```
Quit your text editor, then run `makepkg -s`. When you reopen the PKGBUILD file in a text editor, it now has a git hash as the version name.

### Cloning and locally updating an out-of-date package

Sometimes packages aren't updated as regularly as you would want. 
If there is a specific package you want the latest version of, but the maintainer has become unresponsive, you can download it to your computer and create a local version of the `PKGBUILD` file of the package.

As an example, let's download `reaper-bin`

```bash
PKG_NAME="reaper-bin"
AUR_URL="ssh://aur@aur.archlinux.org/$PKG_NAME.git"

# Remote
git clone $AUR_URL
cd $PKG_NAME
```

Then open the `PKGBUILD` file in a text editor of your choice
```bash
nvim PKGBUILD
```

Then change the `pkgver` variable to the version number you would like.

Exit your text editor and still from within the directory with the PKGBUILD in it, run
`updpkgsums` to update the checksums in the package.

And then, if all is well, you should be able to install the package like so:
```bash
makepkg -si --clean
```
