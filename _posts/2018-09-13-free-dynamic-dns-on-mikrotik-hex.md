---
layout: post
comments: true
title: Setting up free dynamic DNS on your Mikrotik Hex
---

# Intro

If you got inspired by the [previous post](https://mlboyd.xyz/2018/07/06/mikrotik-hex-router-centurylink-pppoe/) and setup your nice, new Mikrotik Hex router, you might want to configure dynamic DNS on it as well.

The advantage of dynamic DNS is that you get a nice hostname to ssh to/reference even though the underlying IP address might change. I've had Centurylink fiber as my ISP for about 6 months, and my IP address has changed a couple of times. Nothing to crazy, but it's nice to just have a hostname and not have to worry about the ip address.

The steps below are pretty simple and should only take 5-10 minutes to setup. With this setup, every two hours the router will see if the IP on the interface has changed. If it has, the router will call out to FreeDNS to update DDNS.

# Configuration

## Step 1 - Get a FreeDNS account and subdomain
Go to [http://freedns.afraid.org/](http://freedns.afraid.org/) and register an account. Then, go to [http://freedns.afraid.org/subdomain/edit.php](http://freedns.afraid.org/subdomain/edit.php) and add your desired subdomain with an A type DNS entry.
"Destination" is your current IP address.

## Step 2 - Grab your DDNS update token
Go to [http://freedns.afraid.org/dynamic/](http://freedns.afraid.org/dynamic/) and click the "Direct URL" entry. Grab your update token. **It's everything after "update.php?"**

## Step 3 - Create the DDNS update script on the router
Open up your Mikrotik terminal and copy-pasta the following script. This script will do the DDNS update if needed. 
(Obviously, change the script to the correct UNIQUE key required to update your Dynamic DNS service)

```
/system script
add name="ddnsupdate" policy=ftp,password,policy,read,reboot,sensitive,romon,sniff,test,write source={/tool fetch address="freedns.afraid.org" host="freedns.afraid.org" mode=http src-path="dynamic/update.php\?YOURTOKENHERE" keep-result=no
}
```

## Step 4 - Create an IP check script to see if DDNS update is needed
To prevent spamming the FreeDNS servers to see if we need to update our IP address, we can do it locally on the router with the following script:

```
/system script
add name="checkcurrentip" policy=ftp,password,policy,read,reboot,sensitive,romon,sniff,test,write source={
:global currentIP;
:local newIP [/ip address get [find interface="pppoe-out1"] address];

:if ($newIP != $currentIP) do={
    :log info "ip address $currentIP changed to $newIP";
    /system script run ddnsupdate; 
    :set currentIP $newIP; 
    }  else={ 
    :log info "No change of IP"; 
    }
    }
```

## Step 5 - Setup scheduler to periodically check our DDNS setup

```
/system scheduler add name="Check IP and update DDNS" on-event="checkcurrentip" interval=2h
```