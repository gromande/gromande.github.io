---
layout: post
title: 'Network Security - Part 4: Host Firewalls'
date: 2018-02-19 02:02:15.000000000 -06:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- InfoSec
- Network Security
tags:
- firewall
- iptables
- linux
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
permalink: "/network-security-part-4-host-firewalls/"
---
So you spent thousands of dollars on a [next-gen firewall](https://en.wikipedia.org/wiki/Next-Generation_Firewall).That means you are secure, right? Even though it is obvious that the answer to this question is no, many people believed otherwise not that long ago. Just because you spend money on security doesn't mean that you'll be secure. Network firewalls provide one layer of protection for your environment but they can be bypassed by experienced attackers, specially if the firewall is not configured properly. In fact, it's not a matter of "if", it's a matter of "when". What will you do when an attacker bypasses your firewall and compromises one of your endpoints?

A common technique attackers use after penetrating a network is called lateral movement.&nbsp;This technique allows attackers to propagate across multiple hosts that are co-located in the same subnet. For example, in a Windows environment, an attacker can start by compromising a user endpoint and then pivot laterally until it reaches a Domain Controller, giving them a stronger position to take over privilege accounts or exfiltrate data.

In this post, we will take a look at host-based firewalls and learn how they can be used to protect against lateral movement.

<!--more-->

# Host Firewalls with IPTables

[IPTables](https://en.wikipedia.org/wiki/Iptables) is a command-line tool that allows you to configure the rules provided by the Linux kernel firewall, also known as [Netfilter](https://en.wikipedia.org/wiki/Netfilter). Netfilter is implemented as a series of hooks (usually called insertion points or chains) that get invoked at different times throughout the Linux networking stack:

- Pre-routing:&nbsp;called when incoming packets arrive at the system but before routing decisions are made.
- Local Input:&nbsp;called after routing, but only if the packet is destined for a process running on the local computer.
- Forward: called when the packet is determined to be sent to another interface, or forwarded.
- Local Output:&nbsp;called when the packet comes from the local server, but it's destined for another system.
- Post-routing:&nbsp;called for all packets leaving the computer, regardless of destination.

![]({{ site.baseurl }}/assets/images/netfilter.png)

As you can probably guess, we are going to configure some of this insertion points to discard unwanted packets arriving at our web server. In particular, we are going to focus on these three chains:

- Forward: we are going disable forwarding completely.
- Output: allow all outgoing traffic.
- Input:
  - Accept HTTP traffic from any network.
  - Accept SSH traffic only from the hypervisor host.

With these rules in place, even if attackers compromise a user endpoint they will have a hard time trying to reach our web server.

# IPTables Rules

Let's jump right into IPTables and start creating rules for our web server.

SSH into nslWebServer (172.16.2.2) and run the following command to display the existing rules:

```
sudo iptables -L
```

All three chains should be empty at this point:

![]({{ site.baseurl }}/assets/images/iptables_default.png)

Run the following command to add a rule that allows SSH access from the management host:

```
sudo iptables -A INPUT -p tcp --dport 22 -s 172.16.1.2 -j ACCEPT
```

\*Note: creating this rule first ensures that SSH is always available even if we mess up the other rules.

Let's break down the command:

![]({{ site.baseurl }}/assets/images/iptables_rule.png)

It is usually a good idea to explicitly allow traffic on the loopback interface, especially if your web server needs to connect to services (e.g. databases, etc...) listening on “localhost”:

```
sudo iptables -I INPUT -i lo -j ACCEPT
```

Notice that in this case we used “_-I INPUT_” to insert the rule at the beginning of the chain rather than at the end ("_-A_" appends the rule at the end), and “_-i lo_” to specify the input interface instead of a single source address.

Don't forget that we want users to connect to our web server running on port 80. Let's create a rule to allow incoming HTTP traffic:

```
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

IPTables policies define the default behavior when the incoming packet doesn’t match any rule. By default, all policies are set to _ACCEPT_, but fortunately you can run the following commands to update them:

```
sudo iptables -P OUTPUT ACCEPT sudo iptables -P INPUT DROP sudo iptables -P FORWARD DROP
```

We can validate our rules at this point by logging into our nslClient VM and making sure that we can still connect to the web server's internal address (_http://172.16.2.2:80_):

![]({{ site.baseurl }}/assets/images/internal_web_server-1024x535.png)

Now, go back to the nslWebServer VM and run the following command:

```
curl http://www.google.com
```

Hhmm...the commands seem to hang until the request times out. Can you guess why?

The problem is that IPTables is blocking response packets coming from the outside. Here is how it works: the curl command sends packets to&nbsp;_www.google.com_ and expects a response packet back. However, this response is getting blocked by the host firewall.

Rest assured, we just need to explicitly allow response packets by adding the following rule:

```
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

The _-m_ flag stands for match, which allows IPTables to match packets based on the connection state (_--state_). This rule will accept incoming packets that are part of an&nbsp;_ESTABLISHED_ or _RELATED_&nbsp;connection. _RELATED_ is a connection type within IPTables that matches new connections that are related to already established connections.

You should now be able to connect to the internet and everything should be working as expected. Display the IPTables again to verify it contents:

![]({{ site.baseurl }}/assets/images/iptables_updated-1024x325.png)

# Persist Rules

By default, IPTables are flushed (removed) with every server restart. There are different ways you can persist IPTables rules. We are going to create a simple Bash script and use the Linux [_crontab_](https://en.wikipedia.org/wiki/Cron) to execute the script at boot time.

Log into nslWebServer and create a file containing all our IPTables commands:

```
#!/bin/sh IPT=/sbin/iptables # flush tables $IPT -F # policies $IPT -P OUTPUT ACCEPT $IPT -P INPUT DROP $IPT -P FORWARD DROP # allowed inputs $IPT -A INPUT -i lo -j ACCEPT $IPT -A INPUT -p tcp --dport 22 -s 172.16.1.2 -j ACCEPT $IPT -A INPUT -p tcp --dport 80 -j ACCEPT #allow responses $IPT -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

Run the following command to open the crontab editor:

```
sudo crontab -e
```

Add the following line at the end of the crontab file:

```
@reboot /opt/set-iptables.sh
```

\*Note: replace _/opt/set-iptables.sh&nbsp;_with your own&nbsp;script.

You can run the following command to display the crontab configuration:

```
sudo crontab -l
```

![]({{ site.baseurl }}/assets/images/crontab.png)

Restart the server and verify that the IPTables rules didn't get removed.

# That's it for today

IPTables is a very powerful tool and there is much more you can do with it. Go ahead and create your own rules!

Finally, I would like to mention that modern Windows Operating Systems come with [Windows Firewall](https://en.wikipedia.org/wiki/Windows_Firewall) enabled by default. Windows Firewall is very easy to configure and it can greatly improve your defenses and protect against lateral movement as well.

