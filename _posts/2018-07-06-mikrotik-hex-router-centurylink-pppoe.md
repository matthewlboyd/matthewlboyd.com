---
layout: post
comments: true
title: Using a Mikrotik Hex router with Centurylink fiber and PPPOE
---

# Intro

Living in a burgeoning technology city like Portland, one would think you would have several options for
an ISP. However, like most places, Portland is limited to just two options: Comcast or Centurylink
(Google Fiber being the perennial tease).

Centurylink recently ran a special where you could get 40MBS fiber internet for 55$ a month perpetually,
supposedly without future price increases. I'm a cheap dirtbag, so the price was right.

Cneturylink tries to get you to use their crumby Zyxel router for something like 10$ a month.
With the setup below you can get rid of the subpar router that Centurylink wants to "rent" to you, and get
an "enterprise" level home network for about 150 bucks.

Inspired by the post [here](https://arstechnica.com/gadgets/2016/09/the-router-rumble-ars-diy-build-faces-better-tests-tougher-competition/), I
decided to go with a Mikrotik Hex router.

The exact Mikrotik model I am using is this one: [Mikrotik hEX RB750Gr3](https://amzn.to/2KThhap).
Coupled with a single [Ubiquiti Unifi Long Range AP](https://amzn.to/2ucPWch) I get good (20-30Mbps) Wifi throughout
my two story home.

I've been really happy with my Mikrotik Hex router since I got it all setup over the winter holidays.
It uses very little power while still providing a lot of configuration options and advanced features for
my scrub-level networking skills.

# Prequisites

1. Call Centurylink and get your PPPOE username and password

2. Open up all the guides and how-tos that you will need, as you will be without internet during setup.

3. A couple ethernet cables to connect your laptop to the router.

4. Download Mikrotik's Winbox program if you want to use it: https://mikrotik.com/download.

5. Self-confidence! The setup looks complicated, but it should go pretty smoothly.

# Configuration

## Step 1
Plug the ethernet cable from your ONT into the far left port on the Hex router. Run a patch cable from the second from the left port on the router to your computer.

## Step 2
Log into the router via Winbox or in your browser at 192.168.88.1. The default credentials are admin without a password.

## Step 3
Go into System > Users and setup a new "full" user with an actual password. Log out from the admin account, and log back in as
the new user.

## Step 4
Open the Terminal on the router and run the following two commands to setup pppoe and the correct VLAN configuration for Centurylink:

`/interface vlan
add interface=ether1 name=e1-v201 vlan-id=201
/interface pppoe-client
add add-default-route=yes disabled=no interface=e1-v201 name=pppoe-out1 password=yourpw user=youruser
`

## Step 5
For some reason, Centurylink's DHCP didn't give me a DNS server, so I had to go into IP > DNS and setup a server there.
Use 8.8.8.8 for Google, or your DNS server of choice.

## Step 6
Open up the Terminal again and run the following command to configure the NAT masquerade rule to use our pppoe interface:

`/ip firewall nat
add action=masquerade chain=srcnat comment="defconf: masquerade" ipsec-policy=out,none out-interface=pppoe-out1
`

## Step 7
You should have internet at this point. Rejoice! Take the crappy Zyxel router back to Centurylink, and get them to stop charging you the monthly
router rental. Make sure to get a receipt or something, as they are known to forget that you don't still have their router.
The next step would be to get a Wifi AP setup, which I will go over in a later post.

Here are some screenshots showing what the Interface and NAT Rules should look like at the end (right click > open in new tab for a larger image):
![alt text][interfacelist]

[interfacelist]:/public/interfacelist.png "Interface List"

![alt text][natrules]

[natrules]: /public/natrules.png "NAT Rules"

## Troubleshooting
Below is the "/export compact" output from the Mikrotik router terminal. This shows all the settings that differ from the default settings. If you compare your own "/export compact" to the one below, you should be able to narrow down any relevant differences. If you're still having trouble, feel free to add a comment, and I'll try to help out:
```
/export compact  
# jul/06/2018 21:15:00 by RouterOS 6.42.5
# software id = IMBD-LQQ9
#
# model = RouterBOARD 750G r3
# serial number = 6F3907EA0947
/interface bridge
add admin-mac=[redacted] auto-mac=no comment=defconf name=bridge
/interface vlan
add interface=ether1 name=e1-v201 vlan-id=201
/interface pppoe-client
add add-default-route=yes disabled=no interface=e1-v201 name=pppoe-out1 password=[redacted] user=[redacted]
/interface list
add comment=defconf name=WAN
add comment=defconf name=LAN
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip hotspot profile
set [ find default=yes ] html-directory=flash/hotspot
/ip pool
add name=dhcp ranges=192.168.88.10-192.168.88.254
/ip dhcp-server
add address-pool=dhcp disabled=no interface=bridge name=defconf
/interface bridge port
add bridge=bridge comment=defconf interface=ether2
add bridge=bridge comment=defconf interface=ether3
add bridge=bridge comment=defconf interface=ether4
add bridge=bridge comment=defconf interface=ether5
/ip neighbor discovery-settings
set discover-interface-list=LAN
/interface list member
add comment=defconf interface=bridge list=LAN
add comment=defconf interface=ether1 list=WAN
add list=WAN
/ip address
add address=192.168.88.1/24 comment=defconf interface=ether2 network=192.168.88.0
/ip dhcp-client
add comment=defconf dhcp-options=hostname,clientid interface=ether1
/ip dhcp-server network
add address=192.168.88.0/24 comment=defconf gateway=192.168.88.1
/ip dns
set allow-remote-requests=yes servers=8.8.8.8
/ip dns static
add address=192.168.88.1 name=router.lan
/ip firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMP" protocol=icmp
add action=drop chain=input comment="defconf: drop all not coming from LAN" in-interface-list=!LAN
add action=accept chain=forward comment="defconf: accept in ipsec policy" ipsec-policy=in,ipsec
add action=accept chain=forward comment="defconf: accept out ipsec policy" ipsec-policy=out,ipsec
add action=fasttrack-connection chain=forward comment="defconf: fasttrack" connection-state=established,related
add action=accept chain=forward comment="defconf: accept established,related, untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
add action=drop chain=forward comment="defconf:  drop all from WAN not DSTNATed" connection-nat-state=!dstnat connection-state=new in-interface-list=WAN
/ip firewall nat
add action=masquerade chain=srcnat comment="defconf: masquerade" ipsec-policy=out,none out-interface=pppoe-out1
/system routerboard settings
set silent-boot=no
/tool mac-server
set allowed-interface-list=LAN
/tool mac-server mac-winbox
set allowed-interface-list=LAN
```