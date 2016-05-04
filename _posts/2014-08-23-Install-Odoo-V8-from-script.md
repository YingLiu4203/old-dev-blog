---
layout: post
title: Create Odoo V8 Docker image from GitHub source code using shell script 
---

The following steps are based on [ANDRÉ SCHENKELS's installation script.]
(https://github.com/aschenkels-ictstudio/openerp-install-scripts/blob/master/odoo-v8/ubuntu-14-04/odoo_install.sh)
with below changes. Some changes such as local settings are necessary because 
we starts from a base Ubuntu operating system. 

* It sets locale LANG to UTF-8 thus Postgresql uses the same encoding 
for template database. The original script doesn't work if it is not set.   
* It uses version 8.0 branch instead of saas-5
* The 8.0 configuration file is in a different folder
* It starts postgresql database to add a database user
* The latest 8.0 requires the installation of python-decorator  
* The start.sh needs to set HOME environment variable. 

I like this method because it installs Odoo v8 from source code and I 
have full control about the configuration and build process. 
As shown below, it is also easy to create a Docker image from it. 

By the way, you can ignore the Docker parts and just use the script to 
install Odoo from its 8.0 branch in GitHub. 

#### 1. First create and run a Ubuntu 14.04 container 

```bash
sudo docker run -t -i --name ubuntubase ubuntu:14.04 /bin/bash
```
    
#### 2. Create and run installation script 
Once inside the ubuntu container, create a shell script named 
odoo_installation.sh with the same content as the script 
at the end of this blog. Plese feel free to 
change the script to fit your needs.

Make it executable and run it: 

```bash
chmod +x odoo_installation.sh
./odoo_installation.sh 2>&1 | tee odoo_installation.log
```
    
This will install both Postgresql and Odoo V8.  
 
#### 3. Exit from the container to create a new Docker image

```bash
exit
sudo docker commit ubuntubase odoov8
sudo docker run --name odoov8server -p 5432:5432 -p 8069:8069 -d odoov8 /opt/odoo/odoo-server/start.sh
```
    
The above command creates a new container odoov8 and starts 
Odoo v8 application. It maps both Postgresql and Odoo port that can be 
connected in the Docker host server port. 

You can browser to the host_ip:8069 to see the database setup page. 
You should use "superadminpassword" as the master password 
because we set admin_passwd to this value.  

#### 4. How to login to the odoov8 container
You can install [nsenter GitHub project](https://github.com/jpetazzo/nsenter) 
and use docker-enter command. 

```bash
# install nsenter into /usr/local/bin
docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter

# login odoo container
docker-enter odoov8server 
```
 
#### 5. To start and stop the Odoo application

```bash
# start command
docker start odoov8server

# stop command
docker stop odoov8server
```

### Revised installation script
---

```bash
#!/bin/bash
#***************************************************************
# Script for Installation: ODOO 8.0 server on Ubuntu 14.04 LTS
# Author: Originally by André Schenkels, ICTSTUDIO 2014
#         Revised by Ying Liu  www.MindIsSoftware.com 2014/08/24
#***************************************************************
 
##fixed parameters
OE_USER="odoo"
OE_HOME="/opt/$OE_USER"
OE_HOME_EXT="/opt/$OE_USER/$OE_USER-server"

# Enter version for checkout "7.0" for version 7.0, "saas-4, saas-5 (opendays version) and "master" for trunk
# use version 8.0 !!!
OE_VERSION="8.0"

# set the superadmin password
OE_SUPERADMIN="superadminpassword"
OE_CONFIG="$OE_USER-server"

# set local thus Postgresql template0 uses the same encoding as Odoo
locale-gen en_US.UTF-8 && update-locale
echo 'LANG="en_US.UTF-8"' > /etc/default/locale
echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

#--------------------------------------------------
# Update Server
#--------------------------------------------------
echo -e "\n---- Update Server ----"
sudo apt-get update
sudo apt-get upgrade -y

#--------------------------------------------------
# Install tools
#--------------------------------------------------
echo -e "\n---- Install tool packages ----"
sudo apt-get install -y vim wget curl git python-pip

#--------------------------------------------------
# Install PostgreSQL Server
#--------------------------------------------------
echo -e "\n---- Install PostgreSQL Server ----"
sudo apt-get install -y postgresql-9.3
    
echo -e "\n---- PostgreSQL $PG_VERSION Settings  ----"
sudo sed -i s/"#listen_addresses = 'localhost'"/"listen_addresses = '*'"/g /etc/postgresql/9.3/main/postgresql.conf

echo -e "\n---- Starting PostgreSQL   ----"
# start postgresql thus we can add user later
sudo /etc/init.d/postgresql start

echo -e "\n---- Creating the ODOO PostgreSQL User  ----"
sudo su - postgres -c "createuser -s $OE_USER"

#--------------------------------------------------
# Install dependencies
#--------------------------------------------------
echo -e "\n---- Install python packages ----"
# need to install python-decorator !!!
sudo apt-get install python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator -y
    
echo -e "\n---- Install python libraries ----"
sudo pip install gdata
    
echo -e "\n---- Create ODOO system user and set password ----"
sudo adduser --disabled-password --quiet --shell /bin/bash --home $OE_HOME --gecos 'ODOO' $OE_USER
sudo echo "$OE_USER:$OE_USER" | chpasswd

echo -e "\n---- Create Log directory ----"
sudo mkdir /var/log/$OE_USER
sudo chown $OE_USER:$OE_USER /var/log/$OE_USER

#--------------------------------------------------
# Install ODOO
#--------------------------------------------------
echo -e "\n==== Installing ODOO Server ===="
sudo git clone --branch $OE_VERSION https://www.github.com/odoo/odoo $OE_HOME_EXT/

echo -e "\n---- Create custom module directory ----"
sudo mkdir -p $OE_HOME/custom/addons

echo -e "\n---- Setting permissions on home folder ----"
sudo chown -R $OE_USER:$OE_USER $OE_HOME/*

echo -e "* Create server config file"
# the 8.0 branch config file is different !!!
sudo cp $OE_HOME_EXT/debian/openerp-server.conf /etc/$OE_CONFIG.conf
sudo chown $OE_USER:$OE_USER /etc/$OE_CONFIG.conf
sudo chmod 640 /etc/$OE_CONFIG.conf
    
echo -e "* Change server config file"
sudo sed -i s/"db_user = .*"/"db_user = $OE_USER"/g /etc/$OE_CONFIG.conf
sudo sed -i s/"; admin_passwd.*"/"admin_passwd = $OE_SUPERADMIN"/g /etc/$OE_CONFIG.conf
sudo echo "logfile = /var/log/$OE_USER/$OE_CONFIG.log" >> /etc/$OE_CONFIG.conf
sudo echo "addons_path=$OE_HOME_EXT/addons,$OE_HOME/custom/addons" >> /etc/$OE_CONFIG.conf

echo -e "* Create startup file"
# we changed a lot here to start Odoo from a command line !!!
sudo echo "#!/bin/sh" >> $OE_HOME_EXT/start.sh
sudo echo "/etc/init.d/postgresql start" >> $OE_HOME_EXT/start.sh 

# need to set HOME to run odod !!!
sudo echo "sudo HOME=$OE_HOME/ -u $OE_USER $OE_HOME_EXT/openerp-server --config=/etc/$OE_CONFIG.conf"  >> $OE_HOME_EXT/start.sh

sudo chown $OE_USER:$OE_USER $OE_HOME_EXT/start.sh
sudo chmod 755 $OE_HOME_EXT/start.sh

echo "Done! The ODOO server can be started with $OE_HOME_EXT/start.sh"
```
 