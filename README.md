# MagicBox
This is a how to on creating a "Magic Box", a device (or virtual machine) that will "Magically" improve the network.

The original idea came from here: https://apenwarr.ca/log/20180808
I recommend reading it, to understand the context behind the idea

I've used this Magic Box in both a physical device at home using a cheap TP Link WR740N hardware version 5 and a transparent virtual machine under Hyper-V at work, with great success

What you need is a device capable of running OpenWRT with enough space for SQM-SCRIPTS package, and know your Internet connection speed. This device will be placed between Your main Router and LAN and transparently lower the overwall network latencies

So if your home network looks like this:

INTERNET PatchCord MAIN ROUTER PatchCord LAN

It will them look like this:

INTERNET PatchCord MAIN ROUTER PatchCord MAGIC BOX PatchCord LAN

In the following example, I'm using the mentioned TP-Link WR740N

---
<B>CREATING A CUSTOM BUILD</B>

<b>This part is only necessary if you need to build a custom image for your device (due to space constraints or whatever), if you can use a standard openwrt build, skipt it.</b>

In the case of the WR740N build that I'm using, is does not have enough internal space for both the standard OpenWRT Image and the SQM-SCRIPTS, so it is necessary to build your own custom image, this is done by downloading the image builder for your device, in this case it is

https://downloads.openwrt.org/releases/19.07.5/targets/ath79/tiny/openwrt-imagebuilder-19.07.5-ath79-tiny.Linux-x86_64.tar.xz

Extract it, open a terminal on this folder, and now yit is possible to create a custom image

In this example, it was removed everything not essential to the box working, including wireless, firewall, ppp/pppoe mods, anything related to ipv6, dns, dhcp, the package manager and even the log application, it was added the graphical interface (for ease of configuration, but can be done without it), so the command line is

make image PROFILE=tplink_tl-wr740n-v5 PACKAGES="sqm-scripts luci luci-app-sqm -wpad-mini -odhcp6c -odhcpd-ipv6only -ppp -ppp-mod-pppoe -firewall -iptables -ip6tables -dnsmasq -opkg -logd"

When finished, the build will be located at the folder: build_dir/target-mips_24kc_musl/linux-ath79_tiny/tmp

Plug in the device to your computer/notebook

Now you have to flash the new build to your device, if it already had a OpenWRT on it it just a matter of reseting to default configuration, and using ssh to upload and flash

scp /your_folder/your_build.bin root@192.168.1.1:/tmp

ssh root@192.168.1.1

