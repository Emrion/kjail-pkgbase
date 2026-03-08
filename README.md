# 1) Description  

This is kjail-pkgbase, the next version of kjail. It's a set of jail management tools that uses exclusively pkgbase. kjail manages only the jails created by itself. It works only with official supported RELEASE(s) and not below 15.0-RELEASE.  
**It requires ZFS.**

Improvements!
- The snap system now can rollback any save, not only the last one.
- It's possible to configure VNET jails including those that use DHCP.
  
# 2) Installation  

Copy the files kinst and kjail.txz at the same place (this place has no importance). Then, chmod +x kinst and run it as root user.  
kinst will ask you on which pool you want to install kjail and then run the following commands (\*kpool\* = choosen pool):

`zfs create *kpool*/kjail`  
`zfs create *kpool*/kjail/base`  
`zfs create *kpool*/kjail/jails`  
`mkdir /*kpool*/kjail/settings`   
`tar -xf kjail.txz -C /*kpool*/kjail`  
`sed -i "" -e "s#/\*\*\*/#/*kpool*/#" /*kpool*/kjail/kjail (adjustement of the location of kjail inside the rc script)`  
`cp /*kpool*/kjail/kjail /usr/local/etc/rc.d/`  



# 3) Deinstallation  

Run the following commands:

`# service kjail onestop`  
`# rm /usr/local/etc/rc.d/kjail`  
`# zfs destroy -rRf *kpool*/kjail`  


# 4) Basic usage  

All commands need the root privilege to be run.
To execute a "command", you can run it with /\*kpool\*/kjail/"command" or service kjail one"command", if you didn't sysrc kjail_enable=YES. If you did, it's just service kjail "command".

To begin with, you need to load a base. loadbase will load the version the host is using. It must be a RELEASE version. It works whether the host is using pkgbase or not. The installed system is FreeBSD-set-base-jail (see pkgbase(7)).  
`# /*kpool*/kjail/loadbase`  

After that, you can create any number of jail you want:  
`# /*kpool*/kjail/jcreate JailName`  
An ipv4 address will be asked (or 'DHCP'), but it's optional.
You can opt for a VNET jail. If you selected DHCP, the jail is forcibly VNET.

You will also be asked if the jail goes on latest repos or stays on quarterly.  

You may want to edit the jail configuration file and/or the associated fstab:  
- /\*kpool\*/kjail/jails/JailName/JailName.conf
- /\*kpool\*/kjail/jails/JailName/JailName.fstab

You start the jail with:  
`# /*kpool*/kjail/start JailName`  

You get a command line into with:  
`# /*kpool*/kjail/cons JailName`  

There, you can install the needed softwares, add users, modify configuration files and so on.  

If you want that this jail be launched automatically at boot, its name needs to be present in /\*kpool\*/kjail/kjail.conf, line jails_list="...". The question is asked at each jail creation. But, after creation, you just have to edit this file to add or remove jail name from the list. Note that jails will be run at boot only if there is kjail_enable=YES in /etc/rc.conf.  


# 5) Principles  

It's a nullfs based jail. There is one base for all jails. These jails use that base but in read-only mode. A nullfs mount is set between each jail (/\*kpool\*/kjail/jails/JailName/root/base) and /\*kpool\*/kjail/base. Soft links are set for numerous system directories (bin boot lib libexec sbin usr/bin usr/include usr/lib usr/libdata usr/libexec usr/sbin usr/src usr/share).

Jails have obviously their own settings in /etc/ and their own softwares/settings in /usr/local.

Basically, when you update the base, you update all the jails. It's more or less the same thing concerning a RELEASE upgrade, although it's more complex in details.

All the jails resides in /\*kpool\*/kjail/jails/. When you create a jail, a new dataset \*kpool\*/kjail/jails/jailName is created, and is automatically mounted in the dir /\*kpool\*/kjail/jails/jailName. Inside this directory, you have JailName.conf which is the configuration file for this jail. You have also JailName.fstab where you can mount what you need for the jail. Finally, the /\*kpool\*/kjail/jails/jailName/root is the "/" of that jail.

When a jail is created, the whole content of /*kpool*/kjail/settings is copied in the root dir of this jail. You can put some files of the host or of your own like /etc/localtime, /etc/crontab, /etc/periodic.conf, /etc/rc.conf...

The change of RELEASE (upgrade) is a real problem for nullfs jails because this requires the update of the config files in each jail. To solve this, kjail uses etcupdate after the base upgrade has been completed. etcupdate needs the sources of the base system, and that's why loadbase fetches them.


# 6) kjail man  

- **bsnap**  
Manages snapshots concerning the base. Note that a snapshot named "ok" is automatically created before any update or upgrade. And the previous "ok" snapshot, if any, is deleted.  
&emsp;`bsnap [create|destroy|rollback] SnapshotName`  
&emsp;`bsnap list`  
All commands can be abbreviated like c* = create, d* = destroy, r* = rollback and l* = list. It's just the first letter that matters.  

- **cons JailName**  
Launches a command line into the specified jail. It must be running. 

- **jcreate JailName**  
Creates a new jail. It's mainly interactive.

- **jdestroy JailName**  
Destroys the jail. This removes all its data.

- **jlist**  
Lists all the jails belonging to the kjail software. Those that are running and those that aren't.

- **jmend JailName**  
Tries to repair a jail after a RELEASE upgrade. Typically, that jail wasn't running during the upgrade process and needs at least an `etcupdate`.

- **jrename OldName NewName**  
Renames a jail.

- **jsnap**  
Manages snapshots concerning the jails. Note that a snapshot named "ok" is automatically created before any "etcupdate" (after the upgrade of the base). And the previous "ok" snapshot, if any, is deleted.  
&emsp;`jsnap [create|destroy|rollback] JailName SnapshotName`  
&emsp;`jsnap list JailName`  
All commands can be abbreviated like described for bsnap. 

- **jup [JailName]**  
Performs a pkg upgrade on the jail JailName or on all jails that are running if no JailName is provided. A snapshot named "ok" is automatically created for the jail before pkg upgrade, and the previous "ok" snapshot, if any, is deleted.

- **loadbase**  
Install the same RELEASE as the host and its sources into /*kpool/kjail/base/.

- **restart JailName**  
Runs a stop and a start of JailName.

- **start JailName**  
If this command is launched from the kjail service without argument, all jails belonging to the kjail starting list are started.

- **stop JailName**  
If this command is launched from the kjail service without argument, all jails belonging to the kjail system and currently running are stopped.

- **update**  
Performs an update (security patches) of the base system. It's advised to update the host system before. Once the base system is updated, this restarts all the kjail jails that were running.

- **upgrade XX.x-RELEASE**  
Performs an upgrade - minor or major - of the base system. The host system has to be at least equal to XX.x-RELEASE. Each running jail will execute `pkg-static upgrade -f` if this is a major RELEASE upgrade. In any case, all running jails will run "etcupdate" at the end of the process.

You can have some requests from `etcupdate resolve`. In general, you have to select (e) to edit the file, correct it with vi commands, and then (r) to indicate that the conflict has been resolved.
