Intro
---
The rc-svcshim script is meant to be used as a shim between an existing 
daemontools-styled supervisor and the venerable (but sometimes senile) 
rc.d service system. The goal of the script is to wedge itself between 
the supervisor and the existing init.d infrastructure on the system.  
The script necessarily takes the place of the original /etc/init.d 
script and essentially performs a translation of the start/stop/restart 
commands into something the supervisor can use.


Installation
---
There are three necessary files.  Two files contain shell environment 
variables that are required to run the shim, and then there is the shim 
itself.

All three files need to be located at the init.d directory location.  
Typically this is at /etc/init.d for RedHat- and Debian-styled systems.  If
this is not the case, please send all three files to the location where
the actual rc control scripts live.

Once installed, you will simply move the old script aside and copy the 
shim, using the same script name.  For example, to use the shim
with a service named "snmpd":

    cd /etc/init.d
    mv snmpd snmpd.orig
    cp shim snmpd

Because of the way the rc.d system works, the existing symlinks in each 
runlevel will continue to point to the correct name, and because we have 
replaced the script with the shim, any calls made during a runlevel 
change will result in the shim running instead.


Limitations
---
The script only supports three operations.  A common init.d operation
is to request a reload of settings.  This is not supported at this time.

While it may be possible to create a "reload" command, there is no 
consistent use of the SIGHIP signal to force a daemon to reload its 
settings, so there is no guarantee that this command will be 100% 
consistent across all daemons.
