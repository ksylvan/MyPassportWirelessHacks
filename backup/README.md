# Backup up a server using your My Passport Wireless device.

These instructions are based on my [original post](http://community.wd.com/t5/My-Passport-Wireless/Backing-up-between-two-My-Passport-Wireless-devices/td-p/811481) about using one MyPW device to backup another, on the WD Community pages.

## Setting up non-interactive ssh

When you ssh into the MyPW device, you notice that there's a pretty
full featured Debian/Linux system in there (*THANK YOU Western Digital
--- so awesome!*) but a few things need to be adjusted.
 
First thing: ssh requires strict permissions on the home directory for
key based (non-interactive) logins to work.

So, make sure the machine that you are trying to backup has the right
permissions on the home directory and the .ssh directory. The My
Passport Wireless device, for example, does not have the right
permissions for its root user. Fix it by doing this:

    # chmod 755 /home/root

Next, generate the keys and set up the authorized_keys file on the
receiving side (the My Passport Wireless device that will initiate the
rsync backups):

    cd /home/root
    ssh-keygen
    ssh-keygen -t dsa
    ssh-keygen -t ecdsa
    cat .ssh/id_rsa.pub | ssh 192.168.1.101 'mkdir .ssh; cat >> .ssh/authorized_keys'

Replace 192.168.1.101 above with the IP address on the server that you
are going to back up.

Once you have done this, you should be able to ssh to the other box
from your My Passport Wirless shell prompt:

    # ssh 192.168.1.101
    
    # ifconfig | grep inet
    inet addr:127.0.0.1  Mask:255.0.0.0
    inet addr:192.168.1.101  Bcast:192.168.1.255  Mask:255.255.255.0

Now you are ready to set up cron to run your rsync script.

## Cron and rsync backups

Copy ./backup script from this repository to your My Passport Wireless
device's DataVolume partition.

On your device:

    # mkdir /DataVolume/Backup

and put the script as /DataVolume/Backup/backup

    $ scp ./backup root@{ip-address-of-your-MyWP}:/DataVolume/Backup/

Now edit the top of the script with the right IP addresses and
locations and run it like this:

    # chmod +x ./Backup/backup
    # ./Backup/backup setup
    Shutting down crond services: done
    Starting crond services: done

After this, your backups should happen at the specified times.
 
Putting the backup script on the DataVolume is important, because it
won't be lost when you do firmware upgrades.
 
Test it by running the script by hand once (after which the rsync will
happen much faster).
