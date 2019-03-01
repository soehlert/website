---
title: "Yubikey"
date: 2016-02-07T23:29:44-06:00
tags: ["sysadmin"]
categories: ["how-to"]
type: "post"
toc: true
body_classes: "blog"
---

# Yubikey Fixes
There are plenty of guides out there on how to set up a Yubikey to use as an ssh key, however, I have found myself looking for much more of a "what to do if things go wrong" guide. All fixes listed here feel...dirty to me, but so far are the only ways I have been to get things working again. I'm partially posting this for myself since these "fixes" are cobbled together from multiple sites and random testing. Please comment if you have a better way to get things working again.

## The Problems
I use yubikeys pretty extensively for ssh, and sometimes run into so issues. To be honest, I haven't exactly figured out what is happening, but I like to say it gets "stale." I am often logged into many places over ssh and also almost always have a socks proxy open over an ssh tunnel.

When things get stale, the symptoms I see are one of two; 1) all my connections are dropped and all tabs and windows are back to my local prompt (I did say I used a lot of ssh/yubikey right?) or 2) my tunnel is still logged in, however, I can not make any hops past that. The second symptom is because my ssh key is no longer loaded, but my tunnel has not died yet.

## The Solutions
I have found a few different ways to get things working again, though I still haven't figured out exactly which order or mix of solutions will work when.

Unplug/replug yubikey. Sometimes this will get the ball rolling again.

Find and kill the current running process. Start it back up. (Possibly also unplug/replug) 

1. `ps -ef | grep gpg` 

Insert the pid you found in the last step. Kill -9 seems drastic, but kill/killall does not actually kill the process in this state. 

2. `kill -9 $pid_of_gpg`

added benefit of testing to make sure card is properly seen

3. `gpg --card-status`

Open new tab/window in your terminal emulator. This has the benefit of setting up a new ssh agent, loading the key again.

A mix of any of the above.

Again, I'm not very happy with these fixes, and would love to hear what you have to say. I've seen these issues on OSX 10.11 and 10.10 using built in terminal and iTerm2.
