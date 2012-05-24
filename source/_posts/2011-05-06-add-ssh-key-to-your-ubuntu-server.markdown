---
layout: post
title: "Add ssh key to your ubuntu server"
date: 2011-05-06 02:27am
comments: true
categories: [ubuntu]
---
Add ssh-key to your ubuntu server is a common task for most developers to be able to remote your dedicated server via SSH. Normally, I do this with my local virtual box server. It's fairly easy to do so. Here, I just copied the steps from Yoolk Wiki, written by my boss, Chris.

Generally, it works well on Ubuntu 9.04, but there is problem with Ubuntu 10.04. I'll show you here:

1. server: `sudo apt-get update`
2. server: `sudo apt-get install openssh-server`
3. server: `mkdir .ssh`
4. client: `ssh-keygen` (don't enter any values, press return three times, yes passwords should be blank)
5. client: `cat .ssh/id_rsa.pub` - copy the output to the clipboard (very carefully, no pre/trailing white space)
6. server: `touch .ssh/authorized_keys`
7. server: `sudo vim .ssh/authorized_keys` - paste clipboard contents (in order to paste from clipboard, you must remote to your server by login through terminal console)

For Ubuntu 10.04, you must run this command `ssh-add`. If it adds duplicate keys, run `ssh-add -D` and run `ssh-add` again. Hope it could help.
