# Intro #

---

The rc-shim script is meant to be used as a shim between an 
existing SysV-styled rc script system, and a daemontools-alike 
supervision system.

The script necessarily takes the place of the original script, typically 
found in /etc/init.d, and essentially performs a translation of the 
start/stop/restart commands into something the supervisor can use.

By utilizing an existing supervisor, and not leaving the daemon startup 
to chance (or worse, not having a monitor that detects when it crashes 
and the system still thinks it's up), two of the weaknesses of the rc 
system can be avoided.


# Installation #

---

There are three basic steps to installation: installing a supervisor,
installing the script and files, and changing the default settings.

### Step 1 #

It is assumed that you have a compatible supervisor installed,
including but not limited to daemontools, daemontools-encore,
runit, or s6.  **You must have a set of run scripts for each service
to be supervised, or the shim will not work.**

### Step 2 #

There are three necessary files.  Two files contain shell environment 
variables that are required to run the shim, and then there is the shim 
itself.

All three files need to be located at the init.d directory location.  
Typically this is at /etc/init.d for RedHat- and Debian-styled systems.  If
this is not the case, please send all three files to the location where
the actual rc control scripts live.

### Step 3 #

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


# Limitations #

---

You must have a definition for each daemon that will receive the shim.  
If there is no definition, then the shim will fail.

The script only supports three operations: stop, start, and restart.  A 
common operation is to request a reload of settings.  This is not 
supported at the moment.

While it may be possible to create a "reload" command, there is no 
consistent use of the SIGHUP signal to force a daemon to reload its 
settings, so there is no guarantee that this command will be 100% 
consistent across all daemons.
