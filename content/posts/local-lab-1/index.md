---
title: "Local Lab 1"
date: 2015-04-09T23:53:00-06:00
tags: ["setup"]
categories: ["homelab"]
type: "post"
toc: true
body_classes: "blog"
---

## What for?
I've been trying to fill in some of my foundational knowledge lately. I've always been the type that does better with a "learn by doing" approach, so I needed a place to test things out. I like to stay well rounded as well, so keeping with those ideas I set to plan out what I might want to play with. There are three main areas I like to focus on; system administration/configuration management, network security, and devops.

## Break it Down
### Config Management
Now that I have the overarching idea down, it's time to plan out each level. Let's start with the system administration and configuration management. We use puppet extensively at work, but I've often felt like it's not quite a complete solution. It is really good for getting things how we want and keeping them in that consistent state. The one place I feel like it doesn't necessarily do a great job is with any sort of quick, one off type of tasks.

I haven't played with it much yet (that's the point of this after all!), but it sure seems like ansible is pretty great at this. This will also give me a chance to compare puppet to something, which is always nice to keep an eye on the competitors to see if they're a better fit.

### Network Security
I've always been interested in getting more hands on with Wireshark as a means to better understand network protocols. I'll need some traffic to see and might as well do a refresher on some networking stuff, so maybe a DNS server would be fun. I hear good things about unbound.

One key component of network security monitoring is log analysis. We use splunk at work, but aren't sold on it fully (thanks to the licensing...of course), so this might be a fun time to look at something like the ELK stack, or at least graylog. With this amount of tools, some of which having a webUI, it seems like as good a time as any to play with LDAP as well, so I can set up a centralized authentication point.

### Devops
I like to be at least somewhat familiar with "devops" tools as they seem to be making great strides in our little world. I'm not a great programmer, so I tend to like more of the "ops" part of devops, but I am one for productivity tools, which seems to be a strong suit of the developers I know best. They seem to find all sorts of cool tools to help them get their job done faster and have their work flow smoother.

We've been using Vagrant at work for a while now, but to be honest, one guy set up a box for us (set up for package building - fpm ftw) and I just vagrant up, vagrant ssh, and vagrant destroy pretty much. I know I can do more, so I'm going to use that as the basis for my little digital playground here. I'll be digging in and trying to be a bit more involved in the set up past just grabbing a box and going.

That about wraps up the idea stage of this project, hopefully next time, I'll have part of this up and running and we can go over my progress.
