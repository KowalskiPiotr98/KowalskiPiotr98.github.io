---
layout: post
title: Self-hosting an email server - the worst best idea
---

## You what?

For the past year and a bit I was self-hosting my own mailserver.
Why use gmail, or outlook, or protonmail, or whatever else is available out there, when you can be the weird guy and do it yourself?

However, as of recently, that mailserver is no more.
To mark this occasion (it's not often you see a side-project to completion, after all), I've decided to sit down and write about things that have happened, and give my thoughts about this whole ordeal.

One thing I must state clearly before you go any further is that this is not a tutorial on self-hosting emails.
You might think it is, as I will give advice on how to do certain things and explain how I've done them, but you should really not use it as a guide for email hosting.
You shouldn't be self-hosting your email anyway, as I'll explain later.
That being said, if you do not heed my warning and go on with email hosting anyway, I hope that you'll find my past experiences usefull on your journey.

## Background

I've been self-hosting various services for some years now.
I wan't doing that because of privacy concerns or anything like that - I just like having Linux boxes to manage, apparently.
The very first project of this nature happened a few years back, and involved 3 VMs hosted in VirtualBox on my main Windows machine - before you ask: it was terrible, but I've managed to configure a DNS server for my local network.

Fast forward a couple of years, ignoring a small episode with me having a VPS to manage, and arrive at the point when I have a home server.
Well... "server" might be a bit generous: it was a Synology Nas and a Raspberry Pi.
But it could run Docker!
At that point I've also started using a very cheap VPS to tunnel traffic into my home network, as I don't have a public IP available from my ISP (perks of living in a big city, I guess).

And then, a seemingly innocent question appeared in my head, that started all of this: "How can you send emails from this thing?".
I needed that as the nas sometimes wanted to notify me of things, and the easiest way of handling that is through email.
Now, yes, I could've just used a gmail account for sending those, as you can send emails using smtp with gmail... but that would be too simple, wouldn't it?
And thus, the journey has begun.

### Side note

Don't confuse self-hosting a mailserver with having your own domain capable of sending and receiving emails.
Before all of that, I've already had my own domain (which is its own arc in this anime) and had it set up with Protonmail.
In other words, I could send and receive emails using an address in my own domain, without hosting anything.
That is also what I'm doing right now, after the server itself got turned off.

## Hosting

To host a mail server, you first need a server. Obviously.
The option that most self-hosting enthusiasts propose is just having your own physical server at home.
I've rejected this approach immediately for 2 reasons, the primary being that I have no public IP, so I literally can't do that.
The other issue is more universal: when things go wrong, you're on your own.

No Internet? Well, you could set up some backup mobile router for those cases if you really want.
No power? Oh, you can always set up UPS and hope all network devices in your area have those too (spoiler alert: they probably don't).
Hardware issues? No problem, just buy 2 of everything and have it ready for swap or, even better, just run a secondary server.
House fire? Floods? Earthquakes? Good luck with that.

That's why, if you need a reliable (or at least more reliable than what's available in the average home) server, it's probably a good idea to just go to some hosting company and buy a VPS.
Or at least it would be, if it weren't for a catch exclusive for email hosting.
See, hosting companies don't want to have a bad reputation.
How do you get bad reputation? Many ways, but sending a lot of spam emails is one of them.
How do they protect themselves against that? By blocking email sending from their servers...

Now, not all of them do that, and usually there are ways around it.
As far as I know, AWS limits outgoing email traffic, Digital Ocean just blocks emails for some time before they decide you're cool, OVH... doesn't care.
Yes, there are no email blocks in place at OVH, so that's what I've gone with.
Obviously, if you abuse that, they can terminate your VPS, but considering the things that are being hosted in OVH, you'd probably need to try hard for that to happen.
Please don't try though.

## The stack

Or in other words: the software that's required to actually run the server.
The first step is simple: use Linux.
Why? Because Windows Server is terrible. But more importantly because the software we're going to use doesn't work on Windblows.
You can run the whatever MS version of email service is called, but unless you somehow get a licence for it, it's probably going to be expensive.

So, Linux then. What else?
Well, in non-email cases, the process is simple: you just run some program and it's done.
If you host a web-app, you just run the app itself, probably an nginx or other apache as reverse proxy, and that's it (I'm talking very simple web-apps here, I know Kubernetes is a *bit* more complex than that).
Even things like DNS are just one executable to run and one set of config files to make (oh the wonderfully weird way Bind is configured...).
But not email.

A "mailserver" is more like a random collection of smaller services than a single thing.
For example, your mailserver might use the following:
- Postfix
- Dovecot
- Amavis
- SpamAssassin
- ClamAV
- OpenDKIM
- OpenDMARC
- Fail2ban
- Fetchmail
- Postscreen
- Postgrey

Now, not all of this is strictly necessary to run the server, but you can clearly see that it's getting quite involved.
What's more, each of these programs is configured independently and they all need to somehow talk to eachother in order to handle your emails.

Anyway, how do you set this up, then?
I dunno, I gave up.
Too many moving pieces for an inexperienced me to attempt and do, especially for a side-project.

