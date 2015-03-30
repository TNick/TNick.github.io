---
layout: post
title:  "Ubuntu server controlled from Windows"
date:   2014-10-26 23:31:00
categories: walkthrough
---

Overview
--------

This post covers the basics of installing and controlling 
an Ubuntu server on a Windows host machine. Once you play with it 
and you become proficient on your machine 
you can rent a remote server and do more interesting
things like publishing your site.


The OS
======

A 32 MB minimal image for Ubuntu can be retreived 
[here](https://help.ubuntu.com/community/Installation/MinimalCD)
and 14.04 normal and server CD are 
[here](http://de.releases.ubuntu.com/trusty/). On Windows 8.1 64bit I was
unable to install regular distribution due to IO errors. Minimal and
server went OK. 


Windows
=======

To interact with an Ubuntu server from a Windows machine we can use 
[MTPuTTY](http://ttyplus.com/multi-tabbed-putty/) as a front-end for either 
[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) or
[KiTTY](http://www.9bis.net/kitty/?page=Download).

`puttygen.exe` (separate download) can be used to generate a ssh key. 
Once the key is generated it also needs to be exported in OpenSSL
and saved inside `%USER_DIR%\.ssh\id_rsa` so that git can pick it up.

[VMware player](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/4_0)
can be used in Windows.


Installing
==========

To install the new operating system you will need to use the virtualization
method of choice. Create a (dynamically expanding) 20 GB virtual hard disk,
insert the .iso file and install following the on-screen guide.

The server installer also allows some pre-made stacks
like LAMP to be installed. I chose not to so I was forced to install everything by hand.
The minimal installer offered the option to install a basic server so I went for it.

I started with customary

    sudo apt-get update
    sudo apt-get upgrade
	
in the server (the minimal installation is already up to date)
then I copied the virtual hard-drives so that I don't have to install Ubuntu
every time I need a fresh installation.

    sudo shutdown 0


Remote Access
=============

to enable `ssh` access I installed OpenSSH server on Ubuntu machine:

    sudo apt-get install openssh-server
	
After this point I was able to use ssh to access remote machine using its ip.
To discover the IP use `ifconfig` in Ubuntu terminal.


Users
=====

While installing an initial user is going to be set-up.
Additional users can be created using:

	sudo adduser new_name

and following the on-screen instructions.

To prepare a user for ssh access: 

	su git
	cd ~
	mkdir .ssh && chmod 700 .ssh
	touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
	sudo chmod -R 0750 /home/new_name

Save your ssh key inside `authorized_keys`:

	echo "ssh-rsa  ...  rsa-key-20141025" >> ~/.ssh/authorized_keys

and replace `...` with your actual key.

In `/etc/ssh/sshd_config`  we may want to allow access to certain users.
Add a line like:

	AllowUsers john mary ghiury


Done
====

Now fire MTPuTTY, add the address of the server and credentials and you are 
in control of your server.

What can you do with your new shining server? Here's some ideas: 

- [Set-up a git server]({% post_url 2014-10-26-git-server %}) 
- [Run a node application]({% post_url 2014-10-25-nodejs-application-deploy %}) 


Resources
---------

- [Ubuntu OpenSSH Server docs](https://help.ubuntu.com/lts/serverguide/openssh-server.html)
