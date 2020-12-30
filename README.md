# MagicBox
This is a how to on creating a "Magic Box", a device (or virtual machine) that will "Magically" improve the network.

The original idea came from here: https://apenwarr.ca/log/20180808
I recommend reading it, to understand the context behind the idea

I've used this Magic Box in both a physical device at home using a cheap TP Link WR740N and a transparent virtual machine under Hyper-V at work, with great success

What you need is a device capable of running OpenWRT with enough space for SQM-SCRIPTS package, and know your Internet connection speed. This device will be placed between Your main Router and LAN and transparently lower the overwall network latencies

So if your home network looks like this:

INTERNET
|
MAIN ROUTER
|
LAN

It will them look like this:

INTERNET
|
MAIN ROUTER
|
MAGIC BOX
|
LAN

In the following example, I'm using the mentioned TP-Link WR740N

---
