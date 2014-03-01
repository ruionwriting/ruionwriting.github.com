---
layout: post
title: "Device Emulation with Google Chrome Dev/Canary"
date: 2014-03-01 11:44
comments: true
categories: [Tools, Chrome, Web development]
keywords: "google chrome, tools, arch, windows 8, canary, dev, device emulation, chocolatey, linux"
---
This week, after seeing a snapshot that my friend 'led' posted on Facebook about some mobile tests for one of his projects, I've noticed about a certain device emulation feature  on Chrome developer tools. Hum.... that is interesting.

A simple Google search directed me right away for something I've already was aware but with some new funky stuff: 

* [Chrome DevTools, Mobile Emulation](https://developers.google.com/chrome-developer-tools/docs/mobile-emulatio)

* [Chrome Release Channels](http://www.chromium.org/getting-involved/dev-channel)

After the sexy photo we'll get on how to install Chrome Canary build on Windows 8 and Chrome Dev on Arch Linux.

<!-- more -->

## Nokia Lumia emulation in action

A simple example on Windows 8:

{% img center /images/2014-03-01_canary_on_windows.png Nokia Lumia emulation  %}

## Installing Canary Build on Windows

Canary Build run side-by-side with your existing Chrome installation. 

### Using Chocolatey (can't get more sweet)

Google Chrome Canary is available through [Chocolatey](http://chocolatey.org/packages?q=canary), so simply install it using `cinst`.

{% codeblock lang:bat  %}
c:\> cinst GoogleChrome.Canary
{% endcodeblock  %}

If you using Windows and you don't know what [Chocolatey](http://chocolatey.org/) is you are missing all the fun.

### Regular Download

Simply [download](https://www.google.com/intl/en/chrome/browser/canary.html) Chrome Canary run the installer.

## Installing Chrome Dev on Arch Linux/linux

Chrome Dev run side-by-side with your existing Chromium installation. 

This is my distro of choice but for any other popular like Debian based get the packages from [Chrome Release Channels](http://www.chromium.org/getting-involved/dev-channel).

`google-chrome-dev` is available through [AUR](https://aur.archlinux.org/), so simply install it using [yaourt](https://wiki.archlinux.org/index.php/yaourt):

{% codeblock lang:bash  %}
$ yaourt -S google-chrome-dev
{% endcodeblock  %}

## Enabling Emulation

Enable the emulation panel following simple [instructions](https://developers.google.com/chrome-developer-tools/docs/mobile-emulation#enable-emulation-panel).

## Final Thoughts

On this mobile era a lot of new tools for assisting web developers are arising. Could be a real challenge making compliant web sites/apps across the most popular devices. Having access to a good amount of devices at the same time for pre-launch tests/QA that also is usually a challenge. New tools are always welcome. 

Thank you Google for continuing to give back cool tools for the comunity and thank you 'led' for the 'tip'.
