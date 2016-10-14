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

There are two pre-requisites and two basic steps to installation: 
installing a supervisor, installing service definitions, installing the 
script and files, and changing the default settings.

### Prerequisite 1 #

You have a compatible supervisor installed.  Currently, runit and s6 are 
supported but nearly any daemontools-styled supervisor should work as 
well, including daemontools, daemontools-encore, etc.

### Prerequisite 2 #

You have service definitions available for each service that will 
receive the shim.  The defintions will reside at a "service definition 
directory", which acts as a read-only repository.  There are a few of 
these available in various forms, but creating them is beyond the scope 
of this document.  The best I can recommend is to write them yourself.

There are a few projects and collections out there if you search, and 
you are impatient or desperate.


### Step 1 #

There are three necessary files.  Two files contain shell environment 
variables that are required to run the shim, and then there is the shim 
itself.

All three files need to be located at the init.d directory location.  
Typically this is at /etc/init.d for RedHat- and Debian-styled systems.  If
this is not the case, please send all three files to the location where
the actual rc control scripts live.

If you do not place the two settings file in the same location as the 
shim, the shim will fail.

### Step 2 #

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
