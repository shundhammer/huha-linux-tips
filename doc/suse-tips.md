# HuHa's SUSE tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License

(considerably less than the Ubuntu counterpart because I've been using SUSE for
so long that most things in a SUSE environment come natural to me)



## Getting a freshly installed SUSE up and running

### Allow 'sudo' without password for yourself:

    sudo visudo

Add line (at the end of the file - AFTER any 'include') with:

    myusername  ALL=(ALL) NOPASSWD: ALL

To keep the shell environment: Locate the line with `env_reset`

    Defaults    !env_reset



----


## Kernel and kernel modules

### Compile your own NVidia driver

    echo "blacklist nouveau" >> /etc/modprobe.d/50-blacklist.conf

Add kernel command parameter:

    nouveau.modeset=0


### Disable kernel module at Grub2 prompt

- At Grub2 prompt: Select entry to boot, then hit 'e' (edit)

- Add to the kernel command line:

      brokenmodules=nouveau


- Boot that boot entry with F10

- After boot: Check `/etc/modprobe.d/50-blacklist.conf`




----

## ssh

### Can't ssh as root Anymore

This affects mostly openSUSE Tumbleweed since mid-2021.

- Log in to that remote machine as a normal user

- Edit `/etc/sshd_config` (notice the `d`! not `/etc/ssh_config`,
  `/etc/sshd_config`!) with root privileges:


      ssh mynormaluser@mytumbleweed
      cd /etc/ssh
      sudo vi sshd_config

- Add

      PermitRootLogin  yes

- Make sure to add it at the very end of the file to avoid any include files
  overriding your explicitly requested change again. They love to do that (also
  in `/etc/sudoers`).

Disabling this by default is considered a security "feature" by those who keep
making everything more and more unusable for people actually trying to **work**
with Linux machines.

They also make work pretty much impossible for system-level Linux developers
who need to work with virtual machines, so they need root access to their VMs
really frequently. Fiddling with ssh keys etc. is not a viable option here;
this is much too complicated and too error-prone to set up.

**I am the fucking boss on my machine, and I get root access to VMs on it
whenever the fuck I please. Do you read me, security folks?** The same applies
to all my machines in my home network.

It's nobody's fucking business to make life even harder for users even in their
home networks where nobody can log in from the outside to begin with. All you
do is encourage people to migrate to BSD or to MacOS X or back to Windows.


----

## X11 / Desktop

### Don't Boot Systemd into Graphical Login

If X11 or Wayland don't work, boot into a text mode login.
From the Grub prompt:

    systemd.unit=multi-user.target


### Get Rid of Ever-Growing ~/.xsession-errors

#### Symptom

The `.xsession-errors` file in your home directory keeps growing and growing
(it can easily exceed 150+ MB).

#### Background Information

Most of that are pointless Gtk+ errors that you can't do anything about anyway;
that toolkit just appears to be changing all the time with application
developers never being able to catch up, so it clutters stderr with all kinds
of errors and warnings which for X11 applications end up in
`~/.xsession-errors`, cluttering your home directory and filling up your disk.


#### Quick Fix

    echo >~/.xsession-errors

**Do not** just remove it:

    rm ~/.xsession-errors     # WRONG!

This would still keep the disk space allocated because active processes (your X
server!) are still keeping that file open, so it would just disappear from the
directory, but the disk blocks are only freed once the last process keeping it
open exits. If you keep your X11 session running, this will not have any
effect.

Don't worry if shortly after the `echo >~/.xsession-errors` command it keeps
coming back; it is now just a _sparse file_ that allocates only disk space for
new messages, i.e. whenever the next message is written to the file, it appends
to the file, but it leaves the disk blocks at the start of the file untouched
and unallocated.

It is still most annoying, though.


#### More Permanent Fix

    cd /etc/X11/xdm
    sudo vi Xsesssion

- Locate that block

```bash
#
# Redirect errors to the standard user log files.
#

for errfile in "${HOME}/.xsession-errors" \
               "${TMPDIR:-/tmp}/xerr-${USER}-${DISPLAY}"
do
    stderr=$(readlink -fs /dev/stderr)
    ...
    ...
    ...
done
unset tmpfile errfile
```


The safest thing is to comment that block out completely.
After it, add

    exec >/dev/null
    exec 2>/dev/null

This just redirects both stdout and stderr of that process (the X session) to
`/dev/null` without cluttering your home directory.

- Restart X11.

- You can now safely delete the old `.xsession-error` file(s):

