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




----


## Kernel and kernel modules

### Compile your own NVidia driver

    echo "blacklist nouveau" >> /etc/modprobe.d/50-blacklist.conf

Add kernel command parameter:

    nouveau.modeset=0


### Disable kernel module at Grub2 prompt

- At Grub2 prompt: Select entry to boot, then hit 'e' (edit)

- Add to the kernel command line:

    ```
    brokenmodules=nouveau
    ```


- Boot that boot entry with F10

- After boot: Check `/etc/modprobe.d/50-blacklist.conf`




----

## X11 / Desktop

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




----

## SUSE internal - not useful for outside users

### suse.com IMAP servers

- gwmail.nue.novell.com
- gwmail.emea.novell.com



### IMAP / Sieve

    sudo zypper in python-managesieve
    sieveshell --user=sh --port=4190 --tls imap-int.suse.de
