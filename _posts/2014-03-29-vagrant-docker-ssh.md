---
layout: post
title: "vagrant docker ssh"
date: 2014-03-29 21:00
categories: vagrant docker
---

# Vagrant Docker

If you are build a saas, using VMs and management tools. You will find [vagrant](http://www.vagrantup.com) is useful for additional features.

But Virtual machines take too much time to load. Now there is a new trending called using
[docker](http://docker.io). Docker is written in [go](http://golang.org), if you haven't heard of, you should probably go to check it out. In this article I am going to run a docker container in vagrant virtual machine

# What is vagrant

Vagrant is a tool for building complete development environments. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases development/production parity, and makes the "works on my machine" excuse a relic of the past.

you can also think it as a VM without the GUI. At its core, Vagrant is a simple wrapper around Virtualbox/VMware.

A few interesting features:

* Boatloads of existing images, just check [Vagrantbox.es](http://www.vagrantbox.es) for example.
* Snapshot and package your current machine to a Vagrant box file (and, consequently, share it back).
* Ability to fine tune settings of the VM, including things like RAM, CPU, APIC…
* Vagrantfiles. This allows you to setup your box on init: installing packages, modifying configuration, moving code around…
* Integration with CM tools like Puppet, Chef and Ansible.

# Install vagrant

Check the [docs](http://docs.vagrantup.com/v2/installation/index.html)

# Docker

Docker is a Linux container, based on [lxc](http://en.wikipedia.org/wiki/LXC) (self-described as “chroot on steroids”) and [AUFS](http://en.wikipedia.org/wiki/Aufs). Instead of providing a full VM, like you get with Vagrant, Docker provides you lightweight containers, that share the same kernel and allow to safely execute independent processes.

Docker is attractive for many reasons:

* Lightweight; images are much lighter than full VMs, and spinning off a new instance is lightning fast (in the range of seconds instead of minutes).
* Version control of the images, which makes it much more convenient to handle builds.
* Lots of images (again), just have a look at the docker public index of images.

# Now Let's get started.

* You should have virtualbox and vagrant ready.
* download an image

{% highlight bash %}
$ vagrant init precise64 http://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box
$ vagrant up
$ vagrant ssh
{% endhighlight %}
* you are all done
* There’s a 4 if you want to access your (soon to be) deployed app; you will need to dig around the Vagrant documentation to perform [port forwarding](http://docs.vagrantup.com/v2/networking/forwarded_ports.html), [proper networking](http://docs.vagrantup.com/v2/networking/private_network.html) and update manually your `Vagrantfile`

# in virtualbox now

* Install Docker. [as explainer on the official website](http://docs.docker.io/en/latest/installation/ubuntulinux/)

{% highlight bash %}
 $ sudo apt-get update
 $ sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring
 $ sudo reboot
 $ sudo sh -c "curl https://get.docker.io/gpg | apt-key add -"
 $ sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
 $ sudo apt-get update
 $ sudo apt-get install lxc-docker
{% endhighlight %}

* verity it worked by trying to build your first container:

` $ sudo docker run -i -t ubuntu /bin/bash`

* let's create a `Dockerfile` to build a new image

{% highlight bash %}
FROM ubuntu
 MAINTAINER My Self me@example.com

 # Update apt sources list to fetch mongodb and a few key packages
 RUN echo "deb http://archive.ubuntu.com/ubuntu precise universe" >> /etc/apt/sources.list
 RUN apt-get update
 RUN apt-get install -y python git

 # Finally - we wanna be able to SSH in
 RUN apt-get install -y openssh-server
 RUN mkdir /var/run/sshd
 RUN echo 'root:screencast' | chpasswd

 # And we want our SSH key to be added
 RUN mkdir /root/.ssh && chmod 700 /root/.ssh
 ADD id_rsa.pub /root/.ssh/authorized_keys
 ADD id_rsa.pub /root/.ssh/id_rsa.pub
 ADD id_rsa /root/.ssh/id_rsa
 RUN chmod 400 /root/.ssh/authorized_keys && chown root:root /root/.ssh/* && chmod 600 /root/.ssh/*

 # Expose a bunch of ports .. 22 for SSH and 3000 for our node app
 EXPOSE 22 3000

 CMD /usr/sbin/sshd -D
{% endhighlight %}

* get ssh-key

{% highlight bash %}
$ ssh-keygen
$ cp -a /home/vagrant/.ssh/id_rsa.pub .
$ cp -a /home/vagrant/.ssh/id_rsa .
{% endhighlight %}

* Now build it, more info about ssh into docker can be found [here](https://github.com/dotcloud/docker/blob/master/docs/sources/examples/running_ssh_service.rst)

`sudo docker build -t eg_sshd .`

Then run it. You can then use docker port to find out what host port the container's port 22 is mapped to:

{% highlight bash %}
$ sudo docker run -d -P --name test_sshd eg_sshd
$ sudo docker port test_sshd 22
# 0.0.0.0:49153
{% endhighlight %}

And now you can ssh to port `49153` on the Docker daemon's host IP address (ip address or ifconfig can tell you that):

```
$ ssh root@127.0.0.1 -p 49153
```

Now you are in a docker container. Yeah!
Finally, clean up after your test by stopping and removing the container, and then removing the image.

{% highlight bash %}
$ sudo docker stop test_sshd
$ sudo docker rm test_sshd
$ sudo docker rmi eg_sshd
{% endhighlight %}

# Let’s wrap it up

So we just saw (roughly) how these tools can be used, and how they can be complementary:

Vagrant will provide you with a full VM, including the OS. It’s great at providing you a Linux environment for example when you’re on MacOS.
Docker is a lightweight VM of some sort. It will allow you to build contained architectures faster and cheaper than with Vagrant.

It takes a bit of reading to get more familiar with these tools, this kind of technology allows you to automate and commoditize huge parts of your development and ops workflows. I strongly encourage you to make that investment. It has helped me tremendously increase the pace and quality of my throughput.
