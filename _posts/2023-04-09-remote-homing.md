---
layout: post
title: Remote homing
---

## ~~Work~~ Home from home
Over the past couple of years, more and more people got to experience working from home. Some loved it, others hated it, but whatever you think, it was sometimes really convenient and made commute times shorter.

During that, a lot of tech was necessary to make it all happen: VPNs, remote access to many things, and such. And that got me thinking: could I do the same? See, I am a tech guy (if that's a surprise to you, I suggest looking at this blog again, this time more carefully) and I have a lot of tech. Usually, when going to my family home, I'd take with me my laptop, which is a bit heavy. But not this time. This time, I've decided to only take my tablet (and my Switch, but that doesn't fit this story) without missing anything.

## The target
The plan is to try to remotely access my PS5 and my PC from my tablet to be able to reasonably game on it. Also, as a bonus, I will try to somehow set up a development environment, without actually needing any other hardware (this ultimately didn't make it into this post, as I've just used GitHub Codespaces, which *just works*).

Here I feel like I need to mention some reasonable assumptions about this plan. I know that it's not possible to exactly replicate gaming on my PC locally with my tablet in another city. Small delays are inevitable, and the screen is *somewhat* smaller, so I'm not gonna be trying to play things like [osu!](https://osu.ppy.sh) that require perfect timing, or playing full-resolution that would be microscopic on the screen of my tablet.

Gaming will also not be the only thing I'll be trying to achieve. From my other blog posts, you might know that I self-host some things in my home network. So, as you might expect, I will be trying to make it possible to access them remotely.

Let's go!

## The easy
Let's get the easy thing out of the way first, as it's not that interesting: PS5. It supports remote play natively, doesn't need to be on the same network, works perfectly well on the tablet, and the image quality and delay are *fine*.

I don't think there's anything else to say here. It *just works*. Again, It probably won't work with timing-critical games, like rhythm games, but it's gonna be okay for a lot of other titles.

If you have your PS5 controller, you can also connect it via Bluetooth to your tablet and use that. Other pads, like Nintendo Pro Controller, will not work though.

## NAT, or how the Internet works
Before we come to the next part, you need to know the basics of how the Internet works. Otherwise, you might think that everything I've been doing was kind of unnecessary. So, let's get into it.

Every device connected to the Internet needs to have a unique address, also known as an IP address. It usually looks something like `66.254.114.41` (for the sake of simplicity I'm going to ignore the existence of IPv6, just like most other people do). Using IPs you can access any computer connected to the Internet...

...as long as it's the last century. You see, every part of this address is a number between 0 and 255, so in total, there are "only" around 900,000,000 available addresses. That might seem like a lot but if you consider that your phone, computer, router, smartwatch, game console, and whatever else connected to the Internet needs a separate IP, that number stops being so large. Add to that all IT companies that have an ungodly amount of servers and devices, and you'll be able to see that this number is not enough. Plus, not all addresses can be assigned to a device, as there are (lots of) special addresses used for other things. Plus, sometimes an address is assigned to something and never used, which makes things just ever so slightly worse.

But... the Internet works, right? How is that possible, then, if we don't have enough addresses? Once again ignoring the existence of IPv6, the simplest solution was to group devices into networks. There's no practical reason for every device in your home network to have its own public IP address. Nobody will ever want to initiate a connection to your phone, so it doesn't necessarily need a unique address on the Internet scale. It just needs some device that has one to forward the message and then listen for an answer.

That's essentially what the router in your home does. It does have a unique IP (that's not always true, but let's pretend otherwise, as it's not important right now) and is capable of working as a "proxy" (not in a "web proxy" sense but, again, let's ignore that detail) between the web server you're trying to talk to and your phone. Devices inside of this network will instead have something called a "private IP address", usually something like `10.0.0.25` or `192.168.0.25`. This IP is only valid in your network, and other networks can (and will) have devices with the same private IPs. The only caveat is that there's no way to directly communicate between devices in those two networks without extra settings or services (one of the reasons is that you can't have multiple devices with the same address in one network).

## All your NATs are belong to us
What would you need to do, then, to talk to a computer inside your home network from outside? The simplest (and least secure) solution is to set up "port forwarding" on your router. It works something like this: when a connection is initiated to the router on a pre-defined port, it forwards this request to another device inside the network. As this is all happening on a lower network layer, this is completely transparent to the client device. This way you essentially publish one port of your device on the Internet.

This is also a terrible idea as there's no extra security in place. If this service you're forwarding the traffic to is vulnerable and can be used as a network proxy, an attacker could essentially gain access to your home network, which is, in general, not good.
How to do it better, then?

Introducing VPNs. And no, I'm not talking about the services falsely promising you privacy and other such things. VPNs are, by design, meant to connect you as if you were in another network. And if you think that it sounds exactly like what we're trying to do here, then you're right.

Here you have two options: you can either set up the VPN on your router or in some other way that requires you to host it yourself or use a service for that. Since I can't access my router from the Internet (thanks, ISP), I had to go with the second option.

This is where [Tailscale](https://tailscale.com/) comes in. By default, they just create a virtual network for your devices (as in: you can see other devices connected to your Tailscale account from wherever you are), but you can also configure something called an "exit node". If you do that, all traffic from your device will be tunnelled through this host. And if that host is in your home network... you've effectively gotten yourself a VPN into your home network.

With that, you can access any device in your home network, as long as you're connected to Tailscale and configure your devices to use that exit node, without needing to install Tailscale on all of them. It's even simpler if you just want to connect to a single device, as you don't need to use the exit node thing, but instead, you can just connect to the "Tailscale IP" of your device. To do that you just need to install the Tailscale app on the device you want to connect to and the one you'll be connecting from and you're done.

The only caveat is that you can't boot your device with Tailscale, so you either need to do wake-on-lan (will talk about that later) from the exit node (which by itself needs to be running all the time) or just leave the device running. It's not a problem for me as I have servers running 24/7 anyway, but you might be unwilling to just leave a computer idling for a long time just because you might want to connect to it at some point. It's also less of an issue if you're leaving for just a couple of hours and know you'll need that PC anyway, but as I was leaving for a couple of days and would most likely not need it, I went with WoL.

## Ransomware Deployment Protocol
Assuming you have your Windows PC running and can connect to it, there are two ways to remotely control it: RDP and VNC. The first one (which actually stands for Remote Desktop Protocol) is the "native" Windows way of doing remote control. It's a built-in Windows feature that you can just enable in the settings... assuming you have Windows Pro - it's not there on Windows Home edition. Once you've done that, just download the official Microsoft app for your mobile device, or launch the client app bundled with Windows, enter the local (or Tailscale) IP of your device, input your login and password, and discover that using your Microsoft Account as a login for Windows breaks this completely.

There's a solution to that: launch your PowerShell and do this:
```powershell
runas /user:<your MS account email> pwd
```
You will be prompted for your Microsoft Account password. What this command does is it "saves" your Microsoft Account password to your PC, because most likely you've used your Microsoft Authenticator or something like that to log in for the first time and then have been asked to set up Windows Hello. Anyway, doing all that will let you log in to RDP using your MS account credentials. Once you've done that, you'll be able to use your PC remotely.

Another option is VNC. It works somewhat differently from RDP, as it effectively shows you exactly what you'd see if you were sat in front of your PC. The differences in day-to-day use are minute, but in some cases, it can be helpful. The downside is that there's no "official" support for that protocol on Windows - you need to install third-party server and client software. Anyway, I've decided to use RDP in this case and I won't be going into much detail about VNC. Just bear in mind that if you don't have the Pro edition of Windows, VNC is your only option.

I should also mention that, if you want to game on your PC using Steam, you can use Steam Link as well. It's probably a bit more optimised for gaming, but ultimately is just another form of remote desktop. Oh, and it also requires you to physically confirm a prompt on your desktop if you don't unlock your PC beforehand. You can get around that using VNC.

## Wakey-wakey, Windows PC
Let's recap how my infrastructure is designed: I have an always-on server that I'm using as an exit-node (it's a Raspberry Pi computer) and a PC that I might want to RDP into while away. Now, I *could* just have this PC always on, but that's a terrible waste of energy, so I'm not going to do that. Instead, I will use something called "wake-on-lan".

The basic idea is this: instead of your entire PC being powered on all the time, it's only your networking card that's active. It's constantly listening for a special message called a "magic packet", and when it sees that message addressed to its MAC address, it sends a message to the motherboard to start the PC. That allows you to start the computer without actually touching the power button.

There's a slight issue with that, though. Since this packet needs to be sent to a MAC address, and not an IP address, it can't traverse through VPN. So, if you use an exit-node, you will need to first access some host physically on the network with ssh, and then send this message out from that host, so it originates in the actual home network. Sending this message using only Tailscale communication is not possible.

Sending this message is very easy, though. There's a Linux program called `wakeonlan` and you only need to know the mac address of the PC you're trying to wake. Just make sure that you know it before your leave, though, unless you're able to check in on your router (for example when you've set up a static DHCP lease for this MAC).

After you've sent that packet, your PC should boot normally, as if you were to press the power button. Wait a couple of seconds and then try connecting with RDP. If everything went right, you should now have remote access to your PC. You can also safely shut it down and boot it up the same way again later.

## The results
I've talked a lot about how to do it, but what's the actual result? Well, it will depend a lot on your Internet connection speed and quality, but from my experience: gaming is entirely possible. Now, you probably don't want to play timing-sensitive games like that, but anything else is totally fine. You need to account for the smaller screen (obviously fewer details will be visible), but apart from that, I see no issue.

What about everything else? If you have a desktop program that you want to use on your tablet remotely, can you do that? Yes, totally. Again, if it's graphic design work or something of a similar nature, the image quality might not be the same, but if you do programming, text editing, technical design, or what have you, it's totally doable.

Should you do it, though? I mean, isn't it easier to just buy a laptop? Well, maybe. Depending on your needs, a tablet can be much better. For starters, the battery life will be much, *much* better, as you don't actually do anything power-intensive on it. It will also be much lighter, as powerful laptops tend to be quite heavy. However, if you mostly use your computer in the same place, and only leave occasionally and for extended periods, it might not be worth the extra cost (you need to buy a tablet, after all) and effort required. Carrying a heavy laptop every day might not be the best idea, but doing so twice a month... I'd say you might need to think about that one.

However, if you *really* need a powerful machine, and feel like laptops are not enough for you, but at the same time you want to be able to take it with you when you leave your home... that might be a viable solution. When talking about high-end devices, laptops will always be less capable than desktops, and more expensive, too, so even including the price of a tablet, the combo of desktop + tablet + raspberry-pi-like computer might turn out slightly cheaper in the end.

You still need to remember, though, that many factors might make this solution not a viable one for your use case. So, as with every project like that, before you start deploying everything I've described, just stop and analyse whether this will work for you.
