---
layout: post
title: (Pi)Holes in my home network
---

## The DNS adventures

At some point, I've deployed a server in my home network.
It was only a simple NAS, but it still counts.
With that server came the need of having a local dns-server.
Well, it wasn't strictly required, but it's always nicer to be able to connect to things using a domain name, instead of just an IP address.

The DNS itself is not hard to configure, especially that Synology offers a built-in DNS server - you only need to point to it from your router in DHCP settings.
So, at the very start, I did just that.
That was enough for me for some time, but as it usually is with my side-projects, I've decided that I needed to go deeper.

## DoH, DoT... DIY

At some point I've stumbled upon [an article](https://www.cloudflare.com/en-gb/learning/dns/dns-over-tls/) about DoH and DoT.
What's that?
It's a way of encrypting DNS queries.
Because yes, by default, your DNS queries are not encrypted in any way.
In fact, it's entirely possible to just hijack and forward them to a completely different server (that'll come in handy later).

DoH (DNS over HTTPS) and DoT (DNS over TLS) change this.
Sounds good, but as with many good-sounding things, there's a catch:
    it's not that simple to turn them on - they need to be supported by the client system to work.
And that's a problem, as even thoug most browsers currently support that, and phones sometimes do too, but the OS support in general is mostly non-existent.
For that reason, I've decided to approach this issue differently.

What I've decided to do was to setup a local DNS server that could be queried using standard, non-encrypted way.
That server, however, wouldn't just query another DNS, but would forward all queries using a Do\[HT\].
By doing so, no special cliend-side support would be necessary, as the device would be able to query the local DNS in a classic way.

## Cloudflare to the rescue

