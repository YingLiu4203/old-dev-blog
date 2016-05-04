---
layout: post
title: Script to add a user and set password in Ubuntu 
---

It took me some time to add a new user and set a password in shell script. 
Eventually the code below worked in Ubuntu 14.04. 

```bash
# quietly add a user without password, also create a group with the same name
adduser --quiet --disabled-password -shell /bin/bash --home /home/newuser --gecos "User" newuser

# set password
echo "newuser:newpassword" | chpasswd

# to add the new user to sudo group 
# the -a option keeps all existing groups
usermod -aG sudo newuser
```