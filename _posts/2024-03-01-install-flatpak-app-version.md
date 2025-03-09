---
layout: post
title: "Install a particular version of a Flatpak package"
tags:
  - Flatpak
  - install
  - version
  - downgrade
---

# Overview

I had Cura break on me due to the [Fluidd](https://docs.fluidd.xyz/)/[Moonraker](https://moonraker.readthedocs.io/) [Plugin](https://github.com/emtrax-ltd/Cura2MoonrakerPlugin) I've been using.

So the simplest thing I thought to do was roll back my Cura version, sadly this is _hard_ with Flatpak and not [exactly supported](https://github.com/flatpak/flatpak/issues/3097).

# How To
First, you need to figure out what commit version of the app you need (Yep commit not version number).
```bash
flatpak remote-info --log com.ultimaker.cura
```

Once you have the commit version you need, you can upgrade to that version (Even if it is a downgrade).
```bash
flatpak update com.ultimaker.cura --commit=ac4bdf52bd8ed4d1da279e4025693e41e89dec16441b2821b792cb0452ef37b2
```

Now that the version you want is installed, we need to _pin_ or mask it so it doesn't update again.
```bash
flatpak mask com.ultimaker.cura
```

Now you are all done.
