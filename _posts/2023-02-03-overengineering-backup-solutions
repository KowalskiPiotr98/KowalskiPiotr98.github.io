---
layout: post
title: (Over)Engineering backup solutions
---

## Preparing for what ifs
As you might've noticed from the contents of this blog, my background is in IT.
I'm mostly doing programming work, but I've also been hacking away at hosting things for a while (as evident by previous articles).

As every sysadmin ever ~~will~~ should tell you, backups are of paramount importance, and sometimes it's quite hard to get it right.
So then, in this post I'll tell you how I handle my personal backups.
It might not be the best way, it might not work for you, but I hope you'll be able to at least see what drove me to do things the way I did.

## When I was... a young boy...
I've been making backups for a long time.
It's probably more of me being a hoarder than being careful, but even as a kid I'd have my own personal CD-RW (which would later be upgraded to DWD-RW), on which I'd store everything that I considered important back in the day.
It might not be "important" now (you know, it was mostly game saves and things like that), but it did help me a couple of times in the past.
Well... not the one time when I've accidentally copied shortcuts onto the dvd instead of actual data, but that's beside the point.

The thing is, I've always been doing backup-like things, and it just sort of stuck.
It also helped that my dad has been working in IT for a long time and has also been doing periodic backups of his own, though to a lesser extent than I'm doing now.
At some point he bought an external hard drive (remember when they needed to be plugged into the wall, because USB didn't provide enough power?) and backups grew in size gradually, as space became more accessible and file sizes grew larger.

## Walking my own path
At some point I've decided that I needed my own backup solution, that didn't require me to use an outdated external HDD (I believe it's still alive and kicking, though).
At first I just bought a more modern external drive (no additional power required), which I still own, that I used to do backups.
However, after some time I've wanted to move to something more convenient.
That was also the time when cloud backup solutions were becoming more of a consumer thing, and services like Dropbox or Google Drive were getting more and more popular.

At that point, I've decided to use those instead, especially that I've already had a smartphone with a reliable mobile Internet.
And I'd been doing so for quite a while.
No local backups, just cloud.

Is that a good solution?
Well... maybe?
To be honest, the probability of losing data stored in Dropbox, OneDrive, or whatever else popular is incredibly low, so if you store all of that data on your PC as well, it's probably fine.
I'd still recommend to have a local, offline backup of the most critical data that you definitely don't want to lose (talking more of legal issues, as losing some documents might be problematic or even costly).
As for all your holiday photos, or the 20GB folder with pictures and movies of your dog... I know it's precious, but you'll probably be fine if it's gone.

## Too much free time
At some point, though, I've decided that maybe I should go one step further.
This was around the same time I was trying to set up some sort of a home server for self-hosting local services, so I've decided to combine those ideas into one.
What came out of it was a [Synology NAS](https://www.synology.com/en-us/products/DS220+), as it's primarily a backup solution but can also run Docker containers, so some light self-hosting would be possible.

I've bought 2x2TB HDD, set it up to use RAID 1, created some volumes with Btrfs, and was ready to roll.
But I didn't really want to depend completely on the NAS, so the next step was to consider off-site backup.
And it went like this:
- I have 3 partitions on the nas, let's just call them 1, 2, and 3 for now,
- partition 1 is synced *with* cloud (Dropbox at the time of writing, used to be OneDrive before that), so that everything uploaded to nas is immediately uploaded to cloud, and vice-versa: everything uploaded to cloud is downloaded to nas (this is useful I'm not at home and need to upload something to my backup),
- partition 2 is entirely local-only - I'm mostly using it to transfer files between computers or store large files there that I don't mind losing that much; I'm also using it to backup some external services, like my GitHub repositories (this blog included), as it's already in the cloud (on GH), and on my PC anyway, so it's not critical to backup to cloud again,
- partition 3 is encrypted and I use it to store more sensitive files; it's also synced *to* cloud: once a day it's uploaded to cloud (Dropbox again) in an encrypted format, that cannot be read (and modified) from the cloud (to avoid accidental damage, the changes to the cloud version are ignored).

You might've also noticed that I'm using Btrfs, so journaling and filesystem snapshots are a thing that I can do quite easily, and they don't take that much space.
Thanks to that, I'm also able to restore old versions of files (within reasonable time period) if I accidentally delete or modify them.

Volumes 1 and 2 are also shared with SMB so I can just mount them as drives in my OS and have all files available.
Since the NAS is in the same network as me, file download/upload is very fast, and I'm not slowed down by network connections to Dropbox or other cloud services.
This is quite handy, as large file uploads take seconds over local network, and then just upload themselves to cloud from the NAS without me needing to keep my PC running.

## Paranoia level: high
What I've described above is probably enough for everyone (assuming just standard personal data, I'm not talking corporate policy thing here).
But that doesn't mean I cannot go one step beyond that.

See, for the entire time one thing was constantly on my mind: all of this backup is constantly online - if my PC gets infected with malware, those mounted drives will probably get encrypted too (hence why partition 3 is not mounted, ever), and since most of the storage is synced with cloud, those files might be encrypted too.

It's not *that* big of a deal, as the journaling mentioned before could help, as well as Dropbox keeping file history for some time (just for situations like this one), but I wanted to have a version of this backup that would survive even if everything got encrypted/deleted/whatever.
So, to solve that problem, I've bought a small external SSD.
Well, I say small, because it's much smaller than even my phone, but it can hold 2TB of data, which is exactly as much as my NAS does.

With that, I once per week plug it into the nas, clone *everything* (it takes minutes thanks to the incremental nature of such backup), unplug it, place it in a safe spot, and am done with it for another week (also once per month I do a backup validation to make sure it's definitely capable of restoring itself).
This backup is additionally encrypted (all volumes), for safety.
And since it's mostly just laying around unplugged, even if for some reason all of my data would get purged from Dropbox and nas, this one would survive.

## Do you *really* need all of that?
Probably not, no.
Backups are one of those weird things that you need to do but never want to use, as that means something somewhere went wrong.
Will I even use this external ssd backup? 99.9999% no.
Will I be glad I had it if this 0.0001% actually happens? 100% yes.

So, what I my recommendations for you?
Err... that depends on how technically inclined you are.
If the phrase "server hosting" makes you scared, then you should probably go with the simple option of having a cloud backup (Dropbox, OneDrive, whatever), with an additional local backup for the most critical data.
However, if you have some free time and are willing to experiment... then what I have has been really fun to setup and I'd recommend you give it a go.
Plus, having NAS running gives you a local server you can use to host things, like Pi-hole or any other service capable of running in Docker (the CPU and RAM are quite limited on NAS, so do keep that in mind).

And the best thing of all this is that it doesn't really require any constant maintenance.
All relevant notifications can just be sent your way using an email, backups are made using a scheduler so you don't have to worry about that yourself, and the nas mostly just stands there happily working.
Once in a while you'll need to run a system update, but it doesn't take that much time.
It's definitely easier than trying to constantly copy your data to an external drive.

And as a final thought I'll just say this: **RAID BY ITSELF IS NOT A VALID BACKUP SOLUTION AND WHOEVER SAID THAT SHOULD BURN IN HELL FOREVER**.
