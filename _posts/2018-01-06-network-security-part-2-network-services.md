---
layout: post
title: 'Network Security - Part 2: Network Services'
date: 2018-01-06 22:33:08.000000000 -06:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- InfoSec
- Network Security
tags:
- dhcp
- dns
- ntp
- pfsense
meta:
  _edit_last: '1'
  _publicize_twitter_user: "@GuilleRoman"
  _wpas_done_all: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/network-security-part-2-network-services/"
---
In [part 1](http://guillermo-roman.com/network-security-part-1-setting-up-your-environment/) of this series we created the environment that we will be using in future posts. At the core of the network architecture is our pfSense router. Before we can start exploring all the security features that pfSense provides we need to configure some basic network services for our LAN (management) and OPT1 (internal) networks.

<!--more-->

# DNS

* * *

The main purpose of the Domain Name System (DNS) protocol is to resolve human-readable host names to IP addresses.&nbsp;

[pfSense](https://doc.pfsense.org/index.php/Unbound_DNS_Resolver) comes pre-installed with [Unbound](https://unbound.net/), which is an open source DNS resolver. We are going to configure this DNS resolver to listen on both internal interfaces (LAN and OPT1) for DNS queries. These queries will then be forwarded to the upstream DNS servers defined under&nbsp;_System \> General Setup:_

![]({{ site.baseurl }}/assets/images/DNS_upstream-1024x283.png)

In my case, I am using&nbsp;Google's DNS as the primary server, and Layer3's DNS as secondary.

Then go to _Services \> DNS Resolver_ and select LAN, OPT1, and Localhost as the network interfaces the DNS resolver should listen on:

![]({{ site.baseurl }}/assets/images/DNS_resolver_interfaces-1024x178.png)

And select WAN as the outgoing network interface (i.e. interface the resolver should use to forward queries to the authoritative servers):

![]({{ site.baseurl }}/assets/images/DNS_resolver_out_interfaces-1024x152.png)

Save you changes.

# NTP

* * *

We also want pfSense to act as the Network Time Protocol server for our network. Having all the VMs in sync is critical if you want to do any kind of network traffic analysis or correlate events from multiple sources.

pfSense keeps its own clock in sync against remote NTP servers, acting as an NTP client itself. Go to&nbsp;_Services \> NTP_&nbsp;to enable the NTP server for both internal interfaces and configure the upstream NTP server pool:

![]({{ site.baseurl }}/assets/images/NTP_server_pool-1024x579.png)

I typically use the NTP servers from the [NTP pool project](http://www.pool.ntp.org/zone/north-america).

Save your changes.

# DHCP

* * *

The Dynamic Host Configuration Protocol&nbsp;automatically provides IP addresses and other related configuration information such as DNS and NTP settings to clients connected to the network.

By default, the pfSense DHCP server will select one IP address from the pool and assign it to the client making the DHCP request. This is fine for devices that do not need a static IP address such as our nslClient VM. However, DHCP also supports static IP mappings based on the client's MAC address. We will use this option to assign static IPs to our web and monitoring servers.

Go to _Service \> DHCP Server \> LAN_ to enable the DHCP server for the LAN interface:

![dhcp_enable]({{ site.baseurl }}/assets/images/dhcp_enable.png)

Configure the range of IP address leaving a few of them out of the range so that they can be used for static mappings:

![]({{ site.baseurl }}/assets/images/dhcp_range-1024x200.png)

Do not specify any DNS servers. By default, pfSense will use the interface's IP (172.16.1.1) if the DNS Resolver is enabled.

Enter the interface's IP address as the NTP server:

![]({{ site.baseurl }}/assets/images/dhcp_ntp-1024x276.png)

Finally, add a static mapping for the monitoring server (nslMonServer):

![dhcp_lan_mappings]({{ site.baseurl }}/assets/images/dhcp_lan_mappings-1024x117.png)

Save your changes.

Go to _Service \> DHCP Server \> OPT1_ to enable the DHCP server for the OPT1 interface.&nbsp;Configure 172.16.2.12 - 172.16.2.254 as the IP range, and 172.16.2.1 as the NTP server.

We need two static mappings for the OPT1 interface. One for the monitoring server, and another one for the web server:

![]({{ site.baseurl }}/assets/images/dhcp_opt1_mappings-1024x148.png)

Save your changes.

# Kicking the Tires

* * *

All our client VMs are running Ubuntu so we can use the same commands to validate our network settings.

By default, there are no firewall rules configured for the OPT1 interfaces, which will prevent our VMs from connecting to pfSense.

Go to _Firewall \> Rules \> OPT1_ and add the following rule:

![]({{ site.baseurl }}/assets/images/default_rule-1024x205.png)

This rule will allow any VM on the OPT 1 to connect to any destination which is not very secure. Firewall rules will be the topic of my next post so we can ignore this for now.

SSH into each VM and run the following command to force the machine to send a new DHCP request to the gateway:

```
sudo dhclient
```

Run "ifconfig" to make sure the network interface(s) is configured with a valid IP address:

![]({{ site.baseurl }}/assets/images/ifconfig-1024x541.png)

Use _nslookup_ to test the DNS configuration (make sure the query is sent to the pfSense address):

![]({{ site.baseurl }}/assets/images/nslookup.png)

Finally, use _systemd_ to verify the time sync settings:

```
systemctl status systemd-timesyncd
```

![]({{ site.baseurl }}/assets/images/timesync-1024x487.png)

If the output doesn't show a message like "_Synchronized to time server \<pfSense IP\>:123_" you might have to restart the timesync service first:

```
sudo systemctl restart systemd-timesyncd
```

* * *

This post should end all the prep work required to configure our environment. In my next post I will talk about firewall rules and network segmentation.

See you next time!

