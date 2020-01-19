---
layout: post
title: Building an Ethical Hacking Lab
date: 2017-09-05 21:55:23.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- InfoSec
tags:
- infosec
- kalilinux
- pentest
meta:
  _edit_last: '1'
  _oembed_76ceb39301905aa634d01d385c0352f5: "{{unknown}}"
  _oembed_a49f355eb2b656cc1d29f2b57a231437: "{{unknown}}"
  _oembed_34fa73533fa6581500ccebe2c2d43ee6: "{{unknown}}"
  _oembed_e787e3719214ef6bbb2b5267e60b72f2: "{{unknown}}"
  _publicize_twitter_user: "@GuilleRoman"
  _oembed_3a5b1a3851806c98d6eb7b632873e5f6: "{{unknown}}"
  _oembed_52dd5e8ff76d13861c1809aef4c770c1: "{{unknown}}"
  _wpas_mess: Building an Ethical Hacking Lab
  _wpas_done_all: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/building-an-ethical-hacking-lab/"
---
If you want to have a successful career in Information Security, building your own personal lab is essential. Not only it is likely to come up during technical interview questions but it will also be your personal geek playground. The lab is where you learn, discover, and have fun.

In this blog post I will show you how to set up a simple virtual lab with cheap hardware and free software that will help you get started in the amazing world of offensive security.

With the advancement in virtualization technologies building a lab is now easier than ever. All you really need is a computer, a virtualization platform, software, and most importantly, lots of enthusiasm.

![]({{ site.baseurl }}/assets/images/HackingLab1-1024x234.png)

<!--more-->

# Hardware

If you are reading this post on a laptop or desktop computer, chances are good that you can use it to host this lab. I typically recommend people to use a computer with at least the following minimum specs:

- i5 Quad Core CPU
- 16GB of RAM
- 250GB HDD, or even better, use an SSD if you can afford it.

Probably the single most important hardware component&nbsp;is memory. VMs are very memory hungry, so don't go below 16GB if you want to run multiple VMs at the same time.

# Virtualization platform

