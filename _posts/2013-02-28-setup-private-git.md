---
layout: post
title: Setup private Git server
date: 2013-02-28 08:43
comments: true
categories: git
---

1. Install Git

{% highlight bash %}
ssh myserver.com
sudo apt-get update
sudo apt-get install git-core
{% endhighlight %}

2. Setup git user

{% highlight bash %}
sudo adduser git
sudo mkdir /home/git/.ssh
sudo cp ~/.ssh/authorized_keys /home/git/.ssh/
sudo chown -R git:git /home/git/.ssh
sudo chmod 700 !$
sudo chmod 600 /home/git/.ssh/*
{% endhighlight %}

<!-- more -->
3. Using git user login to server with ssh，`ssh git@yourserver.com`
4. Create git repo on the server

{% highlight bash %}
mkdir myrepo.git
cd !$
git --bare init
{% endhighlight %}
5. Push local code to server

{% highlight bash %}
git remote rm origin
git remote add origin git@yourserver.com:myrepo.git
# or use below line to replay above two lines, if you already have a remote origin url
git remote set-url origin git@yourserver.com:myrepo.git
git push origin master
git config branch.master.remote origin && git config branch.master.merge refs/heads/master
{% endhighlight %}
6. Bonus points: Make SSH more secure

Edit the SSH config:
{% highlight bash %}
sudo vi /etc/ssh/sshd_config
{% endhighlight %}
And change the following values:

{% highlight bash %}
Port 2207
...
PermitRootLogin no
...
AllowUsers myuser git
...
PasswordAuthentication no
{% endhighlight %}
Where 2207 is a port of your choosing. Make sure to add this so your Git remote:

{% highlight bash %}
git remote add origin ssh://git@yourserver.com:2207/~/myrepo.git
{% endhighlight %}

