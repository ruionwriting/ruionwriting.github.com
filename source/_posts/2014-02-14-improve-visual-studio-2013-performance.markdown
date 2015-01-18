---
layout: post
title: "Improve Visual Studio 2013 performance"
date: 2014-02-14 21:36
comments: true
categories: [Tools, Visual Studio]
keywords: "performance, visual studio 2013, tools, windows 8" 
---
We all know that one of the main causes for Visual Studio to slow down is the use of many extensions simultaneously. Extra functionality means extra stuff happening under the hood, ultimately extra processing. What can I do if I the issue isn't caused by an extension?

I've tested turning of something I really don't need and the fact is that I've noticed some improvements.<!-- more -->

## Change visual experience settings
Go to `Tools > Options`, in the default view of `Environment > General`, there is an option, under `Visual experience` called **Automatically adjust visual experience based on client performance**.
{% img center /images/2014-02-14_VS01.png Visual experience (before) %}

Turning this off will disable visual client visual experience and hardware graphics acceleration.

{% img center /images/2014-02-14_VS02.png Visual experience (after) %}

This will take effect after you restart Visual Studio.

## Startup with an empty environment
This is more a personal preference. I don't want to wait for the default environment to load with news, latest opened projects/solutions/files, etc. Latest projects and files are always available through the `File` menu options. 

{% img center /images/2014-02-14_VS03.png Startup (before) %}

Switch to empty environment and disable content download. It's strange to have content download enabled after switching to empty environment.

{% img center /images/2014-02-14_VS04.png Startup (app) %}

Now every time I start a new Visual Studio instance, I do that a lot during any normal work day, it loads clean and faster.
