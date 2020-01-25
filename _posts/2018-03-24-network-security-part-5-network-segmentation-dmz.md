---
layout: post
title: 'Network Security - Part 5: Network Segmentation (DMZ)'
date: 2018-03-24 22:51:52.000000000 -05:00
password: ''
categories:
- Network Security
tags:
- firewall
- infosec
- network segmentation
- pfsense
---
In [Part 4]({{ site.baseurl }}/network-security-part-4-host-firewalls/) of this series, I talked about the importance of host-based firewalls to protect against lateral movement between host co-located on the same network. Though that was a great improvement, there are still some critical design issues with our network architecture:

![]({{ site.baseurl }}/assets/images/image001.png)

There is only one internal subnet (green network) where all devices live. Servers, workstations, printers, smart phones...everything is connected to the same network. Our network is completely "flat" making it more difficult to manage and secure.

Segmentation is the process of breaking down a network into smaller zones or subnets. This approach provides many benefits including:

- Greater **management and** &nbsp; **troubleshooting** capabilities for network related issues.
- **Better performance** by reducing the amount of local traffic.
- **Improved security&nbsp;** by providing more visibility into traffic flowing across subnets and allowing architects to apply different security controls based on the devices connected to each zone.

Today, we are going to segment our network by creating a [Demilitarized Zone (DMZ)](https://en.wikipedia.org/wiki/DMZ_(computing)) that will contain all devices that should be accessible from the internet, such as our web server. This will allow us to separate devices that are more likely to get compromised from the rest of the network.

<!--more-->

# DMZ Network

* * *

We need to add a second internal network (orange) to our environment:

![]({{ site.baseurl }}/assets/images/network-security-essentials.png)

In reality, we will rename _nslInternal_ to _nslDMZ_ and create a new network (172.16.3.0/24) with the same name (_nslInternal_).

Stop all the VMs and edit the network settings for the nslpfSense VM. Rename Adapter 3 to "_nslDMZ_":

![]({{ site.baseurl }}/assets/images/pfsense-nslinternal-1024x704.png)

Enable Adapter 4 with the following configuration:

![]({{ site.baseurl }}/assets/images/pfsense-nsDMZ-1024x703.png)

\*To avoid having to reconfigure all the firewall rules I decided to attach the DMZ to the existing Adapter 3 and move _nslInternal_ to the new Adapter 4.

VirtualBox will generate different MAC addresses for your environment but at the end your should have four network interfaces:

| VirtualBox Network | pfSense Interface | MAC Address | IP Address |
| --- | --- | --- | --- |
| nslInternet | em0 | 08:00:27:36:70:56 | 192.168.15.4 |
| VirtualBox Host-Only Ethernet Adapter | em1 | 08:00:27:42:7a:36 | 172.16.1.1 |
| nslDMZ | em2 | 08:00:27:ed:b4:1e | 172.16.2.1 |
| nslInternal | em3 | 08:00:27:57:3c:dc | 172.16.3.1 |

Start pfSense in headless mode and log into the admin console. Once logged in, go to _Interfaces \> Assignments:_

![]({{ site.baseurl }}/assets/images/pfsense-assignments-1024x345.png)

The new interface (em3) should already be available. Click Add and save your changes.

SSH into the pfSense box and select option 2 "_Set Interface(s) IP address_", and then option 4 "_OPT2 (em3)_":

![]({{ site.baseurl }}/assets/images/pfsense-menu-1024x689.png)  
You will be asked a series of questions to configure the new interface. Enter the following answers:

| Enter the new LAN IPv4 address | 172.16.3.1 |
| Enter the new LAN IPv4 subnet bit count | 24 |
| Enter the IPv4 upstream gateway address | Leave Blank |
| Enter the new LAN IPv6 address | Leave Blank |
| Do you want to enable the DHCP server on LAN? | yes |
| Enter the start address of the IPv4 client address range | 172.16.3.12 |
| Enter the end address of the IPv4 client address range | 172.16.3.254 |
| Do you want to revert to HTTP as the webConfigurator protocol? | no |

# Network Services

* * *

Let's enable DNS and NTP for the _nslInternal_ network. From the pfSense admin dashboard go to _Services \> DNS Resolver_, highlight OPT2 and save the changes:

![]({{ site.baseurl }}/assets/images/dns-settings-1024x379.png)

Then, go to _Services \> NTP Server_ and enable NTP for the OPT2 interface:

![]({{ site.baseurl }}/assets/images/ntp-server-1024x371.png)

Finally, go to _Service \> DHCP Server \> OPT2_ and add 172.16.3.1 as the primary NTP server:

![]({{ site.baseurl }}/assets/images/ntp-settings-1024x220.png)

# Firewall Rules

* * *

There are currently no firewall rules defined for our new OPT2 interface:

![]({{ site.baseurl }}/assets/images/opt2-rules-empty-1024x324.png)

We just need to duplicate the rule set that we previously defined for OPT1. Make sure to use _OPT2 net_ and _OPT2 address_ instead of _OPT1 net_ and _OPT1 address_ respectively. In addition, we need to add a rule to explicitly allow access to the web server from the internal network:

![]({{ site.baseurl }}/assets/images/opt2-rules-1024x414.png)

Let's also add a new rule to our management interface (_LAN_) to allow access to the _OPT2_ network:

![]({{ site.baseurl }}/assets/images/lan-rules-1024x697.png)

Make sure it shows up right above the "_Deny access to any other RFC1918 network_" rule.

We don't have to make any changes to OPT1. However, since OPT1 is now our DMZ we could be a little bit more restrictive. We could decide to block any outbound traffic from the DMZ network to the internet.&nbsp;&nbsp;After all, our web server doesn’t need internet access, especially if you configure your servers to update/install software from a local repository.

You might be wondering why going through so much pain just to prevent servers in the DMZ from accessing the internet. Malware often communicates to external command and control servers over regular outbound traffic. The potential for an attacker to cause damage will be greatly reduced if internet access is denied. In addition, blocking outbound traffic might also prevent data exfiltration.

Since I don't have the infrastructure in place to install software packages from a local repository I won't block outbound traffic in my environment but it's something you should consider when designing a real DMZ network.

# Let's Try It!

* * *

Now that we have our DMZ setup it's time to update the other VMs and test the new configuration.

Move _nslWebServer_&nbsp;to the _nslDMZ_ network:

![]({{ site.baseurl }}/assets/images/webserver-nsDMZ-1024x701.png)

Start the VM in headless mode and verify that you can still SSH into it:

![]({{ site.baseurl }}/assets/images/webserver-ssh-1024x410.png)

Once inside the VM, run the following commands to verify the DHCP settings:

![]({{ site.baseurl }}/assets/images/webserver-ifconfig-1024x256.png)

DNS settings:

![]({{ site.baseurl }}/assets/images/webserver-dns-1024x250.png)

And NTP settings:

![]({{ site.baseurl }}/assets/images/webserver-ntp-1024x383.png)

Similarly, update _nslMonServer's_ Adapter 2 and verify the network settings:

![]({{ site.baseurl }}/assets/images/monserver-nsDMZ-1024x704.png)

Finally, start the nslClient VM and make sure you can still connect to the internet:

![]({{ site.baseurl }}/assets/images/client-internet-access-1024x445.png)

And that you can access the web server internally:

![]({{ site.baseurl }}/assets/images/client-webserver-access-1024x464.png)

# Final Thoughts

* * *

Segregation can also be achieved by means other than firewalls such as [VLANs](https://en.wikipedia.org/wiki/Virtual_LAN), [Software-Defined Networking](https://en.wikipedia.org/wiki/Software-defined_networking), or a combination of the three. pfSense provides support for VLANs and I encourage you to play around with them.

Our network is looking much better&nbsp;but there is still a lot of work to be done so stay tuned for my next post!

