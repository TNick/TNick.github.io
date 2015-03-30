---
layout: post
title:  "node.js application deployment"
date:   2014-10-25 19:45:00
categories: walkthrough
---

Overview
--------

This step-by-step article lists the steps required to install and run a
node.js application on a clean Ubuntu 14.04 server.

[Node.js](http://nodejs.org/) is a platform built on Chrome's JavaScript runtime for 
easily building fast, scalable network applications. Node.js uses 
an event-driven, non-blocking I/O model that makes it lightweight 
and efficient, perfect for data-intensive real-time applications 
that run across distributed devices.

[nginx (engine x)](http://nginx.org/) is an HTTP and reverse 
proxy server, as well as a mail proxy server. We are going to use it to allow
multiple node.js applications to run on the same physical server.

To see how you can prepare a virtual Ubuntu machine on Windows take a
look at [this post]({% post_url 2014-10-26-ubuntu-server-with-windows %}). 
Git server is described [here]({% post_url 2014-10-26-git-server %})
and has bears some resemblance with this post.



Node Application
================

On the Windows host computer I installed node.js from [here](http://nodejs.org/download/).
I've got an error: 
`Error: ENOENT, stat 'C:\Users\%USER%\AppData\Roaming\npm'`
with `npm` so I did a
 
	mkdir C:\Users\%USER%\AppData\Roaming\npm
	 
and `npm` was happy. I installed express globally using
 
	npm install -g express
	npm install -g express-generator

A minimal `.gitignore` can be:

	.bin
	node_modules/*
	*.user


node.js in Server
=================

Create a new user to run node tasks.

	sudo adduser noderunner

but don't switch to it, yet. Install npm and node:

	sudo apt-get install npm nodejs

In `/etc/ssh/sshd_config`  we may want to allow access to certain users.
Add a line like:

	AllowUsers git noderunner $YOUR_USER_NAME

	su noderunner
	mkdir .ssh && chmod 700 .ssh
	touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
	sudo chmod -R 0750 /home/git
	echo "ssh-rsa  ...  rsa-key-20141025" >> ~/.ssh/authorized_keys
	exit

Pom to the Rescue
=================

[Pod](https://github.com/yyx990803/pod) seems like exactly the application for us.

On the server
	
	sudo npm install -g pod

The switch to `noderunner` user and follow along:

	su noderunner

We need a directory for pod stuff, so I called it `podstuff`:

	pod

To create a new application:

	pod create myapp

In local computer: 

	git remote add deploy noderunner@$IP_OR_NAME:~/podstuff/repos/myapp.git
	git push deploy master

To check the status use either

	pod list

of check `$IP_OR_NAME:19999` using a web browser after 

	pod web

has been issued. The credentials are `admin/admin` and can be changed in 
`~/.podrc`. See an example config file with all options 
[inhere](https://github.com/yyx990803/pod#configuration).


nginx
=====

	sudo apt-get install nginx
	sudo service nginx start
	
	cd /etc/nginx/sites-available
	nano myapp

Fill in the content:


Enable the site:

	cd ../sites-enabled
	sudo ln -s /etc/nginx/sites-available/myapp myapp
	sudo rm default
	
And restart nginx:

	sudo service nginx restart


Resources
---------

- [Ubuntu OpenSSH Server docs](https://help.ubuntu.com/lts/serverguide/openssh-server.html)
- [Git server](http://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
- [The Node Beginner Book](http://www.nodebeginner.org/)
- [node and npm without sudo](https://gist.github.com/isaacs/579814)
- [Pod](https://github.com/yyx990803/pod)
- [Pod Walkthrough](https://github.com/yyx990803/pod/wiki/Setup-Walkthrough)
- [Deploy multiple Node applications](http://skovalyov.blogspot.dk/2012/07/deploy-multiple-node-applications-on.html)