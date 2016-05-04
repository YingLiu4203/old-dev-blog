---
layout: post
title: Create Odoo Docker image from GitHub source code 
---

Install Odoo from GitHub source code is a tedious process as shown in 
[the previous blog](http://www.mindissoftware.com/2014/08/17/Install-Odoo-v8-Ubuntu/)

[Docker](https://www.docker.com/) is a great tool that helps to build, 
ship and run an appcliaiton everwhere. The following are steps that I used to
 create a dockerized Odoo V8 application in Ubuntu 14.04 in digital ocean. 

---

### 1. Create and setup a digital ocean virtual machine  

### 1.1. Create a Ubuntu virtual machine and check/change system timezone

In digital ocean, when selecting image, choose "Docker 1.1.2 on Ubuntu" 14.04
 to create a virtual machine. 

Login as root, check timezone

    date
    more /etc/timezone
    
To change timezone from a selection, run 

    dpkg-reconfigure tzdata

Or from command line, change timezone to "Pacific" directly 

    ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime

Once you change it, restart cron to use the new timezone

    restart cron

#### 1.2. update/upgrade packages

    aptitude update
    aptitude upgrade -y
    
#### 1.3. enable IP firewall forward

edit /etc/default/ufw, change the forward policy line as the following: 

    DEFAULT_FORWARD_POLICY="ACCEPT"
    
save the file and reload UFW: 

    ufw reload

#### 1.4. Add a regular user and add it to sudo group
    
Login as root, create a new regular user account

    adduser <username>

Add odoo to sudo group

    adduser <username> sudo

Exit and login with the newly created <username>
    
    exit

Then login using the newly created <username>

### 2. Pull and run a Ubuntu image
 
    sudo docker pull ubuntu:14.4 
    
    # find the image id
    sudo docker images
    
    # run bash inside the image 
    sudo docker run -t -i <imageId> /bin/bash
    
    # Write down the container id after bash prompt  root@container_id
    # You need this to create a new image

### 3. Set the Ubuntu container

    # change timezone to "Pacific" 
    ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime
    
    # update and upgrade os
    apt-get update
    apt-get upgrade -y

### 4. Install, configure and start PostgreSQL database

#### 4.1 Install postgresql

    apt-get install -y postgresql
    
### 4.2. Edit the postgesql.conf file 
To accept connections on all interfaces (development use only)

    sudo vim /etc/postgresql/9.3/main/postgresql.conf
    
Find the listen parameter to change it as the following

    listen_addresses = '0.0.0.0'

#### 4.3. Edit the pg_hba.conf file
Change the local authentication and allow all remote connection

    sudo vim /etc/postgresql/9.3/main/pg_hba.conf
    
Find the follwing lines 

    local all all peer
    host all all 127.0.0.1/32 md5

change to 

    local all all md5
    host all all 0.0.0.0/0 md5

#### 4.4. Start postgresql database

    /etc/init.d/postgresql start

### 5. Create database account for Odoo 

#### 5.1. Create an odoo user account for Odoo application
 
    adduser --system --quiet --shell=/bin/bash --home=/opt/odoo --gecos 'ODOO' --group odoo

#### 5.2. Add postgresql super user odoo and set password 

    sudo -u postgres createuser -s odoo
    
    sudo -u postgres psql 
    ALTER ROLE odoo WITH PASSWORD 'odoo';
    \q
    
    exit

### 6. Install supporting packages for Odoo

#### 6.1. Install the required dependencies for ODOO

    apt-get install -y python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator python-requests python-pip   

#### 6.2. Install latest gdata-python-client 

    pip install gdata

#### 6.3. Install git

    apt-get install -y git

### 7. Install ODOO 8.0 from Github

#### 7.1 Get source code from GitHub 

    # the hyphen after su lets the system to create a new user environment 
    su - odoo
    git clone https://www.github.com/odoo/odoo --branch 8.0
    exit

#### 7.2. Configure the ODOO application

    cp /opt/odoo/odoo/debian/openerp-server.conf /etc/odoo-server.conf
    chown odoo: /etc/odoo-server.conf
    chmod 640 /etc/odoo-server.conf

#### 7.3. Edit /etc/odoo-server.conf file 
You need to set db_user, db_pasword, addons_path and logfile

    db_user = odoo
    db_password = XXXXXX
    
    addons_path = /opt/odoo/odoo/addons
    logfile = /var/log/odoo/odoo-server.log

#### 7.4. Create a dir for log file and set correct permission

    mkdir /var/log/odoo
    chown odoo:root /var/log/odoo

#### 7.5. Install a boot script
Create /etc/init.d/odoo-server with the following content

    #!/bin/sh
    ### BEGIN INIT INFO
    # Provides: odoo-server
    # Required-Start: $remote_fs $syslog
    # Required-Stop: $remote_fs $syslog
    # Should-Start: $network
    # Should-Stop: $network
    # Default-Start: 2 3 4 5
    # Default-Stop: 0 1 6
    # Short-Description: Business Applications
    # Description: ODOO Business Applications.
    ### END INIT INFO
    
    PATH=/bin:/sbin:/usr/bin
    DAEMON=/opt/odoo/odoo/openerp-server
    NAME=odoo-server
    DESC=odoo-server
    
    # Specify the user name (Default: openerp).
    USER=odoo
    
    # Specify an alternate config file (Default: /etc/odoo-server.conf).
    CONFIGFILE="/etc/odoo-server.conf"
    
    # pidfile
    PIDFILE=/var/run/$NAME.pid
    
    # Additional options that are passed to the Daemon.
    DAEMON_OPTS="-c $CONFIGFILE"
    [ -x $DAEMON ] || exit 0
    [ -f $CONFIGFILE ] || exit 0
    checkpid() {
    [ -f $PIDFILE ] || return 1
    pid=`cat $PIDFILE`
    [ -d /proc/$pid ] && return 0
    return 1
    }
    
    case "${1}" in
    start)
    echo -n "Starting ${DESC}: "
    start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
    --chuid ${USER} --background --make-pidfile \
    --exec ${DAEMON} -- ${DAEMON_OPTS}
    echo "${NAME}."
    ;;
    stop)
    echo -n "Stopping ${DESC}: "
    start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
    --oknodo
    echo "${NAME}."
    ;;
    
    restart|force-reload)
    echo -n "Restarting ${DESC}: "
    start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
    --oknodo
    sleep 1
    start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
    --chuid ${USER} --background --make-pidfile \
    --exec ${DAEMON} -- ${DAEMON_OPTS}
    echo "${NAME}."
    ;;
    *)
    N=/etc/init.d/${NAME}
    echo "Usage: ${NAME} {start|stop|restart|force-reload}" >&2
    exit 1
    ;;
    
    esac
    exit 0


Then change the service file permission 

    chmod 755 /etc/init.d/odoo-server
    
#### 7.6. Create a start script 

Create file /opt/odoo/start-odoo with the following content

    #!/bin/sh
    
    /etc/init.d/postgresql start
    /etc/init.d/odoo-server start
    
Then the script file attribute

    chmod 755 /opt/odoo/start-odoo

Test the installation

     /opt/odoo/start-odoo
     
Check that both Postgresql and Odoo service are running in their ports 

     netstat -anltp
          
#### 7.7. Exit from containter

    exit
    
### 8. Commit container to docker hub

    sudo docker commit <container-id> <username>/<repository>:<tag>
    
### 9. Pull and run the image

    sudo docker pull <username>/<repository>:<tag>
    
    # First time, create the container and run it
    # -p set the port mapping
    sudo docker run -t -i --name="odootest" -p 5432:5432 -p 8069:8069 <username>/<repository>:<tag> /bin/bash
    /opt/odoo/start-odoo
    
    # after the first time, you can run the container using the following command 
    sudo docker start -a -i odootest 
    /opt/odoo/start-odoo
    
Now you should be able to see Odoo up and running from your browser in 
address:  http://host-server-ip:8069

You can also use the 5432 port to manage Postgresql database.  