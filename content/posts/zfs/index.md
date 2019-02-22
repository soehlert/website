---
title: "Index"
date: 2019-02-21T23:01:27-06:00
draft: true
tags: ["sysadmin", "zfs"]
categories: ["how-to", "setup"]
type: "post"
body_classes: "blog"
---

## Why ZFS
The easy answer for me is the fact that we are getting some new storage boxes that will be running ZFS. Since they will fall under my purview and I don't have experience with it, seems like as good of a time as any. I also made the mistake I think all sysadmins have done at some point or another; you create your partitions expecting one to grow faster than the other, but it turns out you have it backwards. Since my partitions had to be redone anyways, it seemed the perfect time to give ZFS a go.

Now why might someone else be interested in ZFS?

1) Pooled storage - This is nice because you can combine multiple disks into one pool. For example, I was able to take 4 disks 4TB large, and turn it into ~6.9TB of usable space after mirroring and striping...which leads to #2

2) Raid is built in. ZFS handles the raid array for you. It also introduces raidz(1|2|3) as choices as well. In a similar vein, LVM is not necessary with ZFS thanks to the pools. This also means ZFS is aware of the physical distribution (the raid array), the logical distribution(the pool), and it

3) Handles the filesystem. This is nice since you can use one set of tools to handle every level of your data. ZFS also makes it possible to set quotas (if you want) and manage attributes at a per filesystem/mountpoint level.

4) Data Integrity - ZFS helps integrity in a couple ways. * Checksumming all blocks with the added bonus of self healing abilities helps to avoid the issue of silent data corruption. * Copy on write storage. This means instead of directly modifying the data block, the block is copied and that is what gets edited. After writing, the pointers are updated to point to the new block and the old block is freed.

Those were the main reasons ZFS intrigued me, though others may also be interested in the snapshots, hybrid storage pools, compression, data deduplication, or its capacity limit (theoretically- 256 quadrillion ZB).

## An Example
Now I'll walk through how I got myself set up and running with ZFS. My server is running Centos 6.6, so I will be using ZFS on linux which has quickly solidified itself according to some real world testing some at my work have done. These people use it extensively, so I figure it must be ready for primetime if they've had this good of luck with the newest releases.

First we need to get it installed (installation instructiosn taken from ZFS on Linux).

```bash
$ sudo yum localinstall --nogpgcheck https://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ sudo yum localinstall --nogpgcheck http://archive.zfsonlinux.org/epel/zfs-release.el6.noarch.rpm
$ sudo yum install kernel-devel zfs
```

Now we want to make sure ZFS is on at boot.

```bash
$ sudo chkconfig zfs on
```

Currently, selinux support is not implemented, so you'll need to turn that off for now. Setenforce does not persist so we'll need to run this command and edit the configuration file.

```bash
$ sudo setenforce 0
$ sudo vim /etc/selinux/config
Now let's create our zpool. To see what disks are installed you can run this command (this will help you know which ones are meant for the zpool)

```bash
$ sudo fdisk -l
```

Once we know that, we can run our zpool command.

```bash
# zpool create tank raidz2 /dev/sda /dev/sdc /dev/sde /dev/sdd
```
A few notes on that command: tank is very commonly used as a play on the word "pool" (think fish tank). I decided to go with raidz2 which is mirrored and striped like a raid6 array. I had added four disks for use in the pool, so they are called out there at the end. Change those to suit your needs.

To verify it worked, you can run this command.

```bash
# zfs list
NAME                    SIZE    ALLOC   FREE    CAP  HEALTH     ALTROOT
tank                     80G    137K     80G     0%  ONLINE     -
```
You should see something like this (the size will be different based on what you've used).

Now that we have our pool set up, we need to create a filesystem.

```bash
# zfs create tank/new
```

New can be whatever you want. I use new as a file dump location on my server. Any time I need to add new files to my server it gets scp or rsync from my laptop to this directory. Feel free to do this for whatever filesystems you would like. Now to set where the filesystem is mounted (you can do per FS with ZFS) you would run a command similar to this.

```bash
# zfs set mountpoint=/new tank/new
```

Now my filesystem named "new" is mounted at /new which is easy for me to find and remember. Again, do this as needed for the filesystems you set up.

## Configuration
This would be a good time to set any attributes you'd like. In order to see the attributes currently set on a filesystem, you use the command:

```bash
# zfs get all tank/new
```

To set attributes, you would do a command just like we did for setting the mountpoint.

Another good command to know is the zpool status command. This lets you keep an eye on how your pool is doing.

```bash
# zpool status
    pool: tank
  state: ONLINE
    scan: none requested
  config:

       NAME        STATE     READ WRITE CKSUM
        tank        ONLINE       0     0     0
          raidz2-0  ONLINE       0     0     0
            sda     ONLINE       0     0     0
            sdc     ONLINE       0     0     0
            sde     ONLINE       0     0     0
            sdd     ONLINE       0     0     0

  errors: No known data errors
```

## Conclusion
Well there you have it. That is how I got up and running with ZFS. Some places to go from here are definitely look at the possible attributes you can use per FS and find out what works best for you. If you have any questions or comments, feel free to share.
