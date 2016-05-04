---
layout: post
title: Install Odoo V8 in Ubuntu from GitHub source code 
---

*This blog is based on 
[ANDRÉ SCHENKELS's post.] 
(http://www.schenkels.nl/2014/07/install-odoo-v8-0-from-github-ubuntu-14-04-lts-formerly-openerp/)
The whole process can be fully automated by download and run [his installation script.] 
(http://www.schenkels.nl/2014/07/odoo-v8-install-script-github-ubuntu-14-04-lts/)*

The following steps were tested in a digital ocean Ubuntu 14.04 system. 
---

### 1. In a freshly installed Ubuntu 14.04, check/change system timezone

Login as root, check timezone

    date
    more /etc/timezone
    
To change timezone, run 

    dpkg-reconfigure tzdata

Once you change it, restart cron to use the new timezone

    restart cron

### 2. update/upgrade packages

    aptitude update
    aptitude upgrade -y

### 3. Add a regular user and add it to sudo group
    
Login as root, create a new regular user

    adduser <username>

Add odoo to sudo group

    adduser <username> sudo

Exit and login with the newly created <username>
    
    exit

Then login with the newly created <username>

### 4. Create an odoo user account for Odoo application
 
    sudo adduser --system --quiet --shell=/bin/bash --home=/opt/odoo --gecos 'ODOO' --group odoo

### 5. Install PostgreSQL database

    sudo apt-get install postgresql -y

### 6. Add super user odoo and set password 

    sudo su postgres
    createuser -s odoo
    
    psql template1
    ALTER ROLE odoo WITH password 'XXXXX';
    \q
    
    exit

### 7. Edit the postgesql.conf file 
To accept connections on all interfaces (development use only)

    sudo vim /etc/postgresql/9.3/main/postgresql.conf
    
Find the listen parameter to change it as the following

    listen_addresses = '0.0.0.0'

### 8. Edit the pg_hba.conf file
Change the local authentication and allow all remote connection

    sudo vim /etc/postgresql/9.3/main/pg_hba.conf
    
Find the follwing lines 

    local all all peer
    host all all 127.0.0.1/32 md5

change to 

    local all all md5
    host all all 0.0.0.0/0 md5

### 9. Install the required dependencies for ODOO

    sudo apt-get install -y python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator python-requests python-pip

### 10. Install latest gdata-python-client 

    sudo pip install gdata

### 11. Install git

    sudo apt-get install git -y

### 12. Install ODOO 8.0 from Github

    sudo su - odoo
    git clone https://www.github.com/odoo/odoo --branch 8.0
    chown -R odoo: *
    exit

### 13. Configure the ODOO application

    sudo cp /opt/odoo/odoo/debian/openerp-server.conf /etc/odoo-server.conf
    sudo chown odoo: /etc/odoo-server.conf
    sudo chmod 640 /etc/odoo-server.conf

### 14. Edit /etc/odoo-server.conf file 
You need to set db_user, db_pasword, addons_path and logfile

    db_user = odoo
    db_password = XXXXXX
    
    addons_path = /opt/odoo/odoo/addons
    logfile = /var/log/odoo/odoo-server.log

### 15. Create a dir for log file and set correct permission

    sudo mkdir /var/log/odoo
    sudo chown odoo:root /var/log/odoo

### 16. Check if the server works. 
After type the following, use a browser go to  http://YourServerIp:8069, 
you should see the login screen

    sudo su odoo
    python /opt/odoo/odoo/openerp-server --config=/etc/odoo-server.conf

After you check it, press CTRL+C to stop the server and log out odoo account

    exit

### 17. Install a boot script
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


### 18. Change service file permission and start service 

    sudo chmod 755 /etc/init.d/odoo-server
    sudo chown root: /etc/init.d/odoo-server
    sudo /etc/init.d/odoo-server start

### 19. Check the logfile and see that the server has started

    less /var/log/odoo/odoo-server.log

### 20. Automatic startup and shutdown

    sudo update-rc.d odoo-server defaults