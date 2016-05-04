---
layout: post
title: Install Odoo nightly build from a Docker image 
---

### Introduction
 
While [the previous blog](http://www.mindissoftware.com/2014/08/19/Install-Odoo-using-Docker/) 
shows an ugly way to create Odoo Docker image from GitHub source code, 
actually install and run Odoo from a Docker image is very easy. 

This [Docker hub repository] (https://registry.hub.docker.com/u/shaker/odoo/) 
has an Odoo V8 image created on 08/17/2014. The image doesn't use the 
local Postgresql installed with Odoo. The same user created a separate 
[Postgresql 9.3]
(https://registry.hub.docker.com/u/shaker/postgresql/) 
that works with the Odoo image.

### Performance Concern 

To use the above Odoo Docker image we need to run two Docker containers:
one for Posgtresql database and one for Odoo. How about the performance?
A quick Google search led me to the post of 
[Postgresql Performance on Docker]
(http://www.davidmkerr.com/2014/06/postgresql-performance-on-docker.html) 
as well as [its discussion in Hacker News]
(https://news.ycombinator.com/item?id=8010101). 
To my surprise, my finding is that a Docker container puts negligible 
performance penalty on the contained application -- especially when 
one doesn't use virtual network and virtual I/O. Actually in the author's 
tests, dockerized applications had better performance than the native ones. 
It is safe to say that it is a good practice to run the database server 
and application server even in production. All required is a **RIGHT** 
network and I/O configuration of all containers and their links. 
Let's do it. 

### Installation Steps 

The following steps are executed in a Digital Ocean Docker 1.12 Ubuntu 
14.04 virtual machine. 

#### 1. A quick installation

I followed the following instructions in the Odoo Docker repository 

    docker run --name postgres -d shaker/postgresql
    docker run --name odoo --link postgres:db -p 8069:8069 -p 2222:22 -d shaker/odoo:8
    
It took less than 5 minutes and I was ready to test. 
Excited, I opened a browser and went to `http://<my server ip>:8069`, 
however, I got **Internal Server Error** message. 

#### 2. Troubleshooting

I couldn't ssh to the odoo container. Luckily, there is a 
[nsenter GitHub project](https://github.com/jpetazzo/nsenter). 

```bash
# install nsenter into /usr/local/bin
docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter

# login odoo container
docker-enter odoo
```
    
The Odoo log showed that it couldn't connect to Postgresql server. 
Login to the postgres container. 

    # login postgres container
    docker-enter postgres
    
I found that Postgresql didn't start. I tried to run it

    /etc/init.d/postgresql start
    
I got the following error message

    The PostgreSQL server failed to start. Please check the log output: 
    ... UTC FATAL:  could not access private key file "/etc/ssl/private/ssl-cert-snakeoil.key": Permission denied
    
It turned out the postgres account couldn't access the file. 
edit /etc/postgresql/9.3/main/postgresql.conf to set `ssl=false`
 
Ran it again and everything worked

    docker stop odoo
    docker stop postgres
    
    docker start postgres
    docker start odoo
