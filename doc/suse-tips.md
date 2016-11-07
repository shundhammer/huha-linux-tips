# HuHa's SUSE tips

(considerably less than the Ubuntu counterpart because I've been using SUSE for
so long that most things in a SUSE environment come natural to me)


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


### Truncate or wrap long lines in Emacs

    M-x toggle-truncate-lines



----

## SUSE internal - not useful for outside users

### suse.com IMAP server

- gwmail.nue.novell.com
- gwmail.emea.novell.com



### IMAP / Sieve

    sudo zypper in python-managesieve
    sieveshell --user=sh --port=4190 --tls imap-int.suse.de

