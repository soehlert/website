---
title: "Gnu Stow for Dotfiles"
date: 2015-03-18T00:40:51-05:00
tags: ["sysadmin", "stow", "dotfiles"]
categories: ["homelab", "how to", "setup"]
type: "post"
toc: true
body_classes: "blog"
---

## Git
So far, we have figured out how to create our dotfiles to configure our CLI applications. The next question is, how do we make these available on all of our machines? This is of course presuming everyone has two laptops, a mac mini server, a linux server, a windows VM on said linux server, a VPS, and a personal VM at work (and that's just what's actually being used..). Everyone else has that right? Ok good, I'm not crazy.

So how do we manage to keep those files available for all our hosts? First step, is we use git. Not only will git centralize things on whatever git server you use (github, bitbucket, gitlabs, gogs, etc), but git is also great thanks to its merging. We use git at work quite extensively for things such as puppet modules, scripts, and config files. I love when work introduces me to new tools that can help me out at home and on other projects. I personally decided to use bitbucket for my git needs outside of work as it's easy to use, I get production uptime with no effort, and yet I can still keep my repos private. Most of my stuff should not see the light of day to be honest, no matter how effective it is for me.

For anyone who is not familiar with git, I would suggest first starting here, as I found it very useful when first trying to wrap my head around git. Git is a distributed version control system. This means, in my very layman's attempts at describing it, that it is a system used to keep track of changes to files over time, centrally. I can work on a script on my work laptop, push my changes to bitbucket, checkout that file on my home server, and I will have the most up to date version of the script. This also means that when I inevitably forget a comma in my puppet module and it fails to work, I am able to look at my last commit, and check out the differences. This makes it a little easier to track down the offending spot in the script (I swear, I'll get better with puppet-lint one day..).

## GNU Stow
Now that we have git available anywhere we want by cloning the appropriate repos, how can we use that in conjunction with our dotfiles? Well, glad you asked. On all my machines, I clone my dotfiles repo. Now all the files are there, but not in the right place. For example, ~/dotfiles/bash/.bashrc does me no good when the system is expecting ~/.vimrc in order to properly set up my vim environment. This is where GNU Stow comes into play. GNU Stow is a symlink "Farm Manager" (which I find to be...strange wording). Basically, GNU stow will symlink files into their proper place, assuming a correct structure inside your dotfiles repo.

For example, I now see something like this: lrwxrwxrwx 1 sam sam 19 Feb 24 22:38 .vimrc -> dotfiles/vim/.vimrc

Now any changes made on any of my machines and pushed into git, just take a git pull to grab and update the proper file. I have thought about putting this on some type of cron job, but I need to think about the ramifications before I just go diving into that and tonight is blogging night, not thinking night.

Installing stow is super easy, as it just requires either a: (OSX) 

```bash
$ brew install stow
```

(centos - with the EPEL repos) 

```bash
$ yum install stow
```

There are a few possible methods for setting up stow, though I decided to roll with a program based approach. Some people do a per host approach, but I felt program based would leave me with less doubling up of files.

```
$ tree -a dotfiles 
├── .git
│   ├── COMMIT_EDITMSG
│   ├── FETCH_HEAD
│   ├── HEAD
│   ├── ORIG_HEAD
│   ├── config
│   ├── description
│   ├── hooks
│   ├── index
│   ├── info
│   ├── logs
│   ├── objects
│   ├── packed-refs
│   └── refs
├── .gitignore
├── .gitlint
├── README.md
├── common
│   ├── .bash_aliases
│   ├── .bash_profile
│   ├── .bashrc
│   ├── .cz
│   ├── .git_profiles
│   ├── .gitconfig
│   ├── .gitignore_global
│   ├── .inputrc
│   ├── .mackup
│   ├── .mackup.cfg
│   ├── .profile
│   ├── .shell_prompt.sh
│   ├── .tmux.conf
│   ├── .tmuxline.conf
│   ├── .vim
│   ├── .vimrc
│   ├── rules_files
│   ├── scripts
│   └── wallpaper.jpg
├── home
│   ├── .config
│   ├── .ssh
│   └── Brewfile
├── server
│   ├── .bash_alias
│   ├── .bash_profile
│   ├── .bashrc
│   ├── .profile
│   └── .ssh
└── work
    ├── .config
    ├── .ssh
    └── Brewfile
```

This set up works best for me because I can specify which programs get stowed for each machine. I created the .profile file as a place to put all my osx specific aliases, for instance, so no reason to stow that on my linux hosts. I also only use the proxy.pac file on my work laptop, so that is the only place that gets stowed. Just a quick example of how to stow the files for, say, bash: $ cd ~/dotfiles $ stow bash

This will take all the files in ~/dotfiles/bash and put them in ~/ (directory structure is very important here). You may notice under proxy there is a directory called .config. It is required in this case because I need the proxy.pac path to look like this: ~/.config/proxy.pac

I think that about wraps it up, though I'd love to field any questions anyone has, so feel free to ask away. I'd also be wrong to not point you over to this blog, as this is a much more detailed GNU Stow post, and where I learned about stow in the first place.