Forwarding queries turned out to be quite an easy thing to do.
Cloudflare offers both DoH and DoT, and they even offer just a forwarder like that: [cloudflared](https://github.com/cloudflare/cloudflared).
It can even be run in docker, so setup is very simple:
```yaml
version: '3'

services:
    tunnel:
        image: cloudflare/cloudflared:latest
        restart: always
        ports:
            - "0.0.0.0:53:5053"
        command: tunnel proxy-dns --port 5053 --upstream https://1.1.1.2/dns-query --upstream https://1.0.0.2/dns-query
```
And that's it!
You can just deploy that as your local DNS and all of your queries will be securely forwarded using https to Cloudflare DNS.

Note that the upstream address is `1.1.1.2`, not the usual `1.1.1.1`, but this server blocks certain domains considered malicious, so that's an extra level of security.

Anyway, that's all nice and good, but there's just one problem: cloudflared itself doesn't allow for any local dns configuration.
To solve that, you'd need a DNS server downstream of cloudflared.
That server would then forward queries upstream to cloudflared, that would then forward them using HTTPS to Cloudflare.

I'm not totally sure what happened here, but I remember having some issues with the Synology nas.
Since it had its own DNS server, port 53 was already being used, so I had to do some magic with this one to make it work.
That's not really important, however, since we're about to go one step further in all of this.

## There's a (Pi)hole in your logic

In case you've never heard of [Pi-hole](https://pi-hole.net/), it's a very lightweight DNS server that promotes itself as a way to stop ads in your network.
It can actually do much more, for example be used to block phishing attempts by blocking certain domains, but we'll come to that.

Anyway, it's a DNS server, and a more configurable one than what I was running on my nas.
So let's deploy it!
The name *Pi*-hole suggests it's supposed to be deployed on a Raspberry Pi, but that's not a requirement.
It can just as well be run on any other system, as well as in Docker, which is the way I've been using it for some time.

The setup is... depending on your needs... either super simple or very weird.
Let's start with the easy option: simply deploying this to your Pi is super easy.
The guided setup is brilliant and as soon as you point your DHCP to the Pi-hole, it'll work.

But we're not here to do things the easy way, right?
Running in Docker on Synology NAS has a certain weirdness to it, because the client IPs you'll see in the web-interface will make no sense.
Fortunately, there's an easy fix for that, that I've found somewhere on the Internet and haven't saved the source...
Anyway, it's that:
```bash
#!/bin/bash
currentAttempt=0
totalAttempts=10
delay=15

while [ $currentAttempt -lt $totalAttempts ]
do
	currentAttempt=$(( $currentAttempt + 1 ))
	
	echo "Attempt $currentAttempt of $totalAttempts..."
	
	result=$(iptables-save)

	if [[ $result =~ "-A DOCKER -i docker0 -j RETURN" ]]; then
		echo "Docker rules found! Modifying..."
		
		iptables -t nat -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
		iptables -t nat -A PREROUTING -m addrtype --dst-type LOCAL ! --dst 127.0.0.0/8 -j DOCKER
		
		echo "Done!"
		
		break
	fi
	
	echo "Docker rules not found! Sleeping for $delay seconds..."
	
	sleep $delay
done
```
Just schedule this to run on startup and the client IPs will start working fine.
If you're installing on bare-metal or in Docker on normal Linux, you most likely won't need that at all.

But that's not the end of the journey - we still need to make this work with cloudflared.
That's where things start getting silly: we need the pi-hole to be the forward-facing DNS server, and then forward queries from that to cloudflared.
Notice how cloudflared doesn't actually need to be accessible from any place other than the Pi-hole.
Trying to set them up as a DNS server on the same machine would be difficult, because only one of them could use port 53.

Anyway, with that in mind, let's get to work.
Let me just paste the `docker-compose.yml` here, as it'll make it easier to explain things:
```yaml
version: '2'
services:
    pi-hole:
        container_name: pihole
        image: pihole/pihole:latest
        restart: always
        hostname: dns-brick.home
        cap_add:
            - NET_ADMIN
        ports:
            - "127.0.0.1:8053:80/tcp"
            - "192.168.1.12:53:53/udp"
        depends_on:
            - tunnel
        networks:
            dns_net:
                ipv4_address: 172.16.238.10
        environment:
            TZ: 'Europe/Warsaw'
            WEBPASSWORD: ':)'
            ServerIP: '192.168.1.12'
            PIHOLE_DNS_: '172.16.238.11'
            WEBTHEME: 'default-darker'
            VIRTUAL_HOST: 'dns-brick.home'
            FTLCONF_REPLY_ADDR4: '192.168.1.12'
        volumes:
            - './etc-pihole/:/etc/pihole/'
            - './etc-dnsmasq.d/:/etc/dnsmasq.d/'
            - './var-log/pihole.log:/var/log/pihole.log'
            - './resolv.conf:/etc/resolv.conf:ro'
    tunnel:
        image: cloudflare/cloudflared:latest
        restart: always
        user: root
        networks:
            dns_net:
                ipv4_address: 172.16.238.11
        command: tunnel proxy-dns --address 172.16.238.11 --port 53 --upstream https://1.1.1.2/dns-query --upstream https://1.0.0.2/dns-query
networks:
    dns_net:
        driver: bridge
        ipam:
            config:
              - subnet: 172.16.238.0/24
                gateway: 172.16.238.1
```
A few things to note that are necessary to understand this: `dns-brick.home` is the hostname of the server this is hosted on, `192.168.1.12` is the IP of this machine.

Now, from the top:
- the `hostname` set for `pi-hole` is mostly for the Pi-hole front end to display it correctly, I don't think it has any other uses,
- `cap_add` I don't think is actually necessary, but I've added it there at some point for some reason, and so now it lives there,
- `ports` are: port 53 for the DNS itself (must be bound to an IP that's accessible from the network), and port 8053 for the front-end management portal (this could be accessible from the network, but I'm running this behind a reverse-proxy),
- `networks` now this is a weird one: I've had to set static internal IP addresses for the docker container, as otherwise they'd be unable to communicate with themselves, as you can't set a DNS server using a domain name, because you need a DNS for that...
- `volumes` notice the overridden `resolv.conf` in there, inside of this file is `nameserver 172.16.238.11`, as otherwise the docker container itself would not have DNS, and that would break certain Pi-hole features,
- now for the `tunnel` container: the `user` needs to be `root` to be able to bind to port 53,
- `networks` assigns a static IP for this container, the same IP as in the `resolv.conf` file and the one that Pi-hole will forward requests to,
- `command` is an actual command that will be run by `cloudflared`, `address` must be set to the static container ip, and `port` to 53 to be able to handle DNS queries on the default port.

If you think that you'll try to set everything up from scratch... trust me, everything I've written above here is the result of hours of debugging.
The main issue here is that you quite often cannot configure DNS clients to use any port other than 53, and it's impossible to set DNS using a domain name.
Due to that, a lot of that `docker-compose.yml` is just dealing with working around those limitations.
And the final result is that you have a pi-hole capable of forwarding requests to cloudflared, which will then forward those requests over HTTPS to Cloudflare DNS.
You can also configure local DNS in pi-hole, setup blocklists and have a caching server.
Just make sure that after all of that, you Pi-hole upstream DNS settings look like this:
![](/assets/pi-hole-dns-settings.png)

## Blocklists

This one is quick, as it only explains a feature of Pi-hole.
You can setup blocklists to basically block requests to certain domains.
More global lists are available [here](https://github.com/blocklistproject/Lists), but you should also look at the site of your local CERT/CSIRT to see whether they provide Pi-hole compatible list for hacking attacks as well.
[CERT Polska](https://hole.cert.pl/domains/domains_hosts.txt) certainly does.

## Redundancy

It's always good to have a backup DNS server.
If you have only one, losing it for whatever reason (planned or not) means essentially losing access to the Internet.
That's why it's also important to not have both of them on the same server.
Fortunately, [through the magic of buying two of them](https://www.youtube.com/watch?v=CZs-YcmxyUw), you can just do the exact same thing on another server and it'll work fine.
Just remember to add both of them to your DHCP, so that your devices will know of both of them.

The only issue here is that Pi-hole doesn't really have any way of automatically replicating itself to another inscance.
Most DNS servers (like bind) have a way of setting primary and secondary servers: when you change DNS records on the primary one, the secondary will update itself automatically.
Pi-hole doesn't support that, but I'm gonna guess that your local DNS records don't change that often, so it may not be that big of an issue for you.
It still can be done manually though, just go to Settings -> Teleporter, generate the file and upload it to your second pi-hole instance and you're good to go.
You'll either have to do it every time you change some settings, or just remember to always do the same change on both instances.
Either way, I'm fumbling around with my local network a lot and that's not been an issue for me, so I presume you'll be fine as well.

## Bonus: blocking requests to other DNS servers

Remember how I've said that you can hijack DNS queries?
So, the thing is, if some app is using a hard-coded DNS server, all of this setup can be circumvented.
Does a thing like that actually happen?
Well... apparently some apps do that do avoid adblockers... supposedly.
Can we stop them? Sure we can!

For that you'll need something extra, though.
You need to be able to set DNAT rules on your router.
Most ISP-provided ones don't allow you to do that, unfortunately.
In my case, I have a router with [OpenWrt](https://openwrt.org/start) installed.
If you have that, just go to Network > Firewall > Custom Rules and add this:
```
iptables -t nat -A POSTROUTING -j MASQUERADE
iptables -t nat -I PREROUTING -i br-lan -p udp --dport 53 -j DNAT --to <your DNS IP>:53
```

What that does is this: any requests leaving your LAN going to port 53 (the DNS port) will be redirected to your local DNS instead.
Since DNS is not secured in any way by default, this will work fine, as the client has no way of actually validating where the response came from.
This will also not redirect the DNS over HTTPS requests, as they use port 443, like all other HTTPS traffic (otherwise you'd need to allow requests to port 53 from your local server).
As I've said though, this most likely requires either custom router firmware or a much more expensive, pro-level router to set.
And I don't think it's that critical to do, so if you don't already have a router capable of doing this, don't feel compelled to get one.

## Final words

This setup has been working for me for a long time now.
I've had no issues with it, trackers are being blocked (I've even cut everything Facebook related from my network), so I'm happy.

Okay... I've had one issue.
I'm running Linux on my laptop, and I have a Windows VM installed on it.
It regularly fails to complete DNS requests in this network and thinks it has no Internet.
I have no idea why, but I blame Windows.

Would I recommend this setup?
Sure, if you already have some systems that are running 24/7, or are willing to buy a Raspberry Pi for it.
Don't try running this on something that is turned off regularly, as it will mean you won't be able to use Internet, or on something very powerful, as Pi-hole uses very little resources.
But if you have something to run this on, then definitely do give it a go.
Pi-hole itself requires very little maintenance, and as it's something deployed only to your local network, you don't have to go crazy on monitoring.

Also, if you do, please consider [donating] (https://pi-hole.net/donate) to the people behind this awesome tool.