What's the alternative then? Well, have someone do this for me!
In this case, I've used [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver), which is exactly what it sounds like: a mailserver in docker.
This wonderful project basically does the whole setup for you, so all that's left for you to do is to fill a config file (and even that is not so simple), and you're ready to `docker run`.
You then need to setup actual email accounts, which you can do with a script provided by the authors of this project, and you're almost ready to ~~send yourself an email~~ continue the setup.

## The DNS (oh man, we're still not done?)

Setting up DNS so that your mailserver works correctly is probably the weirdest part of the setup, because things just go wild here.
As we all know, DNS is used mostly to resolve domain names into IPs or other domain names.
With that in mind, it's now time to setup DNS records, most of which will have nothing to do with domain names or IP addresses.
This will get pretty technical, so if you're not that interested in how email abuses DNS, you can probably skip this one.
For the rest, let's dive in.

First of all, the server itself needs to be reachable.
A simple `A` record for domain `mail.<your actual domain>` pointing to the server IP is fine.
This is the name you'll be using when setting up this email in your desktop client (like Thunderbird od Evolution).
The tricky thing here is that you should also setup proper reverse dns, resolving that IP back to `mail.<domain>`.
If you don't to that, other servers may even reject your email, which is not desirable.
Not all VPS providers allow you to easily do that.

Now that that's done, you need to add MX records.
They are weird, because they have a number attached.
Why numbers? Because people knew you'll want to have a backup of your email server, so they've build a support for that straight into DNS... for some reason.
Anyway, MX records are used by other mailservers trying to send something to your domain.
They'll query the DNS for MX records of the domain (so without the `mail` part), take the one with the lowest number, and get a domain of an actual email server (most likely the `mail.<domain>` from previous point).
That will need to be resolved further, of course, but that's just standard DNS.
Now, the interesting part here is that, if the server doesn't respond, they'll use the next one, with the next lowest value.
This basically makes it so that you can have a hot backup in case something goes wrong.
(By the way, yes, you can add multiple records with the same number. This means the sender can choose any one of them.)

This is where things stop making sense.
Next thing to setup at the DNS side is SPF.
What's that? It's a TXT record for your domain specifying which IPs can send email from your domain.
Why is that necessary? Because when email was designed, cyber security has not yet been invented, so to fix that someone decided to "just use DNS, it's fine".
It's really just a dns record for the domain that returns something like `v=spf1 ip4:66.254.114.41 mx -all` and it's then parsed by the receiver.
If the sender IP doesn't match, another DNS record will tell the receiver to either do nothing, file it as spam, or reject it completely.

The last bit of DNS abuse is the DKIM, or kind-of public signature key that email servers use to verify whether the message was actually sent by the authorised server.
It's a CNAME record. I have no idea why.

