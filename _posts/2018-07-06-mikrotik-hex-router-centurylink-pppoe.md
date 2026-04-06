---
layout: post
comments: true
title: Using Mikrotik Hex router with Centurylink fiber and PPPOE
---

# Intro

With this setup you can get rid of the subpar router that Centurylink wants to "rent" to you.

Inspired by the post [here](https://kmwoley.com/blog/bypassing-needless-centurylink-wireless-router-on-gigabit-fiber/), I
decided to setup a Mikrotik Hex router to work with my Centurylink fiber internet here in Portland, Oregon.

The exact Mikrotik model I am using is this one: [Mikrotik hEX RB750Gr3](https://amzn.to/2KGcHQI)

I've been really happy with my Mikrotik Hex router since I got it all setup over the winter holidays.
It uses very little power while still providing a lot of configuration options and advanced features.

# Prequisites

1. Call Centurylink and get your PPPOE username and password

2. Open up all the guides and how-tos that you will need, as you will be without internet during setup.

3. A couple ethernet cables to connect your laptop to the router.

4. Download Mikrotik's Winbox program if you want to use it: https://mikrotik.com/download

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

Here are some screenshots showing what the Interface and NAT Rules should look like at the end:
![alt text][interfacelist]

[interfacelist]:/public/interfacelist.png "Interface List"

![alt text][natrules]

[natrules]: /public/natrules.png "NAT Rules"