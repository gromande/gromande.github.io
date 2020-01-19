---
layout: post
title: 'Follow up: Tomcat HA Cluster Automation using Ansible and Vagrant'
date: 2016-03-18 22:16:16.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Automation
tags:
- ansible
- haproxy
- tomcat
- vagrant
meta:
  _edit_last: '1'
  _jetpack_dont_email_post_to_subs: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/follow-up-tomcat-ha-cluster-automation-using-ansible-and-vagrant/"
---
In one of [my previous posts](http://guillermo-roman.com/tomcat-ha-cluster-automation-using-ansible/) I presented a solution&nbsp;to automate a&nbsp;Tomcat cluster installation using Ansible.&nbsp;One of the assumptions I made at the time was that the actual machines (physical or virtual) were already running and that Ansible could access them via SSH. Well, that might be true for a real production-like environment but what if you wanted to test this out on your local machine? Wouldn't it be cool to spin up&nbsp;some virtual machines first and&nbsp;then use Ansible to provision them with the click&nbsp;of a button? [Vagrant](https://www.vagrantup.com/) to the rescue!

Vagrant's motto is _"Development environments made easy"_ and in this post I plan to prove that&nbsp;and show how we can improve&nbsp;our existing &nbsp;[sample project](https://github.com/gromande/ansible-tomcat-cluster). I am not going to spend time going through the installation steps since I believe the&nbsp;official [Vagrant&nbsp;installation guide](https://www.vagrantup.com/docs/installation/) is pretty clear already.

The code for the sample project can be downloaded from [here](https://github.com/gromande/ansible-tomcat-cluster).

<!--more-->

# Prerequisites

To be able to run the project you'll need to install the following software:

- [Ansbile](http://docs.ansible.com/ansible/intro_installation.html#getting-ansible)
- [Vagrant](https://www.vagrantup.com/downloads.html)
- [Virtual Box](https://www.virtualbox.org/wiki/Downloads) **\***

**\*** This is needed since Vagrant doesn't provide the virtualization environment. Vagrant also supports other virtualization providers such as VMWare or AWS but VirtualBox is the default free option...and we like free. You can read more about virtualization providers&nbsp;[here](https://www.vagrantup.com/docs/providers/).

# Project Goal

Before we dive into the&nbsp;sample project let me give you a high level overview of what our goal is. In a nutshell, we want to use Vagrant to spin up&nbsp;a bunch of virtual machines and then run an&nbsp;Ansible playbook to provision the VMs with the software we want.

We only need three VMs&nbsp;in our case: two for the Tomcat containers and another one for the HAProxy.

The sequence of events&nbsp;will look like this:

![vagrant_tomcat_ha]({{ site.baseurl }}/assets/images/vagrant_tomcat_ha-1024x413.png)

# Sample Project

If you download the project you will see that the only major addition is&nbsp;a new folder called "vagrant" that contains two files: _Vagrantfile_ and _nodes.json_.

If you are already familiar with the standard Vagrant configuration you will know that all the meta-data is kept in a file called [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/). Our Vagrantfile looks like this:

```
#Parse nodes config file nodes = (JSON.parse(File.read("nodes.json"))) # Vagrantfile API/syntax version. Don't touch unless you know what you're doing! VAGRANTFILE\_API\_VERSION = "2" Vagrant.configure(VAGRANTFILE\_API\_VERSION) do |config| config.vm.box = "centos/7" nodes.each\_with\_index do |node, node\_index| node\_name = node[':node\_name'] # name of node config.vm.define node\_name do |config| #Use default SSH private key for every node config.ssh.insert\_key = false config.vm.hostname = node[':hostname'] config.vm.network :private\_network, ip: node[':ip'] config.vm.provider :virtualbox do |vb| vb.customize ["modifyvm", :id, "--memory", node[':memory']] vb.customize ["modifyvm", :id, "--name", node\_name] end #Run provisioner at the end, when all nodes are up if node\_index == (nodes.size - 1) config.vm.provision "ansible" do |ansible| ansible.limit = "all" ansible.playbook = "../site.yml" ansible.inventory\_path = "../hosts" #Disable host key checking so that the hosts don't get added to #our known\_hosts file ansible.host\_key\_checking = false end end end end end
```

The first thing you'll notice&nbsp;is that it creates multiple virtual machines based on the contents of another file called&nbsp;nodes.json. Why bother creating another file? The idea is that you can keep&nbsp;all the VM specific settings such as the VM name, IP Address, hostname, and memory capacity into&nbsp;the json file&nbsp;whereas the Vagrantfile remains pretty much static, even if you decide to add more VMs later on. It also provides and easier way of reading and managing your VM settings. This is what my nodes.json file looks like:

```
[{ ":node\_name": "ansible", ":ip": "192.168.82.2", ":hostname": "ansible.groman.com", ":memory": 512 }, { ":node\_name": "ansible1", ":ip": "192.168.82.3", ":hostname": "ansible1.groman.com", ":memory": 1024 }, { ":node\_name": "ansible2", ":ip": "192.168.82.4", ":hostname": "ansible1.groman.com", ":memory": 1024 }]
```

We also prevent Vagrant from automatically generating a new random SSH key pair for each VM. Instead we make sure that Vagrant provisions the VMs with the same default SSH public key.&nbsp;This would make things a little easier&nbsp;for us since we can use the same key to connect to all the VMs. The default private key is located&nbsp;inside you vagrant.d directory and it's called "insecure\_private\_key". In a Unix-like system the default location should be:

```
~/.vagrant.d/insecure\_private\_key
```

As you can probably imagine this&nbsp;key should never be used&nbsp;in a real environment (in which case you should use you own key pairs) but it will do the trick for a development environment like the one we are trying to build.

You might also notice&nbsp;that Vagrant will assign&nbsp;a set of private static IPs to our VMs so that we can easily access them just like if they were running on a remote location. Now it's a good time to edit your hosts file and map those IP addresses to the corresponding FQDNs as defined in our Ansible's host file:

```
192.168.82.2 ansible ansible.groman.com 192.168.82.3 ansible1 ansible1.groman.com 192.168.82.4 ansible2 ansible2.groman.com
```

Vagrant will then run the Ansible playbook after&nbsp;all the VMs are functioning. Sometimes you can chose to provision the VMs one at a time but our current playbook requires us to run it only after all the VMs are up.

And that's it! Just run the following command from inside the vagrant folder to create and provision your VMs:

```
vagrant up
```

The process might take some time to finish but at the end you should be able to access you application by going to this url:

[http://ansible.groman.com/snoop](http://ansible.groman.com/snoop)

Feel free to ask questions and leave some feedback.  
Happy automation!