```
rm ~/.xsession-error*
```

They should not come back.

Since `/etc/X11/xdm/Xsession` is a config file, the change should survive
package updates.  That file is not only used by _XDM_, but also by _LightDM_
(tested and verified).

Not sure if it also works with _KDM_ and _GDM_ (please let me know), but if
not, they should have an equivalent file where you can change the error log
file.


----

## Package management

### Allow vendor change in Zypper

https://en.opensuse.org/SDB:Vendor_change_update

    vi /etc/zypp/zypp.conf

Add

    solver.allowVendorChange = true


### Zypper Documentation

How-To:

  https://en.opensuse.org/SDB:Zypper_usage

SLES Manual:

  https://documentation.suse.com/sles/15-SP1/html/SLES-all/cha-sw-cl.html#sec-zypper
  https://documentation.suse.com/sles/15-SP1/html/SLES-all/cha-upgrade-online.html#sec-upgrade-online-zypper-plain


----

## Applications

### Change Thunderbird "Trash" folder

    cd ~/.thunderbird/*.default
    vi prefs.js

Add for the correct (!) account:

    user_pref("mail.server.server4.trash_folder_name", "Gelöscht");


### Fix Emacs complains about:

    ** (emacs:6834): WARNING **: Couldn't register with accessibility bus: Did not
       receive a reply. Possible causes include: the remote application did not
       send a reply, the message bus security policy blocked the reply, the reply
       timeout expired, or the network connection was broken.

Fix:

    export NO_AT_BRIDGE=1


### Truncate or wrap long lines in Emacs

    M-x toggle-truncate-lines



-----

## Removable media


### Assign volume label to VFAT partition / USB stick:

    fatlabel /dev/sdc1 'MyLabel'

(from package `dosfstools`)


Alternative:

    sudo mlabel -i /dev/sdc1 ::'MYLABEL'

Caution: This forces the label to be uppercase.


-------------

# Time Zones

## GUI: Use YaST

YaST lets you select your time zone from a world map:

    sudo yast2 timezone


## Text-driven menu: tzselect


```
[sh @ balrog-tw-dev] ~ 7 % sudo tzselect

Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
 1) Africa
 2) Americas
 3) Antarctica
 4) Asia
 5) Atlantic Ocean
 6) Australia
 7) Europe
 8) Indian Ocean
 9) Pacific Ocean
10) coord - I want to use geographical coordinates.
11) TZ - I want to specify the timezone using the Posix TZ format.
#? 7

Please select a country whose clocks agree with yours.
 1) Albania		  18) Hungary		    35) Portugal
 2) Andorra		  19) Ireland		    36) Romania
 3) Austria		  20) Isle of Man	    37) Russia
 4) Belarus		  21) Italy		    38) San Marino
 5) Belgium		  22) Jersey		    39) Serbia
 6) Bosnia & Herzegovina  23) Latvia		    40) Slovakia
 7) Bulgaria		  24) Liechtenstein	    41) Slovenia
 8) Croatia		  25) Lithuania		    42) Spain
 9) Czech Republic	  26) Luxembourg	    43) Svalbard & Jan Mayen
10) Denmark		  27) Malta		    44) Sweden
11) Estonia		  28) Moldova		    45) Switzerland
12) Finland		  29) Monaco		    46) Turkey
13) France		  30) Montenegro	    47) Ukraine
14) Germany		  31) Netherlands	    48) United Kingdom
15) Gibraltar		  32) North Macedonia	    49) Vatican City
16) Greece		  33) Norway		    50) Åland Islands
17) Guernsey		  34) Poland
#? 14

Please select one of the following timezones.
1) Swiss time
2) Germany (most areas)
#? 2

The following information has been given:

	Germany
	Germany (most areas)

Therefore TZ='Europe/Berlin' will be used.
Selected time is now:	Di 19. Okt 16:32:42 CEST 2021.
Universal Time is now:	Di 19. Okt 14:32:42 UTC 2021.
Is the above information OK?
1) Yes
2) No
#?
```


## Command line: zic

    sudo zic -l Europe/Berlin

Check with:

    [sh @ balrog-tw-dev] ~ 9 % ls -l /etc/localtime
    lrwxrwxrwx 1 root root 33 Okt 19 16:27 /etc/localtime -> /usr/share/zoneinfo/Europe/Berlin


## Check available time zones

    cd /usr/share/zoneinfo
    ls
    cd Europe
    ls
