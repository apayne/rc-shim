# Introduction #

---

The rc-shim script is meant to be used as a shim between an existing 
SysV-styled rc script system, and a daemontools-alike supervision 
system.  By utilizing an existing supervisor, some of the weaknesses of 
the rc system can be avoided.

The script takes the place of the original script, typically found in 
/etc/init.d, and performs a translation of the start/stop/restart/status 
commands into something the supervisor can use.



# Installation #

---

## Prerequisites #

1. You must have a supervision suite installed.  Currently, runit and s6 
are supported, but nearly any daemontools-styled supervisor that 
supports launching via service definition directories should work.  
Specifically, the supervision suite must support a service scan 
directory.

2. You must have a set of supervision definitions installed.  The 
definitions must preside separately of both the directory containing the 
symlinks, and the directory where the supervisor states are maintained.

3. You will only be able to use the shim with rc scripts that launch a 
single daemon.  Scripts that launch multiple daemons (on Linux, 
typically Samba or the bluetooth stack, for example) will require 
additional setup to be used properly.

## Installation Steps #

1. Download the shim to a separate location.

2. Copy the os-settings and supervisor-settings files to where your 
init.d scripts are located.  They must all be in the same directory.  
Example:

    sudo cp os-settings /etc/init.d/
    sudo cp supervisor-settings /etc/init.d/

3. Adjust os-settings to match your installation.  If you do not change 
the default settings, the shim will fail and refuse to run.  This is by 
design.

4. Adjust supervisor-settings to match your supervisor's installation.  
There are presets embedded in the file which can be used by uncommenting 
the appropriate entries.  If you do not change the default settings, the 
shim will fail and refuse to run.  This is also by design.

5. At the moment, only manual installation is supported.  For each rc 
script you wish to replace with a shim, rename the original, and create 
a copy of the shim in its place.  Example:

    sudo mv /etc/init.d/snmpd /etc/init.d/snmpd.original
    sudo cp shim /etc/init.d/snmpd

6. Test your results.

    sudo service snmpd restart

## How this works #

The basic idea behind "System-V styled" rc scripts is that a set of 
shell scripts will stop or start various services as needed.  The 
scripts are stored in a specific directory (on Linux systems, typically 
/etc/init.d) and each script will stop or start a service as needed.

When your system boots up, shuts down, or changes run levels, there is 
special directory corresponding to the runlevel, that contains a set of 
symbolic links.  These links have special names that indicate what 
sequence they are called in, and if a service is to be stopped or 
started.  The symlinks themselves point to the rc scripts, and the 
entire system is simply calling each rc script with the corresponding 
stop or start command using the corresponding symlink.

The shim takes advantage of the fact that the links merely point to the 
appropriate script.  It functions by replacing the rc script with 
something that is designed to talk to a supervision suite.  Instead of 
attempting to start or stop the service directly, the shim interprets 
the "start" and "stop" commands and then sends signals to the supervisor 
to take the appropriate action, as well as maintaining any symbolic 
links in the supervisor's scan directory.  The supervisor then handles 
the start/stop command on behalf of the shim.  Because the existing 
symlinks in each runlevel will continue to point to the shim, everything 
else - including the use of the "service" command - functions normally.

This should be fully compatible with system startup, shutdown, and 
runlevel changes.  The only change that takes place is that instead of 
simply recording the PID file for later use and hoping for a successful 
start, a full supervisor will be employed, along with whatever 
advantages it conveys.  As far as the rest of the system is concerned, 
the shim is just another rc script.


## Troubleshooting #

Here are some considerations to help you fix a problem or two:

0. *Do you have a way to access your system in the event that a shim 
malfunctions?* Because the rc scripts are called at boot time and 
shutdown, it is possible to really foul things up to the point you don't 
have a working system you can access.  While I have tried to make the 
shim as safe as I can, there are still circumstances beyond my control.  
This includes things like having a malformed run script for ssh on a 
remote system, and then discovering after reboot that the shim can't 
start the service because the run script is wrong, completely locking 
you out in the process and requiring a trip to the physical machine 
itself.  This unpleasant situation can be avoided with testing. Test 
each entry you change out by manually starting and stopping the service, 
and then test it again.  Wait until you are 100% satisfied with the 
results before changing out the rc script.  Or to put it in a shorter 
way: *If in doubt, don't switch it out!*

1. *Do you have a set of service definitions, active services, and a 
service control directory?* The shim only provides an interface to the 
existing service manager, not service definitions themselves.  You must 
supply your own run scripts for the shim to have something to work with.

2. *Did you change all of the settings correctly?*  If you fail to change 
either of the settings files, the scripts will immediately fail.  This 
is by design and was meant to prevent the script from making an 
assumption about the state of the system it was installed on by forcing 
the systems administrator to specify it.

3. *Is the rc script name different from the actual service name?* The 
script will attempt to guess the daemon's name using the name of itself, 
but this is not always successful because there is no requirement that 
the name of the rc script match the actual daemon name.  A common 
example is the ISC bind service having the rc script named as "bind", 
but the daemon is "named".  The shim has a provision for this, you can 
simply edit that specific copy of the shim to have the correct name. 
Locate the line in the shim that looks like this: *SVCNAME=$MYNAME*.  
Using our "bind/named" example, replace "$MYNAME" with "named".  This 
skips the "guess" that the script takes and explicitly identifies the 
name of the daemon itself.  Of course, the service definition directory 
will have to have the same name for this to work correctly.

4. *Does the rc script actually try to launch multiple daemons?* An 
example would be Samba, which usually has two daemons launched together, 
smnd and nmbd.  Many installations will provide a single rc script to 
launch both at the same time, typically named "smb" or "samba".  This is 
contrary to how supervision works because a supervisor is usually 
assigned to a specific daemon, and not a group of daemons.  There is a 
work-around for this that requires a bit of work but will allow the shim 
to continue to function, while fully integrating with the supervisor. 
You will need to create a service definition that consists of a service 
scan, and then place each daemon needed as a definition into the new 
service scan directory.  Then when you use the shim, you will launch the 
service scan, not the individual daemons.  By using a nested service 
scan, the emulation of a "multi daemon launch" is maintained, at the 
expense of the increase in the size fo the supervision tree.  In this 
example, you would create a "samba" definition to match the naming of 
the original rc script, and the daemon launched is acutally another 
service scan.  That service scan will then have in it the service 
definitons for smbd and nmbd.  Whenever the service is brought up, the 
service scan will bring up the other two with it.  Please note that this 
weakens the the assurance you receive of a correct start in the same way 
that an asynchronous start would, i.e. you are assuming the supervisor 
will deal with all of the issues and will not be able to directly see 
the results. Also note that any status report you will receive will be 
for the service scan and not the individual service.