It's all fine that the DNS is being used as a security configuration distribution server.
It's not like DNS queries are not encrypted, not verified, and can just be hijacked in the right environment... right?... RIGHT?!
(One day I might write a post about how I'm hijacking all DNS queries in my local network, passing them to my own DNS server, which then uses HTTPS to query an actual server.)

## The rest

At this point, your email should be working and you should be able to send and receive emails.
The client setup is identical to other email providers.
The only catch is that you have no way of actually checking you emails without an email client, but that can be rectified with a simple [roundcube](https://github.com/roundcube/roundcubemail) installation.
The very important thing to do now is configure 3 things, without which things will go wrong: backups, backups, and monitoring.
Why backups twice? You need to backup the data from the server in case it gets lost, and you need a backup server in case this one fails.

Data backups were fairly simple in my case: I've setup a script on my nas at home that would ssh into the server a couple of times a day and just rsync the contents of docker containers to itself.
The nas itself was then replicated so that this data would not get lost (my backup solution is another blog post to write).

Server backup is kinda more involved, since you'd ideally have it in an entirely separate data centre (even using a different provider, for reasons I'll get into in a bit).
I just used [ProtonMail](https://proton.me) but that's because I've already had an account there anyway.
You should look for other providers, but I'd strongly suggest not self-hosting the backup.
And this actually saved me once.
While this server was live, OVH has experienced an issue where it just totally dropped off the Internet.
If I had no backup, or if I was hosting it on another OVH machine, I could risk losing email.
Since I was using ProtonMail instead, I just received emails with my Proton inbox for a while, until OVH resolved their issues.
This is the "hot backup" I was writing about in the DNS section.

The last thing you absolutely need is monitoring. Two types of it, actually: server metrics monitoring and downtime monitoring.
Downtime monitoring is quite simple: you just periodically ping one of the ports on your server to see if it responds.
If it doesn't, you raise an alarm.
Seems easy, was a bit of a headache due to fail2ban blocking the IP from which those tests were running, but managed to get it working by the end.
To make this happen, I've used [Uptime Robot](https://uptimerobot.com/).

As for metrics collection, I've been using [New Relic](https://newrelic.com/) for that.
It's very easy to setup and can be used quite extensively before you have to pay anything.
It's also proven useful many times, especially that one time when [there was an issue](https://github.com/docker-mailserver/docker-mailserver/issues/2154) causing CPU usage to spike (huge thanks to the devs for fixing that over the weekend).
Newrelic is also capable of collecting logs which, and I can't stress this enough, you **MUST** read.
Maybe not all of them, as there can be quite a number, but at least find a subset of more important ones (like warnings etc.) and read those.
Apart from that, I've also had logwatch setup to send me daily emails with server stats and higher priority logs (this is included in the docker-mailserver by default).

## The domain fiasco

At the beginning, there was a domain.
A very cheap one, bought at [Namecheap](https://www.namecheap.com/).
The tld was `.pw` - because it was cheap.
Can you see the problem here? No? Well, for the longest time I couldn't as well.

The issue first became apparent when, after a docker-mailserver update, I've enabled some community spam filtering.
Suddenly, some of my log digests were being marked as spam... which was worrying.
Upon further investigation I've found that they were being marked as spam by a rule called `KAM_SOMETLD_ARE_BAD_TLD`.
What's that?
Well, as [one user mentions](https://lowendtalk.com/discussion/139820/spamassassin-kam-sometld-are-bad-tld):
> If you send emails using a TLD like:
 .stream, .trade, .pw, .top, .press, .bid & .date 
 You have a high probability to become considered as SPAM.

Okay... now what?
Simple, really... buy a new domain, setup the server to accept both of them for backwards compatibility, change the old email to the new one everywhere I've used it, and done!
The most ridiculous thing is that the last step was the most difficult one.
Apparently, in 2022, some websites don't allow you to change your email...

Or, as is the case with [BilKom](https://bilkom.pl/), they do something so hilariously weird, I have to mention it.
When you sign up for an account, your email becomes your login.
Usually in those situations, those 2 fields are just bound to the same database column, or at least are always changed at the same time... but not here.
Your login is hard-wired to your first email. You can't change it.
You can change your email and that works fine, but I just can't shake the feeling that someone will at some point mix those 2 values up and will try to send an email to my login instead of my email, because "well, they're always the same anyways".

PayPal is also of note here, as apparently it blocks money transfers out of your account when you have a `.pw` email.
I've only found out about this when, after changing it, I got a notification saying that I'm now able to do that.

## So, how was it?

Surprisingly... uneventful.
Well, yeah, there were the cases I've described above, but that's much less issues than I was anticipating.
During the slightly over a year of running, there was only one instance of me losing emails (that I know of), and that was only because DHL has their email servers setup so incorrectly, my server treated them as a high risk spam.
For the most part though, this server was just sitting there doing its thing with very little maintenance necessary.

Was it worth it?
Uhh... I guess, maybe?
Look, the journey was definitely worth it, as I've learned things about hosting, DNS, and so much about email I'll never be able to enjoy Steins;Gate again.
But the end result, as in: the email server that just worked... was much less exciting and much more tiring than I've hoped for.
You need to check for updates from time to time, read logs daily, and make sure nothing breaks every time docker-mailserver updates.
And in return you get the possibility of having infinite accounts for your domain for free.
Well, not free, as I was running it on a dedicated VPS, to make sure nothing unrelated to email kills the machine.
And having infinite emails... it's nice for sending email if you self-host things, but you don't an email server that can receive emails for that.
Depending on the volume of emails, you might not need a server at all, because things like [SendGrid](https://app.sendgrid.com/) exist.

## Conclusions

It was definitely a journey.
I guess I could call this project a "success", if you want that sort of declaration.
I mean, it worked fine, I've had no major issues using it, and I've learned things, so it's all good!

Now, if after all that you're still considering self-hosting an email, here's why you should and shouldn't do it.

### Why should you self-host your email?

You'll learn a ton about email, probably more than you need.

You might also consider self-hosting when you have lots of other self-hosted services and just need a notifications server, but for some reason other services like SendGrid or Amazon SNS don't work for you.

For the privacy nuts out there: yes, this also gives you control over your data, as long as you ignore the fact that the hosting company probably has access to your VPS.

~~You can always use it to pick up girls.~~

### Why should you not?

Self-hosting email is generally a bad idea, for many reasons, and whatever your case may be, I'd advise to only do it as a last-resort.
When something breaks, you're on your own - and while that might be one of the "fun" things about self-hosting, it's probably not the best for critical services like emails.
There are also risks you can't do anything about: apart from the domain situation, apparently some companies do the same with IPs:
    if there's a single IP in a cidr that's doing bad things (scanning the Internet, brute-forcing passwords, sending spam), they'll just treat the entire cidr as spammy and will block your emails.
Now, I think haven't had that happen, but you might, and you might not even know that it's happening, as you don't get notified your mail was filed as spam somewhere else.

The learning curve is also very steep: for the first couple of months I've only used this server to receive non-critical emails - only after I've seen it working for some time have I switched to receiving more important ones.
But if you get things wrong, you might be losing emails, or emails sent by you might be getting lost, and you might not even know about this, as that's not so easy to test.

