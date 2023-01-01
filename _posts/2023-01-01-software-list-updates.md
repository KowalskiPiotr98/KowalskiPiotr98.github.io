---
layout: post
title: Software list updates
---

## HAPPY NEW YEAR
It's the First of January 2023 when this blog post is published (I sure hope so...) and, as many people do, I've decided to do a look-back at the past year.
Specifically, at least in terms of this blog, in terms of the software I use for daily tasks and how that list has changed over time.

Just in case you don't know, there's a constantly (not) updated list of software that I use.
It's in the top navigation bar of this page!

## The major changes
Since the last update of this list I've made one major change to how I work with computers.
That is, I've switched from Linux to Windows 11.

Now, before all you Linux lovers jump at me to serve justice, let me explain myself.
The trouble with Linux is that if anything breaks (and thing will break) you need to fix it yourself.
While that is also true for Windows, Linux tends to break more often... sometimes.

I used to use Linux on 2 laptops: a personal laptop and a work laptop I was given by the company I work for.
On the company one, Linux worked great, and I had to spend almost no time on maintenance.
Some things would break sometimes, but it was no more annoying than occasional technical issues that happen on Windows.
The trouble was, however, my personal laptop.
Things just would not work.
Hibernation would work 20% of the time, the rest - it would fail to boot.
Whenever I had an external monitor plugged in, I'd need to unplug it and plug it back in after locking my screen - otherwise it just wouldn't work anymore.
Battery was being used at an alarming rate.

At some point I got too tired to deal with it and decided to switch to Windows.
However, since I don't like having different setups for work and personal programming, I've also switched my work laptop back to Windows.
Since then, I've been working on Windows 11, and I'm still missing some Linux exclusive features and software.
That being said, Windows is still quite capable programming machine, especially if you include the fact that WSL has systemd now, essentially gifting you a fairly well integrated Linux VM.

## Chrome is dead! Long live Chrome!
A notable change is that I've replaced Chrome with Chrome.
Or, to be more specific, Vivaldi was replaced with actual Chrome.

I've been on a grand journey to find the perfect browser for some time now.
I've been through Firefox, Edge (the new one), Brave, Vivaldi, and now back to Chrome.
And on that journey I've learned that every single browser out there is bad in some way.

Vivaldi is probably the nicest browser in terms of productivity - you can remap keybindings, it has integrated email and calendar, you can split tabs, take screenshots, and many other things.
The trouble is, I've never used any of that.
Instead, I was constantly worried by the long delay between new Chromium version and Vivaldi update using it, plus you can't close dev tools with F12 (you don't know how annoying that is for a web developer).
Plus, Bitwarden, which I've used as a password manager at the time, doesn't work too well with Vivaldi, for some reason, which was a pain to deal with.

Chrome is fine.
It works.
It also probably tries to steal all my personal data, but what doesn't at this point.

## Anything but LastPass
You might've noticed that when writing about Chrome I've mentioned Bitwarden.
As it so happens, I'm no longer using this password manager.

Before you run screaming to your Bitwarden vault to move all your password: IT'S TOTALLY SAFE.
This move was not done due to security concerns, and I'd still recommend Bitwarden to anyone trying to get a good free (or at least cheap) password manager.
It's not [LastPass](https://palant.info/2022/12/23/lastpass-has-been-breached-what-now/), after all.

What prompted me to change is that I've seen many people recommending either Bitwarden or 1Password.
And from looking online I've gathered that 1Password is as secure as Bitwarden, but has better UX.
Since Bitwarden can be a bit clunky sometimes I've decided to give 1Password a go and loved it immediately, so I've decided to switch.
If price is your deciding factor, however, then you should stick with Bitwarden, as 1Password does not have a free plan and the cheapest one is still more expensive than paid Bitwarden.

## Out with the old
I've not removed software I've been using on Linux, even though I'm no longer using it.
I have, however, removed some software that I've no more need for as I've found better alternatives elsewhere, or I'm just not finding it useful anymore.

Samsung Notes was only useful for me when I was taking notes on my tablet with my stylus.
That no longer happens as I pretty much write everything in Obsidian, so that got purged.

Play Books is still probably the best mobile app for e-books, if you need synchronisation.
What changed is that I've bought a Kindle in the meantime, so I just don't need that on my phone.

Rss2email is still a great tool when you want to send rss entries using your email.
However, since I've given up on self-hosting email servers, that was no longer viable for me and sometimes generated too much spam for SendGrid to handle for free.
All of that is now handled using IFTTT.

## In with the new
A lot of new entries got added as well.
I will group some of them by the reason they've appeared

### The Windows switch
When moving back to Windows I needed some apps to replicate Linux functionality I was now missing.
This is where PowerToys comes in, which is a program I'd recommend to anybody using Windows.
I've also installed ShareX to take screenshots.

### Self-hosting no more
In the meantime I've also stopped self-hosting things publicly.
That meant that I needed to move some services elsewhere.
This is where IFTTT and SendGrid come in, as they now handle email sent by my local machines and certain automation tasks previously handled by my servers (or Make).

The local self-hosting is still going fine, which is where changedetection.io and Ganymede are currently being hosted.

### Fish out of the water
When switching to Windows one of a deal-breakers was the ability to use fish as shell.
Let's be honest, PowerShell is terrible and nobody should be punished by using it, so I needed a way to use fish (or bash at the very least).
That's where Cygwin comes in, as it's capable of running fish and many other programs.

But you need to run fish in something.
I've been using Windows Terminal for some time and it's fine, but lacks certain features I've grown used to over time.
That's why I've decided to try something totally new and have gone for Termius.
It's more of an SSH management thing, where you can setup hosts, keychains, and other things and easily connect with ssh or sftp, but it also allows local terminal usage.
It has its own issues (primarily that you need to pay to unlock certain features), but I think it's overall a more capable app than Windows Terminal.

## 2022++
So that's that in terms of software changes.
I do hope that at some point this year some I'll be able to spend more time writing this blog, as right now I'm busy with other things.

Happy new year, everyone!