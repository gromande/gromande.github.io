---
layout: post
title: 'Network Security - Part 7: Remote Access & VPNs'
date: 2018-07-03 20:00:39.000000000 -05:00
password: ''
categories:
- InfoSec
- Network Security
tags:
- infosec
- pfsense
- vpn
---
In today's world, It's very common for companies to have multiple offices around the world and to allow employees to work remotely. This desired for mobility must&nbsp;be taken into consideration when designing your network to ensure people can access company resources from anywhere, at anytime, and with a high degree of security. The most common remote access technology used today are Virtual Private Networks (VPNs).

At a high level, a VPN allows users to access corporate assets (email servers, web applications, shared folders, etc...) remotely, just like they would if they&nbsp;were connected to the internal network. There are two main types of VPNs.

Site-to-Site or Point-to-Point VPNs connect two or more networks:

![]({{ site.baseurl }}/assets/images/VPN-Site-to-Site.jpg)

Whereas Remote Access VPNs allow users to connect to a site from a remote location (e.g. home office):

![]({{ site.baseurl }}/assets/images/VPN-Remote-Access.jpg)

Though the concept is the same, the underlying technology varies greatly depending on which VPN type you want to implement. In this post, we will discuss how to configure our _pfSense_ router to serve as a Remote Access VPN server using a popular software called OpenVPN.

<!--more-->

# Create Certificates

Two of the most important services that VPNs provide are confidentiality (making sure the communication is encrypted to protect data in transit against eavesdropping attacks) and authentication (ensure that remote users are who they claim to be before granting them access to the internal network).

Confidentiality is achieved through the use of encryption and digital certificates. Authentication can also involve&nbsp;certificates (i.e. certificate-based authentication) but that is not the only way. Most VPN servers support standard username and password credentials as well. Most of them even integrate with you standard corporate user repository such as Active Directory or LDAP.

Let's start by creating a _Certification Authority (CA)_ that we will use to create certificates certificates. Log into the pfSense admin console and go to _System \> Certificate Manager \> CAs_ and create a new CA with the following attributes (or similar):

![]({{ site.baseurl }}/assets/images/ca.png)

Then go to _System \> Certificate Manager \> Certificates_ and create a new certificate for our OpenVPN server:

![]({{ site.baseurl }}/assets/images/server_cert.png)

# OpenVPN Server

pfSense comes already pre-installed with OpenVPN. Go to _VPN \> OpenVPN \> Wizard_ and start creating a new _Local User Access_ VPN server:

![]({{ site.baseurl }}/assets/images/wizard1.png)

Choose&nbsp;_NSLabOpenVPNCA_ as the CA and NSLabOpenVPNServer as the server certificate:

![]({{ site.baseurl }}/assets/images/wizard2.png)

![]({{ site.baseurl }}/assets/images/wizard2b.png)

In the next page, use the default values for the _Generic Settings_ and _Cryptographic Settings_ but update the _Tunnel Settings_ to make sure remote users connecting through the VPN can access the DMZ network (172.16.2.0/24):

![]({{ site.baseurl }}/assets/images/wizard3.png)

Update the DNS and NTP client settings to point to the internal gateway (172.16.2.1):

![]({{ site.baseurl }}/assets/images/wizard4.png)

Finally, let the wizard create a new WAN rule to allow clients to connect to the OpenVPN port (1194) from the internet:

![]({{ site.baseurl }}/assets/images/wizard5.png)

Your OpenVPN server is now configured. To verify the new firewall rules go to _Firewall \> Rules \> WAN_ and make sure that the OpenVPN rule shows up above the "Default deny any" rule (\*Move up the rule if it shows up at the bottom of the list):

![]({{ site.baseurl }}/assets/images/WANRules.png)

A new interface called "_OpenVPN_" should've also been created:

![]({{ site.baseurl }}/assets/images/OpenVPNRules.png)

You can add rules to this&nbsp;OpenVPN interface&nbsp;if you want to filter the traffic entering the firewall across the VPN.

# OpenVPN Client

Now that our OpenVPN server is running and accessible from the internet we need to configure the OpenVPN client, but first, we need to create some test users. Go to _System \> User Manager \> Users_ and create a new user:

![]({{ site.baseurl }}/assets/images/user.png)

Instead of creating the client configuration file from scratch we are going to install the _OpenVPN Client Export_ package available for pfSense. Navigate to _System \> Package Manager \> Available Packages,&nbsp;_search for "_openvpn-client-export_", and install the package:

![]({{ site.baseurl }}/assets/images/client-export.png)

Then, go to _VPN \> OpenVPN \> Client Export_ and click on “_Inline Configurations - Most Clients_":

![]({{ site.baseurl }}/assets/images/client-config.png)

Transfer the _.ovpn_ file to the nsClient VM&nbsp;and run the following command to install the OpenVPN client from the Ubuntu repositories:

```
$ sudo apt-get install openvpn
```

Verify that you are connected to the external network (192.168.15.0/24 in my case):

![]({{ site.baseurl }}/assets/images/ifconfig_1.png)

\*Run the following commands to switch to the external network if you are connected to the internal network:

```
$ sudo ifdown enp0s3 $ sudo ifup enp0s8
```

Verify also that there are only two IP routes in your internal routing table:

![]({{ site.baseurl }}/assets/images/route-1.png)

Next, open a different terminal and run the following command to connect to the OpenVPN server:

```
$ sudo openvpn --config \<path-to-ovpn-file\>
```

When prompted, provide the credentials of the test user you created earlier. If everything works fine you should see the following output:

![]({{ site.baseurl }}/assets/images/client-connection.png)

You should also see a new _tun_ interface (all packets with destination 172.16.2.X will be routed through this interface):

![]({{ site.baseurl }}/assets/images/ifconfig_2.png)

As well as a new IP route for the 172.16.2.0/24 network:

![]({{ site.baseurl }}/assets/images/route-2.png)

Finally, make sure you can connect to the web server using its internal IP address (172.16.2.2):

![]({{ site.baseurl }}/assets/images/browser-1024x449.png)

And that's it for today!

I hope this post helped you understand how VPNs work a little&nbsp;better. Stay tuned for the next chapter on network security!

