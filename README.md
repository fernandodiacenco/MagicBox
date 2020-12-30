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

First I recommend that on yout main router you configure your dhcp settings to deliver IPs at later values, for example if your network is 192.168.1.0/24 I recommend your dhcp starts at 192.168.1.21 to 192.168.1.254, this way you reserve the first 20 or so ips for devices that are better configured with static ips, for example printers, access points, ip cameras, voip phones, smart lamps, the magic box, etc

Then decide what IP address will be assigned to the box, this is just for managing it when necessary and has no value in connectivity since it will be transparent of the network

So lets say it is: 192.168.1.1, access it on your browser

Go to SYSTEM -> SYSTEM and change the HOSTNAME to something more apropriate, for example MAGICBOX or SQM or whatever you decide

Go to SYSTEM -> ADMINISTRATION And change your router password

Go to NETWORK -> INTERFACES and now the fun begins...
