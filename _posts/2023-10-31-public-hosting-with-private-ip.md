---
layout: post
title: Public hosting with private IP
---

## A very "me" problem

Let me get started by saying that you might never encounter this problem.
It only applies when you, at the same time
1. **need on a very deep level** to host a thing from home, from your own hardware, with everything (good or bad) that this entails,
2. do not get a publicly routable IP address assigned by your ISP.

If any of the above is not true, then there are other ways around it, but otherwise you'll need to somehow get traffic to your device, even though no public IP exists for it.

Before you ask, no, I had no good reason for doing this (which probably applies to everything I'll ever write about on this blog).

## Initial problem

Before I start with the deep dive, let me establish some form of network layout, so that you have something to work with:
- the server actually hosting the service I want to publish, which is in my home, physically only connected to my internal network with no public IP at all, blocked by every possible firewall regardless of whether it's even routable (this will be called the "server" later on),
- that is then connected to a local WRT router which can be safely ignored for the purpose of this post,
- that is then connected to an ISP-provided edge router (this will be called the "router" later on),
- then the ISP does things,
- then Internet happens.

The router itself doesn't have a public IP - it's entirely within a local ISP subnet, and most likely shares the public IP with which I'm seen from the Internet with other routers, most likely other residents of the same block of flats or the neighbourhood.
I could probably pay the ISP to assign a routable IP to my router, but why do something so simple when you can spend a lot of time achieving the same thing in a much more convoluted way, right?

As a side-note, when trying to somehow route myself to this router from outside, I've found something that I *believe* to be a unique, public IPv6 in the settings of this router.
Unfortunately, I haven't managed to connect to it either, so it's probably just being blocked at the ISP level.

## Bad solution - VPN

The obvious solution that might come to mind when you hear "access from outside to internal-only device" is a VPN (the *real* kind, not the *nordvpn* kind).
And, sure, that would allow anyone connected to the VPN to access that server, but that's kind of not the issue here.
This server was intended to be public, and creating a VPN would require users to first log into that, which is not a good solution.
Furthermore, this just moves the problem a bit... because how do you make the VPN access point public?

Okay, so how about a slight modification... you get another device with a public and routable IP address, establish a VPN between that device and your home network, and then just forward the traffic from that device to the server using a VPN IP.
This, believe it or not, is actually a working solution.
It's not without its drawbacks, though, by far the worst one of which is complexity.

I don't know if you've ever tried to create a self-recovering VPN network (because this server must automatically reconnect if something were to happen to the connection, and then also be able to get the same vpn IP as before, for the forwarding to work properly), but let me tell you: it's a bit of a nightmare.
It's also not the most efficient thing, as you'll either send a lot of unnecessary traffic between those devices or spend too much time trying to get split tunnelling working as intended.

That is all to say: only consider this option if you've already got a VPN network of some kind and don't mind potentially compromising the security of it by potentially allowing external traffic to enter it (which is a whole another issue which I'm definitely not going to explore too deep in this post).
This leads us, however, in a somewhat better direction: use something that has a public IP to route traffic to the thing that doesn't.

## Working solution VPS + reverse SSH tunnel

(Is it just me, or does "reverse SSH tunnel" always sounds like some sort of deliberately overcomplicated magical term, somewhat similar to "Gottfried's Omni-opening Grimoire" or other things ((bonus point if you get that reference))… anyway, back on topic).

This solution kind-of builds upon the previous one.
We keep the VPS with a public IP, but instead of setting up VPN network, we use reverse SSH tunnels.

But wait, what is a "reverse SSH tunnel"?
Well, it's just like a normal SSH tunnel, but in reverse... okay that probably didn't help.
SSH tunnels work by tunnelling all traffic from the client through the server.
Imagine a scenario in which some service is only available from a specific server in the network (think of a case when it's only available on `localhost` on this server).
You can use SSH tunnels to get access to that service directly on your local box.
An example would be: you connect to `localhost:9876` on your local box, which is then sent through SSH to a server, that forwards that traffic to `localhost:3344` (this "localhost" is now on the server, not your box), essentially giving you direct access.
This is extremely useful in some situations, for example when trying to access some database with more advanced tools than just terminal, that's normally only accessible from some specific network segment.

So that's SSH tunnel. How does it work in reverse, then?
If SSH tunnels channel traffic from your box to a remote box, reverse of that would be that traffic from the remote box ends up on your box.
So, with that in mind, we can now channel all traffic getting to `<public-ip>:<some port>` to any computer capable of connecting to that `<public-ip>` with ssh.
And since that ip is public, any computer (or at least any you want, please configure your firewall correctly) can access it using SSH, regardless of whether it has public ip or not.
This is thanks to the fact that clients initiating traffic don't need unique public IPs thanks to the magic (and horror) of NAT (which is not a story for this post).

This is nice and all, but 2 issues still need mentioning: reliability and speed.
There is a fundamental problem with it: ssh connections don't automatically restart when broken, so when, for whatever reason, your connection gets dropped, it will not restart.
This is easy to solve, however, as `autossh` exists.
This is a small piece of software that automatically restarts ssh connections if they get broken.
As a bonus you can set it up as a systemd service, making it start automatically with system and restart should it ever crash.

The other issue does not have as pleasant of an answer as this one.
The speed is not going to be great: you're essentially making any TCP connection go through another layer of TCP.
What's even worse, though, is that you do the same to UDP (not)connections, making the most brilliant of protocols: UDP-over-TCP.
This is probably a dealbreaker if you need proper UDP, but if you just want to forward something with HTTPS, that should not be a big issue.
If you expect to handle a large volume of traffic, you might want to consider just getting a public IP or moving that service over to the VPS, as this is really not designed for speed.

## Current best solution (if you don't need high throughput and reasonable domain names)

The above solution is good, but it still requires you to have some manner of external server with a public IP.
This is suboptimal, as you still need to pay extra for a thing you might not otherwise need.
There is a way out of this, however, as long as you don’t need those connections to be of high speed.

That solution is [Tailscale](https://tailscale.com/).
It’s not *technically* designed for this, but it can get the job done.

Tailscale, by default, is used to create a sort-of VPN for your devices.
You install it on any device you want, and those devices can access each other, even if they’re on totally separate private networks.
You can extend that to be a complete VPN by using one of those devices as an „exit-node” which will tunnel all Internet traffic through it.
But another thing you can do is „Tailscale Funnel”.

Tailscale Funnels allow you to publish any service online without any sort of public IP or external server… with *some* limitations.
First of all, there are throughput limits.
I’m not sure exactly how limited that is, but probably don’t expect it to work too well with high traffic or high bandwidth requirements.
Second of all, it heavily restricts which ports you can publish, so publishing things like email servers is not possible at the moment (pretty much only HTTP is doable at this time).
Third of all, you need to use your Tailscale domain name, which is hardly human-readable (I admit I haven’t tried `CNAME`ing it, but I don’t have high hopes for that to work).

All of which is to say that is that it’s not a good solution if you need efficient, professionally-looking hosting of services.
However, if you need that, you generally don’t use your home servers in the first place.
For people that just want to publish some of their home-services, for some reason, it’s an ideal solution, though.
It’s free, low effort, and works perfectly fine.

If you want an example, I sometimes need to upload a file from an external network and an external computer to my network (imagine your friend needing to send you a gigabyte worth of holiday photos).
You can setup some form of pastebin in your home, publish it using Tailscale, and use that service to upload the files.
It’s easy, hassle-free, and definitely simpler in cases when you can’t or don’t want to use services like Dropbox.

## Practical summary

I’ve written that Tailscale is currently the best solution.
That’s true, as long as we’re talking about some personal service, that’s not shared with too many people.
If you want something more public, then doing this will not work well for you, and that’s not only due to bandwidth limitations, as home-hosting larger services is just not practical.
For personal services, though, it works just great.
