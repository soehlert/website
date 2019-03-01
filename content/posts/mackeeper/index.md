---
title: "Mackeeper"
date: 2015-03-04T23:46:50-06:00
tags: ["security", "virus"]
categories: ["how-to", "story"]
type: "post"
toc: true
body_classes: "blog"
---

## Background
So I heard from a friend that they were having some computer issues. I agreed to take a look and asked what was going on. Their description of things was a little strange, but I already could see in my head what was going on…so I thought. “I’m seeing lots of popups, I keep getting warnings about viruses, and my computer is not running right” (paraphrased). Sounded like any old windows virus, nothing a quick offline scan wouldn’t likely fix. So the day comes, and the laptop is dropped off for me. Surprise, surprise, it’s a Macbook Air. This was unexpected.

## First look
So first things first, let’s log in and see what we’re dealing with. Off the bat, I see a few downloads and that’s all it takes. I see Mackeeper and know that’s what we’re dealing with. Hopefully that’s where it ends. Then I see “Logmein-support-setup.dmg” and immediately, my heart sinks. The user mentioned hearing from someone that they had lots of hard drive issues and it was going to take $400 dollars to fix. That sounded strange to me, but I presumed the user had been talking to Geek Squad or some other useless, overpriced “help.” Now I saw what really happened. The user got in contact with “support” from MacKeeper who made up some issues to try and scare a few hundred bucks off them. Great, hopefully that’s all they did. I’m writing this as we go, so let’s see what we find.

## Let's Dive In
First off I see lots of people suggesting to just drop the .app dir in the trash and that would bring up MacKeeper’s uninstaller. Ya, I’m not gonna trust that one. I see some others suggest the files that MacKeeper uses. It looks suspiciously small and incomplete; let’s attack this more manually.

```bash
~ root# ps aux | grep -i keeper
```

This brings back something running as the user from the MacKeeper.app dir and two things running as root out of /Library/Application\ Support/MacKeeper. Hmmm. Let’s start with the first one.

```bash
~ root# lsof -p 244
```

This shows a LOT of stuff. Nothing too exciting, though starting to see mentions of zeobit. Let’s keep that one in mind, could be useful. A few firewall related cache files (note to self, let’s check fw). Aaaaand then there’s this.

```bash
TCP localhost:49152 (LISTEN).
```

Welp. That’s no good. Firewall check? It’s off. Grr. Well, I’ll fire off a tcpdump session for that port, just in case something comes up while I’m poking around.

Looking at the files open for the two root processes, not much turns up really. Everything seems centered off /Library/Application\ Support/MacKeeper/ so a quick rm -rf and that should be gone. The rest of the files open for the root processes are cache and tmp dirs. Nothing too exciting living in there, and they’ll go away when I reboot. Seems like I should reboot and see where we’re at since I can’t telnet to port 49152 (even with the firewall off) and tcpdump coming up empty.

## Are We Good?
Reboot and check for anything related to MacKeeper or zeobit. Check active connections with

```bash
~ root# lsof -i
```

and I see a few things connecting to Apple HQ, some mDNS stuff, airport stuff, and ntp stuff. Looking much better now. Let’s check out running processes and see if anything comes up. Nope, it all looks clean (quite a bit of googling to see what each one is, but they all look like expected Apple-brand stuff. Let’s check permissions with Disk Utility booted from the install disk. Things are a little messed up, not sure if it’s related or not, but lets fix that just to be safe.

Unfortunately I can’t check for rootkit’s as I can’t find software compatible with this newest version of OSX. I ran a pass with Clam AV and it all came back clean. The last thing to do is remove anything that might be left in Safari, cookies and whatever else I might find.

All in all, this turned out a lot easier to deal with than I thought at one point. It would have been nice to run a rootkit check, but I would be willing to bet that the Logmein-support people were only doing that to “prove” they were “finding issues” and not that they were actually installing backdoors or anything.
