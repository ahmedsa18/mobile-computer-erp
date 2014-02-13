mobile-computer-erp
===================

erp solution for mobile/computer repair shop using python and openerp


Installing OpenERP v7.0 on Ubuntu 12.04 (x64) from launchpad repository.

just after a clean installation of ubuntu 12.04 x64 Server LTS

Commands are complete so you can copy paste these commands below to and complete the process as a newbie.

Installation from launchpad makes update and patching easier. If you just want to try openERP I recommend you to install with a .deb package. Installation with .deb package just takes 1-2 minutes.

Update apt source list

sudo apt-get update

Download and install updates

sudo apt-get upgrade

install required packages for openerp

sudo apt-get install graphviz ghostscript postgresql-client \
python-dateutil python-feedparser python-matplotlib \
python-ldap python-libxslt1 python-lxml python-mako \
python-openid python-psycopg2 python-pybabel python-pychart \
python-pydot python-pyparsing python-reportlab python-simplejson \
python-tz python-vatnumber python-vobject python-webdav \
python-werkzeug python-xlwt python-yaml python-imaging

install some other packages that we will probably need in future

sudo apt-get install gcc python-dev mc bzr python-setuptools python-babel \
python-feedparser python-reportlab-accel python-zsi python-openssl \
python-egenix-mxdatetime python-jinja2 python-unittest2 python-mock \
python-docutils lptools make python-psutil python-paramiko poppler-utils \
python-pdftools antiword postgresql

install gdata client (since ubuntu package is old we download and install from source)

wget http://gdata-python-client.googlecode.com/files/gdata-2.0.17.tar.gz 
tar zxvf gdata-2.0.17.tar.gz 
cd gdata-2.0.17/
sudo python setup.py install

create a new openerp system user for openerp and other related processes

sudo adduser openerp --home /opt/openerp

Create database user for openerp

cd ..
sudo -u postgres createuser -s openerp

move to the install directory

sudo su openerp
mkdir /opt/openerp/v7
cd /opt/openerp/v7

run the bazaar to download the latest revision from launchpad
(this will take long if an error occurs just rerun command)

bzr branch lp:openerp-web/7.0 web

(this will take longer if an error occurs just rerun command)

bzr branch lp:openobject-server/7.0 server

(this will take much more longer if an error occurs just rerun last command)

bzr branch lp:openobject-addons/7.0 addons

It would be nice if you check if the downloaded revision no pass the tests from http://runbot.openerp.com

Exit from openerp user shell

exit

Copy OpenERP configuration file to /etc

sudo cp /opt/openerp/v7/server/install/openerp-server.conf /etc/openerp-server.conf

Edit the configuration file

sudo nano /etc/openerp-server.conf

Write your database operation password and remove the semicolon of that line. Also add the addons_path to the end of file, my file looks like below.

[options]
; This is the password that allows database operations:
admin_passwd = PASSWORD
db_host = False
db_port = False
db_user = openerp
db_password = False
addons_path = /opt/openerp/v7/addons,/opt/openerp/v7/web/addons
;Log settings
logfile = /var/log/openerp/openerp-server.log
log_level = error

Change the file permissions and file ownership to the openerp user.

sudo chown openerp: /etc/openerp-server.conf
sudo chmod 640 /etc/openerp-server.conf

Create log directory for openerp

sudo mkdir /var/log/openerp
sudo chown openerp:root /var/log/openerp

Copy Logrotate file from source to /etc/logrotate.d folder

sudo cp /opt/openerp/v7/server/install/openerp-server.logrotate /etc/logrotate.d/openerp-server
sudo chmod 755 /etc/logrotate.d/openerp-server

Run the server

sudo su openerp
cd /opt/openerp/v7/server/
./openerp-server -c /etc/openerp-server.conf &

test your installation from http://yourserverIP:8069



Starting Server Up Automatically
================================

After playing with init script in source I decided to use Open Sourcerer's init script since it is clean and nice. You can download the original from: http://www.theopensourcerer.com/wp-content/uploads/2012/12/openerp-server

We just need to change the location of the daemon. below is the init script you can copy paste this to the file.

#!/bin/sh

### BEGIN INIT INFO
# Provides:             openerp-server
# Required-Start:       $remote_fs $syslog
# Required-Stop:        $remote_fs $syslog
# Should-Start:         $network
# Should-Stop:          $network
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Enterprise Resource Management software
# Description:          Open ERP is a complete ERP and CRM software.
### END INIT INFO

PATH=/bin:/sbin:/usr/bin
DAEMON=/opt/openerp/v7/server/openerp-server
NAME=openerp-server
DESC=openerp-server

# Specify the user name (Default: openerp).
USER=openerp

# Specify an alternate config file (Default: /etc/openerp-server.conf).
CONFIGFILE="/etc/openerp-server.conf"

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

You can download this edited copy with the following command First Exit from openerp users shell

exit
sudo nano  /etc/init.d/openerp-server

Make the init script executable.

sudo chmod +x /etc/init.d/openerp-server

Add openerp-server to system startup

sudo update-rc.d openerp-server defaults

Restart the server to check if init script works

sudo shutdown -r now

After restart you should be able to connect to the server via http://yourip:8069

Notes:

    Used some parts from this nice tutorial: http://www.theopensourcerer.com/2012/12/how-to-install-openerp-7-0-on-ubuntu-12-04-lts/
    I tested above commands with a clean ubuntu 12.04 install and OpenERP works as expected.


To update this installation to the latest revision
==================================================

sudo /etc/init.d/openerp-server stop
sudo su openerp
cd /opt/openerp/v7/addons/
bzr pull
cd /opt/openerp/v7/web/
bzr pull
cd /opt/openerp/v7/server/
bzr pull

It would be nice if you check if the downloaded revision no pass the tests of http://runbot.openerp.com You should run the server with your database name and update your database with the following command.

./openerp-server -c /etc/openerp-server.conf -u all -d YOURDATABASENAME


To Apply a Patch
================

To apply a patch you should have a patch file or URL of the patch file.

Cd to the project folder (addons,web,server etc). apply patch with bzr command bzr patch.

to apply a patch to the addons branch:

cd /opt/openerp/v7/addons/
bzr patch PATCH_FILE_NAME_OR_URL


To install an additional independent V7 test server on the same OS
==================================================================

On production you would need to test some code changes or patches or some new modules. To be able to test without risking your production server a test server is a good idea. Best option is to have a virtual machine with openerp installed. However if you like to have an seperate openerp setup.

coming...


Notes:


Why 64 bit not 32
=================

I am using an x64 version of ubuntu because my server has more than 4GB of RAM. And 64 is bigger than 32 :) "Unless you have specific reasons to choose 32-bit, we recommend 64-bit to utilise the full capacity of your hardware." https://help.ubuntu.com/community/32bit_and_64bit you can read related article.


openerp user
============

My openerp user is just a normal user (not a system user) who can login to the system. I choose to have it this way since I login as openerp user and do required tasks with this uid. You should select a secure password. Some Linux experts thinks this is not secure enough, keep in mind.


Postgresql openerp role
=======================

openerp postgresql role is superuser in my installation, it is not required to but it is easier for me. Some Linux experts thinks this is not secure enough, keep in mind.


init.d script for autostart
===========================

I just found out that there is also a suitable init.d script for debian already "hidden" :) in the source of OpenERP server in the file /opt/openerp/v7/server/debian/openerp.init you can change the pathnames and use this file also.

Please give your feedback about instructions above so I can enhance it.







Source:
=======
http://help.openerp.com/question/2562/how-to-install-openerp-v70-on-ubuntu-1204-from-launchpad-repository/