sysupgrade -v -n -F /tmp/*.bin

This will flash the new build and restore again to default settings

After finishing you can access the interface on http://192.168.1.1 login: root no password, or you can configure by ssh

---

<b>CONFIGURATION</b>

First I recommend that on your main router you configure your dhcp settings to deliver IPs at later values, for example if your network is 192.168.1.0/24 I recommend your dhcp starts at 192.168.1.21 to 192.168.1.254, this way you reserve the first 20 or so ips for devices that are better configured with static ips, for example printers, access points, ip cameras, voip phones, smart lamps, the magic box, etc

Then decide what IP address will be assigned to the box, this is just for managing it when necessary and has no value in connectivity since it will be transparent of the network

So lets say your primary router is on IP 192.168.1.1 and you choose for the SQM the IP 192.168.1.2, access this IP on your browser (http://192.168.1.2)

Go to SYSTEM -> SYSTEM and change the HOSTNAME to something more appropriate, for example MAGICBOX or SQM or whatever you decide

Go to SYSTEM -> ADMINISTRATION And change your root password

Go to NETWORK -> WIRELESS and delete the wireless interface, it is not needed

Go to NETWORK -> INTERFACES and delete the WAN6 Interface, it is not needed

Add New Interface -> Name: BRIDGE - Protocol: Static, IP: 192.168.1.2, MASK 255.255.255.0, Containing the interfaces: Eth0 & Eth0.1

Save and apply, and now check if you can still access the IP

---
<b>CONFIGURING THE SQM</b>

Now this is the magic part in the magic box, the SQM (Smart Queue Management) is what will make the connection better

Usually you just instantiate the SQM on the WAN interface, defining Download and Upload Speed and the Scheduler and be done with it

BUT there is a catch: If you instantiate the SQM on both WAN and LAN interfaces in one direction only (EGRESS) you bypass the IFB (usually used if you have a wireless interface on the same router) and you benefit from about 5% less overhead, making the connection more efficient

ANOTHER catch: This trick is for a box where traffic passes TROUGH it, if you are for example hosting something IN the box, the SQM cannot properly calculate the traffic and you are better served instantiating sqm on the WAN interface with download/upload, But in this example the box is transparent and not hosting anything

So, Go to NETWORK -> SQM

    Select WAN Interface
    Check [x] Enable SQM
    Set Download Speed to 0 (Disable)
    Set Upload Speed to your UPLOAD speed in bits, for example 10000 equals to 10mbps
    Go to Queue Discipline and select Cake / Piece of Cake
	  Check [x] Show and use advanced config
	  Then Check [x] Show and Use Dangerous Configuration.
	  Advanced option string to pass to the egress queueing disciplines: <b>dual-srchost</b>
    Go to Link Layer Adaptaton Tab
    Select Which link layer to account for: Ethernet with Overhead
    Per Packet Overhead (byte): 44
	
    Now click ADD and select LAN interface
    Check [x] Enable SQM
    Set Download speed to 0 (disable)  
    Set Upload Speed to your DOWNLOAD speed in bits, for example 20000 equals do 20mbps
    Go to Queue Discipline and select Cake / Piece of Cake
	  Check [x] Show and use advanced config
	  Then Check [x] Show and Use Dangerous Configuration.
	  Advanced option string to pass to the egress queueing disciplines: <b>dual-dsthost</b>
    Select Which link layer to account for: Ethernet with Overhead
    Per Packet Overhead (byte): 44

Save and apply

The best way to check if its working ig disabling SQM on both interfaces, going to https://dslreports.com/speedtest, running the test and making note of the "bufferbloat" at the end, enabling SQM again, and re-running the test, the bufferbloat score should have increased from whatever was previous to an A+

---
<b>INSERTING THE BOX ON YOUR NETWORK</b>
The magic box would sit between the main router and the rest of the network, so in this case I am assuming a simple switch for this diagram

INTERNET - MAIN ROUTER <patch cord on the WAN interface> MAGIC BOX <patch cord on the LAN interface> SWITCH - DEVICES

This way all the traffic must pass trough the box, but it is still reachable to configure/adjust by the IP address you assigned to it
---
<b>CONCLUSION</b>

You can use this box in any Openwrt box, may it be an access point or a x86-64 box to improve your connection, it would work even without LUCI (the graphical user interface of openwrt) as long as you have the sqm-scripts package installed and can configure the system over ssh

In my experience I use a bunch of old TPLink wr740n v4/v5/v6 routers and Magix Boxes, they are limited to 100mbps at the lan so this would be the maximum download/upload it can achieve, and so far I didn't suffer from speed constraints due to CPU usage on saturated network activity, but your mileage may vary

I also have two magic boxes in virtual machines at work, running as transparent bridge, no problems so far

You can play around with the configuration to find the absolute best option for you, for example in the queue discipline tab you can choose Layer_Cake instead of Piece_of_Cake if you have a more powerfull router/machine, or you can choose simple.qos or simplest.qos if you are suffering from CPU constraints, perhaps even choosing fq_codel instead of cake

On the Link Layer Adaptation tab you can adjust the values to be more exact to your environment, but in the case you are not sure do overestimate, it is always better to overestimate the linklayer than to underestimate, it is not that the box will stop working and the world will end if you underestimate, but it will be less efficient if you do.

---
<b>REFERENCES</b>
https://forum.openwrt.org/t/transparent-cake-box/2161/23

https://forum.openwrt.org/t/transparent-cake-box-where-to-apply-sqm/78698

https://openwrt.org/docs/guide-user/network/traffic-shaping/sqm-details

https://github.com/moeller0/ATM_overhead_detector
