---
layout: post
title: Workflow of overcompensation
---

## The need for speed

I like to work fast.
No... wait... I like to work efficiently.
And this post will be about the technical side of that.

I'm not going to tell you about what I do to be a good programmer (write some code, find your mistakes, write some better code, goto 1), but I can tell you what to do to be an efficient programmed.
Granted, if you're still struggling with writing code, this post will probably not be too useful for you, as you're gonna be limited by the skill level, but if you feel like you can do more, but hit the efficiency ceiling, this *might* help you.

You just need to remember one, very important thing: don't use this post as a ready solution.
Rather, use it as an inspiration.
Your workflow is probably going to be very different from mine.
You're also probably working on something different from me.
Because of that, my workflow might not be the best for you, but you can always adjust and tweak parts of it to make it yours.

Don't be afraid to experiment with your own ideas as well.
This was not a thing I've created in an afternoon without trial and error.
What I'm about to describe is the result of finding inspiration, trying various things, rejecting failed ideas, and buying many, many, many, many keyboards...

## The software

This part will talk about the software side of things: various programs that I use that help me do the things that I do.
Note that this will not be completely in-line with what's available at the [software tab](/software.md), as that lists *cool* software, that's not always necessary for efficient work.
Sometimes it's just there because it's cool in a very specific case.
I'll also try to split this into various sections talking about specific kind of software to make this somewhat searchable if you're only interested in a specific kind of software.

### This is how you install Arch

So at some point in the past I wrote about running back to Windows because I've had issues with Linux.
Well, I'm happy to report that I no longer use Windows for anything other than occasional gaming.
I'm even ready to claim that I wouldn't be able to use Windows now for anything productivity-related.

So does that mean I'm back on Linux? Well, kind of.
The big surprise here is that I've since moved mostly to macOS, but I'll talk about that later.
For now, let's just stop on Linux for a minute, because yes, I still use it on my work laptop.
The difference, however, is that I used to use Ubuntu everywhere.
However, I would sometimes run into weird issues with it, like i3 being slightly broken sometimes while being installed alongside Gnome, or snap being extremely difficult to work with (thanks, Firefox), or either having outdated packages or having to do major upgrades twice a year that do sometimes break things, so this time I've decided to try something different.
Everybody, please welcome, Fedora Linux... on workstation only (all my servers still happily run on Ubuntu).

Now, I'm not gonna claim that Fedora is definitely always better for desktop than Ubuntu, but in my experience, it's somewhat easier to work with.
I've found that more packages are available for Fedora than Ubuntu, and more of them are actually being kept up-to-date.
Fedora also comes with a version with i3 installed by default, which just makes it infinitely better and easier to get going than whatever Ubuntu flavour you choose.
I still occasionally run into issues (did you know that your kernel can sometimes fail to hibernate because you've at some point connected a USB-C device into your laptop? it doesn't even have to be connected anymore), but none of them have ever made me think of going back to Ubuntu or, even worse, Windows.
Maybe it's my laptop that's fortunate enough to be compatible with everything, or maybe it's just me that's grown accustomed to things being always slightly wonky on Linux, but I can with certainty say that I haven't had any major issues while running Fedora (and now that I've said that, my laptop will explode tomorrow morning).

So now let's address the apple in the room... why macOS?
I've once described Macs to a friend as "they're like Linux, it's just that they actually work".
Given the previous paragraph, you might think that it's somewhat disingenuous to say so, but I have a good explanation for that.
Everything good that you know about Linux is (usually) true for macOS as well: most productivity/programming/sysadmin software that's out there for Linux is usually available for mac as well.
What's more, this software is somewhat commonly written for Mac and then made to work on Linux as a side effect, with some caveats sometimes.
The difference is that you need to have technical knowledge to be able to set some things up on Linux.
If you don't know about swapfile or swap partition, secure boot, grub, and probably some other things, you won't be able to enable sleep/hibernation on Linux - on Mac you just close the lid.
Customising your Linux usually requires editing a text file, which sometimes can destroy your system if you're not careful, while Macs usually give you some form of a GUI, even if limited.
And good luck setting up screen sharing when using Webex in Firefox in Fedora...
So, you see, it's not *necessary* to use macOS to be productive, but if you want to be productive in things other than Linux, you might want to use something that's not Linux... only attempt Linux if you're already proficient in it and are willing to spend considerable amount of time prepping everything to be *just* right.

Granted, once you do get Linux *just* right, it's gonna be a great experience, but things do still sometimes require maintenance.
The good thing is... it's extremely easy to switch from Linux to macOS.
However, both systems allow for a significant amount of customisation and have a wide array of productivity and development software available for them, so if you're not willing/able to spend a significant amount of money for a Mac, then Linux is a good alternative. Not "good enough", no. "Good".

Side note here, Apple Silicon is absolutely amazing and might be one of the best innovations in current PC/laptop market.
Trying M2 CPU for the first time was genuinely mind-blowing.

Oh, and Windows?
I completely gave up on Windows.
It's just not made for productivity.
And don't even ask me about the "Copilot button".

### IDE

This part is going to be completely useless for you if you don't do any programming, but oh well...
Nowadays, I use JetBrains IDEs for pretty much all of my programming work.
I really like how they look and feel, and they're very solid pieces of software that do what they're supposed to.
They also have an excellent implementation of vim emulation, which is an absolute *must-have* for me right now.
They also come with a handy settings-sync feature, which makes my life easier as I tend to juggle between various computers with them.

They are also very helpful when you're like me and like to write in many various programming languages, as there's an IDE for most of the popular languages: Java, go, C#, Python, JavaScript, and so on.
All of them behave in a similar way, so switching from one to the other doesn't take almost any learning.

