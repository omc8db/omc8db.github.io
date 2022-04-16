+++
title = "Disk Spanning + SSD caching with LVM (to hold all my video games)"
date = 2022-04-16

[taxonomies]
tags = ["linux"]
+++

I use Linux every day at work. It's easily the system I'm most comfortable in. That's why it was kind of a shock to realize I don't run Linux at home anymore.

My laptop is a 2014 macbook that I don't tinker with at all. Everyone should have a system that they don't mess with. You never know when you'll need it to get yourself out of trouble.

My desktop runs windows 10, because *video games*.

The last time I messed with Linux for gaming was back in high school, around 2010 probably. I remember spending several hours tweaking Wine to try to get Command and Conquer Generals running, only for it to get about 0.5 fps in OpenGL (it ran fine in Windows) and then break my whole desktop until I hard reset it. The experience was hot garbage.

A few weeks ago my friend Caleb casually mentioned that he'd been running his gaming desktop exclusively on Linux for months without problems. That was surprising, because Caleb is not a programmer. I did some googling. According to the internet at large, the state of Linux gaming in 2022 is night and day better than the last time I messed with it. 
* Wine is now Proton, with actual paid engineering staff at Valve maintaining it and commercial products that use it (go Steam Deck!). 
* Drivers work for just about everything out of the box. 
* Vulkan has made it possible to write low-level GPU code instead of trying to map between two complex and leaky abstractions (DirectX->OpenGL translation), and some plucky open source devs [implemented the whole DirectX API on top of Vulkan](https://github.com/doitsujin/dxvk). That alone gives me much higher hopes.

### Maybe it's time to give Linux gaming another shot

This would be a boring blog post if I just said "I installed Linux and it ran my games", so let's do something interesting instead. Let's see if I can improve my experience in a way that's impossible on Windows.

My desktop is actually pretty old, and it was built before I was making tech money. I haven't felt any need to upgrade since then, because an i5-2500k and GTX 970 can still run any game I want.

One of the downsides of being so old is the storage. It has an SSD for the primary drive, but SSDs weren't cheap back in the day. I have
* A 250 GB SSD
* A 1 TB HDD
* A 500 GB HDD

In Windows I made all these drives different volumes and used [Steam Library Manager](https://github.com/RevoLand/Steam-Library-Manager) to migrate games to the SSD when I wanted to play them. This worked, but it was inconvenient. If I wanted to play a game that was on my hard drive, I had to drag it over to the SSD and wait up to half an hour for it to sync. To download a new game, I had to pick which one to kick from the SSD and move back to my disk. And since there were two disks, I had to pick which one to move it to.

# Can I make all my hard drives look like one volume with the SSD space as transparent cache?

I think Logical Volume Manager (LVM) should be able to do this.

Disclaimer: A 1TB SSD is like $80 these days. Buying your way out of this problem is the better answer. I'm doing this for fun, to prove I can.

# The Plan

I'm not going to risk slowing down or corrupting the main OS. That's getting a regular-old 50GB partition at the start of the SSD. All the fun stuff I'm doing here will only be for my /home/ directory, where the games and data files are stored.

I'm going to create an LVM span that covers both the 1TB and 500GB disks and then set up the remaining 200GB of my SSD as a cache partition.

Will 50GB be enough for my primary partition? Experience says yes, but I'm not worried! LVM will let me shrink the cache and expand the primary up to the size of the full SSD, and then span onto another SSD if needed. So it doesn't matter if I guessed wrong.

## Other options

LVM is one of 3 options I'm aware of for doing fancy multi-volume stuff like this.

* OpenZFS also supports spanning, splitting and cache disks. However it's not in the mainstream kernel and every guide I've seen requires a good deal of tinkering to get even a regular disk working. It also chugs RAM like crazy.
* BTRFS might do this. I know it has some volume manager features, but it's different from anything I'm used to and I'm still not convinced of its maturity. It's not hard to find forum posts even from this year where people claim it ate their data without warning. I won't have anything irreplacable on this system but it would still be annoying to deal with.
* LVM is old, mature, in the mainline kernel and well documented. It's what I'll be trying first.

# The Install

I picked Ubuntu 20.04 for my distro. It's a long term support release so the install should be good for several years. Ubuntu 22.04 is also a long term support release and it's coming out next week, but it's switching the default display manager from Xorg to Wayland. The steam deck runs on Xorg and the Nvidia drivers are well tested with it, so sticking with this slightly older but still supported release will likely prevent me from getting hit with teething pains for new software.

I do not like rolling releases because I hate doing incremental maintenance on my systems, so Arch is out. I want all my core changes to come in a well tested bundle so I can spend an hour every 4 years doing a full reinstall instead of spending 30 minutes every week debugging a new package breaking change or incompatibility.

Before starting, I backed up all my data to an external hard drive and made sure my photos were synced with Google. If things go wrong I feel safe knowing I can nuke everything and roll back to how things were.

I downloaded the xubuntu 20.04 iso and wrote it to a flash drive with Rufus.

Next I deleted all my partitions to start with a blank slate and told Ubuntu to install as normal on the SSD, with LVM enabled.

The installation was smooth, and the system rebooted into my shiny new Linux desktop.

## Doing the LVM Magic

Now that I had a functioning system, it was time to set up my LVM pools.

I started by shrinking the main partition from 250G to 50G so I'd have space for the cache.
This is where I hit the first speed bump: You can't shrink a live ext4 partition, and I can't unmount root while I'm using it.

I rebooted back into the livecd and resized the partition [using the guide from red hat](https://access.redhat.com/articles/1196333).

So far so good.

Let's start by creating a logical volume that spans both hard disks.
I read the documentation from `man lvmcache`, and was slightly confused. 

LVM has a hierarchy of types
* Physical Volume (PV): A real disk. All three of my disks must be PVs
* Volume Group (VG): A collection of PVs you can span across
* Logical volume: A partition. I will need a logical volume for cache and a logical volume for data.

It's not clear if my cache and data logical volumes need to be in the same volume group. Should I extend my primary volume group to include the slow disks, or create a separate one for them?

I decided to put my SSD on one volume group and both HDDs in a separate one.
```bash
pvcreate /dev/sdb /dev/sdc
vgcreate data /dev/sdb /dev/sdc
```

At this point I got a warning: "Devices have inconsistent physical block sizes (4096 and 512)"

Uh oh. Do I need to do anything about that? Can I even change a device's physical block size?
Let's try repartitioning both disks.

I see [this stack overflow post](https://access.redhat.com/articles/1196333) about how to change block sizes.

Can I use this to set both hard disks to the same value?

```
# blockdev --getbsz /dev/sdb
4096
# blockdev --getbsz /dev/sdc
4096
```

Wat. They're already the same?

I'm going to get a second opinion: [Does LVM need a partition table?](https://serverfault.com/questions/439022/does-lvm-need-a-partition-table)

So apparently LVM will let me create a PV from a raw block device but it's not recommended. I'm going to use gparted to partition both these disks as GPT/LVM PV/4k blocks.

Now `pvdisplay` shows sdb**1** instead of sdb and sdc**1** instead of sdc. This is probably better.

```
# vgcreate vgdata /dev/sdb1 /dev/sdc1
WARNING: Devices have inconsistent physical block sizes (4096 and 512).
```

I don't know what it wants from me or how to fix this, so I'm going to ignore this and move on. I have a working VG. Probably.

From here I'm going to try to follow the instructions in `man lvmcache` to set up my drive. I also read [Luc de Louw's blog post on storage tiering](https://blog.delouw.ch/2020/01/29/using-lvm-cache-for-storage-tiering/) and [the Red Hat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/lvm_cache_volume_creation) and found myself confused at the difference between cache volumes and cache pools. The man page shows a cache volume, which looks nice and simple so I'll use it.

```
# lvcreate -n home -l 100%FREE vgdata
# lvcreate -n cache -l 100%FREE vgxubuntu
# lvconvert --type cache --cachevol cache vgdata/home
cache single cache not found.
# lvconvert --type cache --cachevol vgxubuntu/cache vgdata/home
Please use a single volume group name ("vgxubuntu" or "vgdata")
```
Well, there's my answer. It must be in a single volume group.

Time to delete the cache lv and shrink the main volume group, then join the rest of my ssd PV to vgdata.

Nope, [can't do that either](https://superuser.com/questions/1007939/lvm-two-or-more-volume-groups-vgs-on-one-physical-volume-pv)

> A Volume Group (VG) comprises Physical Volumes (PV). If the PVs are in one VG, they can't then be placed in another Volume Group.

So maybe what I need to do is add everything to a single volume group.

```
# vgremove vgdata
# vgextend vgxubuntu /dev/sdb1 /dev/sdc1
# lvcreate -n home -l 100%FREE vgxubuntu /dev/sdb1 /dev/sdc1
# lvcreate -n cache -l 100%FREE vgxubuntu /dev/sda2
# lvconvert --type cache --cachevol cache vgxubuntu/home
Cache data blocks 380215296 and chunk size 128 exceed max chunks 1000000
Use smaller cache, larger --chunksize or increase max chunks setting
```

After this I ran a round of `lvdisplay`, `vgdisplay`, and `pvdisplay` to confirm that the home logical volume was created and existed on the HDDs but not my SSD. However the cache was unavailable! Looks like that warning was fatal. I'll try what it says and increase the chunk sie. The default (128) is unitless. Is that bytes? Oh, the man page explains. It's 128k. That's more reasonable but still pretty tiny.

The other concerning thing in the man page is the default write mode: 
"The default dm-cache mode is writethrough"
That's gonna be glacial! It will wait for the spinning disks on every write operation. I get that it makes
sense for data integrity, but it will be quite painful for my usage and I don't care if I lose some data when there's a power outage. The ext4 filesystem will survive.


```
# lvconvert --type cache --cachevol cache vgxubuntu/home --chunksize 512k --cachemode writeback
```
That worked! Let's put a partition on it and see what happens!

I created an ext4 partition on the drive and set the mount point to /home, then copied all the existing contents of /home there.

The moment of truth - I hit reboot and waited.

And it didn't work. On startup it took me to a login screen, and then failed to log me in.

It must not be mounted at the right point in the boot process. Bummer :(

I hit ctrl+alt+f1 to get a non-graphical login and then edited the fstab to remove the broken entry. Let's see what else we can do - maybe I should mount it to /mnt/bigstorage and then bind-mount in stuff like games.

Wait ... everything in it is owned by root. I just didn't move the home folders correctly.
I unmounted it, copied everything back *preserving permissions* with `cp -Rp /home/* /media/home/` and rebooted again.

This time it was healthier! The system logged in successfully and home has 1.3TiB free!

## Other tweaks

I use a model M keyboard, which is missing a windows key. Let's remap the caps lock key to do that instead. Easy enough, I set this command to run at startup.

```
setxkbmap -option caps:super
```

I was very pleasantly surprised to see the system came with nvidia drivers that not only worked, but performed well! I ran [WebGL Aquarium](https://webglsamples.org/aquarium/aquarium.html) as a quick and dirty GPU benchmark to see if the drivers were loaded and it ran identically to Windows.

The driver version was slightly out of date. But the newer one was available right in the package manager - I didn't even have to search the internet for it!

```
apt install nvidia-driver-510 nvidia-dkms-510
```

# Testing

Now we have two places to store files: 

* `/` always puts them on the SSD
* `/home/` will store them permanently on the HDD and cache what's in use on the SSD

I used gnome-disks builtin benchmark to compare my two logical volumes

<TODO image here>

* Pure SSD: 555.7 MB/s, 0.06 msec random access
* Mixed volume: 99.4MB/s, 20.58msec random access

This benchmark is deliberately using data that we couldn't cache, so it's showing the base performance of the disks. That's about what I expect.

Let's try a write benchmark:
`dd if=/dev/zero of=$path_on_disk`

* /home/owen/zeros: 173 MB/s
* /opt/diskbench/zeros: 271MB/s

htop shows both of these maxed out a single CPU core in kernel time. Not what I expected.

I'll post some more benchmarks once I think of a way to measure this.
DiRT Rally 2 load times might be a good example.