When it comes to choosing a virtualization platform you have several options. I won’t go into a detailed review of each platform here but I encourage you to do some research on your own. For this post, I will be using [Oracle’s VirtualBox](https://www.virtualbox.org/) for three main reasons: it's very simple to use, it's available on all major platforms (Windows, Mac, and Linux), and it's free.

Installing VirtualBox is pretty straight forward so I won't cover the installation steps in this post. Check out the official documentation [here](https://www.virtualbox.org/manual/ch02.html).

# General considerations

Arguably the most important thing to keep in mind when building a hacking lab is **network isolation**. It is really important to make sure that:

1. Your vulnerable VMs are not exposed to the internet.
2. The attacker VM cannot launch attacks against unintended targets. 

Thus, I always suggest to use an **Internal Network Adapter** (more information about network modes [here](https://www.virtualbox.org/manual/ch06.html)) to ensure that your VMs can see each other but cannot communicate with the outside world. This is vital since penetration testing can be a very destructive process. Tools such as [nmap](https://nmap.org/)or [metasploit](https://www.rapid7.com/products/metasploit/)&nbsp;can easily bring an entire network down.

You can use the VirtualBox command below to create a DHCP server for you internal network. In my case the network name is "pentest\_net" and the IP range is 10.10.10.2-10.10.10.12:

```
VBoxManage.exe dhcpserver add --netname "pentest\_net" -ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.2 --upperip 10.10.10.12 --enable
```

Some VMs might also need a NAT adapter to be able to download software and/or updates from the internet. I strongly suggest you disable the NAT adapter by default, and only enable it when strictly required.

# Pentest VM

If you are new to offensive security you should check out one of the pen test distributions available online for free. These Linux-based distributions are great for beginners since they come with a bunch of pre-installed software and tools. Without a doubt the most popular one is **Kali Linux** , and for that same reason is the one that we will be using today.

First, download the latest version of Kali here:

[https://www.kali.org/downloads/](https://www.kali.org/downloads/)

Then, run the following command to register the VM:

```
VBoxManage.exe createvm --name KaliLinux --ostype "Debian\_64" --register
```

Give it 2GB of memory:

```
VBoxManage.exe modifyvm KaliLinux --memory 2048
```

Create an empty hard driver file in VDI format:

```
VBoxManage.exe createhd --filename '.\VirtualBox VMs\KaliLinux\KaliLinux.vdi' --size 18000 --format VDI
```

Add a SATA controller for your HDD:

```
VBoxManage.exe storagectl "KaliLinux" --name "SATA Controller" --add sata
```

Attach the VDI file:

```
VBoxManage.exe storageattach KaliLinux --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium '.\VirtualBox VMs\KaliLinux\KaliLinux.vdi'
```

Add an IDE controller for your ISO file:

```
VBoxManage.exe storagectl "KaliLinux" --name "IDE Controller" --add ide
```

Attach the ISO file:

```
VBoxManage.exe storageattach KaliLinux --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium kali-linux-2017.1-amd64.iso
```

By default, all VMs are created with a NAT adapter. This adapter will allow your VM to communicate with the internet. You will need this adapter for installing/updating software in your Kali Linux VM. However, we will also need a second Internal Network adapter to allow your VM to connect to the "pentes\_net" network created earlier:

```
VBoxManage.exe modifyvm KaliLinux --nic2 intnet --intnet2 "pentest\_net"
```

Start you VM from the GUI, select install, and follow the installation instructions:

Once the installation process is done, open up a terminal in Kali Linux and type "ifconfig" to see the list of available network interfaces. You should see something like this:

```
eth0: flags=4163\<UP,BROADCAST,RUNNING,MULTICAST\> mtu 1500 inet 10.0.2.15 netmask 255.255.255.0 broadcast 10.0.2.255 inet6 fe80::a00:27ff:fecb:23ff prefixlen 64 scopeid 0x20\<link\> ether 08:00:27:cb:23:ff txqueuelen 1000 (Ethernet) RX packets 5 bytes 1520 (1.4 KiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 23 bytes 2189 (2.1 KiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 eth1: flags=4163\<UP,BROADCAST,RUNNING,MULTICAST\> mtu 1500 ether 08:00:27:f5:ea:ab txqueuelen 1000 (Ethernet) RX packets 0 bytes 0 (0.0 B) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 0 bytes 0 (0.0 B) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 lo: flags=73\<UP,LOOPBACK,RUNNING\> mtu 65536 inet 127.0.0.1 netmask 255.0.0.0 inet6 ::1 prefixlen 128 scopeid 0x10\<host\> loop txqueuelen 1 (Local Loopback) RX packets 20 bytes 1116 (1.0 KiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 20 bytes 1116 (1.0 KiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

Notice that we have two ethernet interfaces (eth0 and eth1). The first is attached to the NAT adapter, while the second one should be attached to the Internal Network adapter.

Run the following commands to enable eth0 so that we can connect to the internet:

```
$ ifconfig eth1 down $ ifconfig eth0 down $ ifconfig eth0 up
```

Then, run the following commands to update your packages and installed the guest additions:

```
$ apt -y update & apt -y dist-upgrade $ apt -y install virtualbox-guest-x11
```

This will take some time but make sure you disable eth0 and enable eth1 as soon as the installation is done:

```
$ ifconfig eth0 down $ ifconfig eth1 up
```

If you run "ifconfig" again you should only see eth1 and the loopback interface:

```
eth1: flags=4163\<UP,BROADCAST,RUNNING,MULTICAST\> mtu 1500 inet 10.10.10.2 netmask 255.255.255.0 broadcast 10.10.10.255 inet6 fe80::a00:27ff:fef5:eaab prefixlen 64 scopeid 0x20\<link\> ether 08:00:27:f5:ea:ab txqueuelen 1000 (Ethernet) RX packets 2 bytes 1180 (1.1 KiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 8 bytes 1184 (1.1 KiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 lo: flags=73\<UP,LOOPBACK,RUNNING\> mtu 65536 inet 127.0.0.1 netmask 255.0.0.0 inet6 ::1 prefixlen 128 scopeid 0x10\<host\> loop txqueuelen 1 (Local Loopback) RX packets 20 bytes 1116 (1.0 KiB) RX errors 0 dropped 0 overruns 0 frame 0 TX packets 20 bytes 1116 (1.0 KiB) TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

# Vulnerable VMs

Once you have your attacker machine setup you’ll need some victims. Though I strongly recommend you to play around with a commercial OS such as Windows (preferably unpatched versions of Windows XP or Windows 7), for this post we will use intentionally vulnerable VMs that were specially designed for learning.

We will install and configure two target VMs:

- **Metasploitable 2** : Linux VMs which was created in an intentionally insecure manner.
- CentOS minimal running **WebGoat** , which is a java-based web application great for learning Web Application Pen Testing.

**Metasploitable 2**

Download the Metasploitable 2 files from this link:

[https://sourceforge.net/projects/metasploitable/files/Metasploitable2/](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)

Notice that the zip file already contains a VirtualBox image in vmdk format (_metasploitable-linux-2.0.0\Metasploitable2-Linux\Metasploitable.vmdk_), so all we need to do is attach it to the SATA controller:

```
VBoxManage.exe createvm --name Metasploitable2 --ostype "Ubuntu\_64" --register VBoxManage.exe modifyvm Metasploitable2 --memory 1024 VBoxManage.exe storagectl "Metasploitable2" --name "SATA Controller" --add sata VBoxManage.exe storageattach "Metasploitable2" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium '.\VirtualBox VMs\Metasploitable2\Metasploitable.vmdk' VBoxManage.exe modifyvm Metasploitable2 --nic1 intnet --intnet1 "pentest\_net”
```

**WebGoat**

WebGoat is a just Java-based web application, so the first thing we need is an operating system.&nbsp;Download CentOS minimal from here:

[https://www.centos.org/download/](https://www.centos.org/download/)

And use these commands to create the VM:

```
VBoxManage.exe createvm --name WebGoat2 --ostype "RedHat\_64" --register VBoxManage.exe modifyvm WebGoat2 --memory 512 VBoxManage.exe modifyvm WebGoat2 --nic2 intnet --intnet2 "pentest\_net" VBoxManage.exe storagectl "WebGoat2" --name "SATA Controller" --add sata VBoxManage.exe createhd --filename '.\VirtualBox VMs\WebGoat2\WebGoat2.vdi' --size 18000 --format VDI VBoxManage.exe storageattach "WebGoat2" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium '.\VirtualBox VMs\WebGoat2\WebGoat2.vdi' VBoxManage.exe storagectl "WebGoat2" --name "IDE Controller" --add ide VBoxManage.exe storageattach "WebGoat2" --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium CentOS-7-x86\_64-Minimal-1511.iso
```

Start the VM from the GUI and follow the installation instructions.

Once the VM is up and running we need to install docker in order to run the WebGoat container. Run this command from the CentOS terminal to add the official Docker repository, download the latest version of Docker, and install it:

```
$ curl -fsSL https://get.docker.com/ | sh
```

Start the Docker daemon when the installation is done:

```
$ sudo systemctl start docker
```

Verify that it's running:

```
$ sudo systemctl status docker
```

Lastly, make sure it starts at boot time:

```
$ sudo systemctl enable docker
```

Now run the following commands to download the container image and run it:

```
$ docker pull webgoat/webgoat-8.0 $ docker run -d -p 8080:8080 webgoat/webgoat-8.0
```

At this point the WebGoat application should be running and listening on port 8080 inside your CentOS VM.

You don't need the NAT adapter anymore so feel free to remove it or disable it. Though we want to make sure that the VM can connect to our "pentest\_net" network.

Edit the following file using vi:

```
$ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

And set the following property at the end:

```
ONBOOT=yes
```

Finally restart the network service:

```
$ service network restart
```

## Validation

If everything worked as expected you should now be able to ping the target VMs from Kali. Connect to your Kali VM and do a ping sweep using [fping](https://fping.org/) to discover their IPs:

```
$ fping -a -g&nbsp;10.10.10.1/24
```

The hosts.txt file should contain three IP (you might get different IPs in your environment):

```
10.10.10.2 10.10.10.3 10.10.10.4
```

# Where to go from here

Now it’s time to try hacking into your VMs!&nbsp;

The following Metasploitable tutorials are a good place to start playing:

[Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/)

[Metasploitable 2 Exploitatability Guide](https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide)

The&nbsp;[WebGoat project](https://www.owasp.org/index.php/Category:OWASP_WebGoat_Project)&nbsp;has some useful tutorials.

Finally, the following book is also a great introduction to penetration testing:

[The Basics of Hacking and Penetration Testing](https://www.amazon.com/Basics-Hacking-Penetration-Testing-Second/dp/0124116442)

