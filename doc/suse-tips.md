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

    brokenmodules=nouveau

- Boot that boot entry with F10

- After boot: Check `/etc/modprobe.d/50-blacklist.conf`




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

    vi ~/.thunderbird/*.default
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
