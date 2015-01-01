---
layout: post
title:  "Git server in Ubuntu"
date:   2014-10-26 19:49:00
categories: walkthrough
---

Overview
--------

This step-by-step article lists the steps required to prepare a git server.
It will be capable of hosting your source code and you will be able 
to do push and pull over it.


The OS
======

I'm using Ubuntu 14.04 right now. The server is assumed to be
prepared as described in [this post]({% post_url 2014-10-26-ubuntu-server-with-windows %}) (basically an Ubuntu with OpenSSH). 


Git Server
==========

We want to deploy our changes as git commits.
This will take care in the end of two things:
version control and deploying the application.

	sudo apt-get install git

Prepare git user:

	sudo adduser git
	su git
	cd
	mkdir .ssh && chmod 700 .ssh
	touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
	sudo chmod -R 0750 /home/git


Save your ssh key inside `authorized_keys`:

	echo "ssh-rsa  ...  rsa-key-20141025" >> ~/.ssh/authorized_keys

Edit `/etc/shells` and add `/usr/bin/git-shell` as the last line, then make 
it the login shell for user git:

	sudo echo "/usr/bin/git-shell" >> /etc/shells
	echo "/usr/bin/git-shell" | sudo chsh git

	sudo mkdir /home/git/git-shell-commands
	sudo cat >/home/git/git-shell-commands/no-interactive-login <<\EOF
	#!/bin/sh
	printf '%s\n' "Hi $USER! You've successfully authenticated, but I do not"
	printf '%s\n' "provide interactive shell access."
	exit 128
	EOF
	sudo chmod +x /home/git/git-shell-commands/no-interactive-login
	sudo chown -R git:git /home/git/


Now create a test repository:

	cd ~
	mkdir project.git
	cd project.git
	git init --bare

Clone the test repository in a local directory, make some changes, commit and push:

	git clone git@192.168.110.128:~/project.git
	echo "some test" > test.file
	git add .
	git ci -am "initial commit"
	git push

You will have to configure some globals if this is your first time:

	git config --global user.email "nicu.tofan@gmail.com"
	git config --global user.name "Nicu Tofan"
	git config --global alias.co checkout
	git config --global alias.br branch
	git config --global alias.st status
	git config --global alias.ci "commit -v"
	git config --global alias.logg "log --pretty=format:\"%h %s\" --graph"
	git config --global alias.last "log -1 HEAD"
	git config --global alias.unstage "reset HEAD --"
	git config --global push.default simple


Resources
---------

- [Git server](http://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