"But VS Code is free and can do most things too" I hear you say.
To that I say: before getting started with Rider, I had to use VS Code to edit a large monolithic asp.net application.
And let me tell you, it was such an incredibly bad experience, that I'm not touching VS Code for anything ever again.
Sure, small projects or projects written in languages that are not as overcomplicated as C# have a chance of working better (I've actually heard good things about how VS Code works with go and JS).
However, the fact VS Code doesn't do anything by itself and relies on plugins to call sdk tools to actually do the work and then fumble the response back to the UI makes this horribly not scalable for large projects.
If you're only ever working on small applications or only ever write small-ish microservices, then you *might* be fine, but if you're ever going to write some bigger or, god forbid, something with front-end that's not plain html/js, then keep a safe distance.

As a side-note, this post is written in Writerside, which is another JetBrains... documentation IDE?
It's basically their markdown editor they ship with every other IDE but separate...
Sounds somewhat weird but works quite well, so I'm not gonna complain!

### Internet Explorer

You might not think so, but for me, a web browser is a critical part of equipment.
That's because, as a web developer, I spend a lot of time looking at a browser window, reading logs, analysing network requests, and so on.
There's also the fact that nowadays you do almost everything using a web browser, including anything related with Git{Hub|Lab}.

To make things a bit more complicated, I use something else at home and at work.
That's because at home I just use Safari, as it's an amazing browser, has great sync with mobile Safari, and just works well in general.
There are occasional issues with video loading on mobile, but that's something I can work with.
The dev tools are also good.
The only downside is a relative lack of extensions: react dev tools are completely missing, vimari is somewhat lacking compared to vimium (more on that later), and so on.

At work, meanwhile, I use Firefox... and Chrome.
The thing is, Firefox is somewhat lacking in... most regards to Chrome.
However, for everyday *normal* use it's totally fine (as long as you don't try to use Webex on Fedora for some inexplicable reason), which is what I use it for.
For development, however, I've decided to stick with Chrome, as the dev tools just feel better in Chrome.

Now, I've mentioned vimium/vimari.
This is an extension that lets you... *pauses dramatically*... use vim keybinds in a browser.
That's a very specific thing, but for me, who spends most time with vim keybinds, it's such a useful tool that I genuinely feel lost when I cannot use it.
It makes it much easier to use the browser without a mouse, and doesn't break your flow as much when you need to switch to a browser.

### 137 GB of music

...is how much I have on my phone.
This number might seem large... well, because it is.
I have a very specific taste in music, particularly the one I listen to when I work (which is 99% of the time I listen to music), and it doesn't fit very well with streaming services like Spotify.
For that reason, I've had to do something different to most people.

I mostly get my music from Bandcamp, though sometimes I just download files from Mixcloud or Soundcloud.
Wherever I get the files from, I need to somehow get them to my phone and then somehow play them.

When I was still using Android, I'd copy files over using something like smb, and then use Poweramp to play them.
If you're on Android, there's no better music player than Poweramp, as it can do pretty much everything you'd ever need and all alternatives are somewhat lacking.

There's just one issue... it's only for Android... there's no iOS version.
So, with that in mind, I needed to do something different when I've switched to Apple.
I've experimented a bit with everything and eventually settled on just using Apple Music.
However, working with local files in Apple Music can be somewhat problematic, so I'm gonna summarise my workflow:
1. I define all my playlists on my Mac and keep all local files there. All playlists contain only my local files. Mixing local files with music bought with iTunes or available with Apple Music subscription *will* break. If you buy songs on iTunes, make sure to download them locally and *then add them to the playlist again* by drag-and-drop.
2. When all playlists are set as they should be, I then use Finder to sync my iPhone with my mac. You need to connect your iPhone to your mac with a cable for it to get recognised, but you can then check for the iPhone to show up using wifi. This process is a bit finicky and sometimes break for no reason. If that happens, a reboot and re-pairing of the device usually helps... eventually. What you need to remember is that the initial sync will take a while.
3. After syncing, all your music is available on your iPhone. To change anything in the playlists though, you must change them on your Mac and then sync again (this will be much quicker, as only new songs will be transferred).

At this point, some of you might start shouting "BUT DOESN'T APPLE MUSIC OFFER A CLOUD SYNC FEATURE IF YOU SUBSCRIBE??!!??!?!".
And, well, yes, it does offer such a feature, but I've managed to very quickly find out the limits of it.
You cannot upload files longer than 2 hours and larger than 200 MB (I think, I don't actually remember the exact value and cannot be bother to look it up now).
This is a problem for me as I have a lot of mixes that last over 2 hours, or take up more space due to high quality, which is why I cannot use that feature.

There's one more hurdle to go through, as some file formats will refuse to sync at all, even locally.
Before, I used to have most of my library in `flac` format, as it's high quality and was supported by Poweramp.
The trouble is, it's not supported by Apple Music at all.
The fix was quite simple (in theory), as I just needed to convert all my library to mp3 or something else supported by Apple.
Unsurprisingly, converting over 100 GB of music took a good minute to complete, but all new songs I get I just download in `mp3`.

And no, Apple Music doesn't allow you to just play files you have stored locally on your iPhone, so copying files over and selecting from storage will not work.
There are alternatives that allow you to do that, but after trying some of them, I've reached a conclusion that all music players for mobile are cursed in some way.
You might give some of them a go, but none of them are really designed for libraries of the size I have.

### Honorable mentions

In this section I'm only gonna say that I have a sometimes-updated list of software I use [here](/software.md), so if you're wondering what I recommend for something, just look there.
The fact is, when you work with computers as much as I do, you at some point stop thinking about every single piece of software you use, as there's just too much of it.
If you have a question about something that I *might* know, but is not listed anywhere here or on the list mentioned before, feel free to create a post in the comments section (there's a link to it at the top of the page).

## The hardware
