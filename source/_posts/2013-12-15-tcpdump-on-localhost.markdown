---
layout: post
title: "tcpdump on localhost"
date: 2013-12-15 15:03:14 -0800
comments: true
categories: [TCP/IP]
---

[tcpdump](http://en.wikipedia.org/wiki/Tcpdump) is a powerful tool that allows
you to capture TCP/IP traffic over a network. It can be too smart sometimes.
Here's a story of how I got baffled about tcpdump.

<!-- more -->

I was debugging some weird behavior of a production service and thought I'd
bring up a local instance of the service, do a tcpdump, and see what's going on.

I made sure I only had one Internet interface running (en0). I started tcpdump
capturing traffic on that interface: `tcpdump -i en0 port 8080`. Then I pointed
a local client to talk to it by specifying the IP of the service as
**10.30.0.86** which is the internal IP of my laptop.

Fired up the client and it was talking to the local service. I saw logs on both
ends which confirms they were talking to each other. However, surprisingly, I
didn't see anything on the tcpdump console.

I'm not a tcpdump expert, so I fired up [Wireshark](http://www.wireshark.org/)
and then ran the client again. Applied a Wireshark filter that only lists
traffic between 10.30.0.86 and 10.30.0.86. Not so surprisingly, Wireshark
displayed no traffic as well.

So what's going on? tcpdump and wireshark haven't failed me much in the past. So
I trust them. Why are they not working this time?

Fortunately, my team has a linux guy who knows about linux networking well. His
theory was that the TCP/IP stack of Mac is trying to be smart about
local IPs. If it detects both the client and the server are running on
the same local host, it would use the **loopback** interface (ie lo0)
which apparently is faster than a real IP for some reason.

So I tried again and this time I asked tcpdump to capture traffic on **lo0**.
Magically, tcpdump started spitting out logs on the console.

Lesson learned.
