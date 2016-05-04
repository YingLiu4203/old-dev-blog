---
layout: post
title: Create Odoo 8.0 Ubuntu Docker Image 
---

## Overview
Odoo 8 was released on 09/18/2014. We created a self-contained 
[Odoo 8.0 Docker image]
(https://registry.hub.docker.com/u/yingliu4203/odoo8nightly/) that 
was installed from [the latest Odoo 8.0 nightly build] 
(http://nightly.openerp.com/8.0/nightly/deb/). 
The Docker image building source code is in [a GitHub repository]
(https://github.com/YingLiu4203/odoo-v8-nightly-docker). 

* It uses the recently available Odoo 8.0 nightly build
* It is self-contained. We use a local postgresql database that 
comes with the Odoo 8.0 installation.
* It does not use extra configuration files. All configuration files 
are created using Docker commands.
* It creates an user account ##odoo## with initial password ##odoo##, 
you can use ssh to login to the container. The ssh port number is mapped
to 2222 in the docker host. 

## To Use It

Run the following command in a machine that has Docker installed. 

```bash
sudo docker run --name odoo8 -p 2222:22 -p 5432:5432 -p 8069:8069 -d yingliu4203/odoo8nightly
```

The above command creates a Docker instance named "odoo8" from the 
Docker image.  If not for a Docker bug, the above command is all you need. 
However, there is a "Permission denied" Postgresql error when 
you run an image that was built in Docker Hub. To fix it, 
you need to login to the container using a tool called
[nsenter](https://github.com/jpetazzo/nsenter) and change 
the owner of a Postgresql directory.

```bash
# use ssh to login to the docker container to fix file permission bug 
sudo chown postgres:postgres -R /var/lib/postgresql/9.3/main/
```

After the above fix, all services are in good standing. Just type 
"http://your-host-ip:8069" in your browser and enjoy. 

To stop it:

```bash
sudo docker stop odoo8
```

To restart it:

```bash
sudo docker start odoo8
```

The created Docker container exposes SSH (from 22 to 2222), 
Postgresql (5432) and Odoo (8069) ports in the host. 
You can connect to these services remotely.To use SSH, 
you need to config user and password in the container.
 
In the container, you can stop Postgresql or Odoo service
by finding their process id and `kill`  them.
Then you can start the services using the init.d service command

```bash
/etc/init.d/postgresql start
/etc/init.d/openerp start
```

## Technical issues 

The [Docker build file] 
(https://github.com/YingLiu4203/odoo-v8-nightly-docker/blob/master/Dockerfile)
looks simple. However, there were some technical issues in creating
this file. We document them here thinking that somebody may want
to fork and customize it.  

* Set locale. Otherwise, Postgresql and Odoo may use different locale.
* Make /var/run/sshd directory. Otherwise, SSH may not run.
* Set "ssl=false" in Postgresql configuration to fix another Docker bug.  
* Run all services in interactive mode using Supervisor. 
A service starts/stops when a container starts/stops. If you want,
you can kill any service and re-start it inside a container.
* Create home directory for openerp user. Odoo creates this 
account but doesn't create the home directory. 
* Set **"HOME"** environment variable for **odoo** user. Odoo may
complain if it is not there. 
* Use CMD, not ENTTRYPOINT, to run supervisord
* Even I put `chown postgres:postgres -R /var/lib/postgresql/9.3/main/`
into the Dockerfile, the downloaded image still has wrong owner for 
base/ directory. 

The tricky thing is that some issues, such as the Postgresql directory
ownership bug, only occur when you image is built by Docker Hub. 
 