---
layout: post
title:  "Enable SSH in VirtualBox VM"
date:   2016-03-31 11:46:00 +0900
categories: ssh virtualbox
---

To validate a Python script I wrote using password SSH, I setup the SSH server in a Virtualbox VM. Following are the steps:


Install SSH server 
---------------
* In Virtualbox Manager, choose the VM, Settings->Network, attached to ```NAT``` (to connect to Internet)
* start the VM, execute the following command
{% highlight shell %}
sudo apt-get install openssh-server
{% endhighlight %}
* shutdown the server

Test SSH
------------------
* In Virtualbox Manager, choose the VM, Settings->Network, attached to ```Bridged Adapter```
* start the VM, find the VMs IP
{% highlight shell %}
ifconfig
{% endhighlight %}
* In the host, execute the following command to login the guest VM
{% highlight shell %}
ssh <guest user id>@<VMs IP>
{% endhighlight %}
and then give the password to login.


Setup passwordless SSH
------------------
* create the ssh directory in the VM. In the host, 
{% highlight shell %}
ssh <guest user id>@<VMs IP> 'mkdir -p .ssh'
{% endhighlight %}
* generate the SSH public/private key (if not generated yet). In the host
{% highlight shell %}
ssh-keygen -t rsa
{% endhighlight %}
* send the public key to the VM
{% highlight shell %}
cat .ssh/id_rsa.pub | ssh <guest user id>@<VMs IP> 'cat >> .ssh/authorized_keys'
{% endhighlight %}

SSH it
-----------------
* In the host, 
{% highlight shell %}
ssh <guest user id>@<VMs IP>
{% endhighlight %}



---

### Tested on
* VirtualBox Version 5.0.16 
* ubuntu 14.04 LTS as guest
* OS X El Capitan version 10.11.3 as host

### References:
http://stackoverflow.com/a/10532299/1216961
https://www.youtube.com/watch?v=5BsShkcweIs
http://www.linuxproblem.org/art_9.html




