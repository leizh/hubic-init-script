Overview
--------
This is an init script to run the [hubiC](https://hubic.com) client as a service 
in a non graphic environment. The service runs as the `hubic` user.
For the uninitiated, hubiC is a cloud storage provider that offers very reasonable
prices (5 euro/month for 10TB) for backup needs and a Linux client that can run on the server.

Features
--------
 * start|stop|restart
 * reload : update the sync directory and the exclusions list
 * supports backup of directories (after manual configuration)
 * supports synchronization
 
Dependencies
------------
 * [hubiC linux client](https://hubic.com/en/downloads)
 * dbus-launch (package dbus-x11)

Installation
------------
On Debian and Ubuntu, run these commands as root:

    git clone https://github.com/leizh/hubic-init-script.git
    . hubic-init-script/build-deb.sh
    dpkg -i hubic-init-script_0.3_all.deb

This will install the script ready to be run at boot-time, but you will need to 
edit the following three files to set various config values:
```
/etc/default/hubic  
/etc/hubic/exclusions  
/etc/hubic/password
```

## Debugging errors
Consider trying out the standalone client before debugging any errors with the init script:
```
# start a shell as the hubic user
sudo su -l -s /bin/bash hubic

# export the DBUS environment variable
export $(cat /run/hubic.run )

# check that the dbus daemon is actually running
hubic status

# if not, start the daemon like this
dbus-lanch --sh-syntax > dbus.dat
source dbus.dat && rm dbus.dat

# you might get "Command failed: System.InvalidOperationException: Already connected." on this. Not dangerous :)
hubic --password_path=/etc/hubic/password $EMAIL "$SYNC_DIR"

# you should be able to get meaningful output at this stage
hubic status
```

## Backing up directories
These directories need to be readable for the `hubic` user! We are also assuming hubic has been 
configured and found working at this point. This configuration only needs to be done once, as
the everything will load automatically on each boot after this.

```
# Read the DBUS session environment variable
export $(cat /run/hubic.run )

# check existing backups
sudo -u hubic hubic backup info
       Name  Attached  Local path  Last backup      Size
     Bilder        No           -            -  12.87 GB
musikkarkiv        No           -            -       0 B
       priv        No           -            -  15.93 GB

# attach the ones that are accessible on this machine, so that they can continue
sudo -u hubic hubic backup attach Bilder /mnt/data/Bilder
sudo -u hubic hubic backup attach priv /mnt/backup/priv/
sudo -u hubic hubic backup attach musikkarkiv /mnt/data/musikkarkiv/

# create any other new backups you might need
sudo -u hubic hubic backup create  --frequency=weekly /mnt/data/video_archive/
```
