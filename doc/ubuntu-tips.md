# HuHa's Ubuntu Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Getting a freshly installed Ubuntu up and running


### Install bare minimum tools

    sudo apt-get install vim ssh aptitude synaptic muon
    sudo apt-get install aptitude synaptic muon
    sudo apt-get install zsh mmv emacs gnuserv
    sudo apt-get install exif exiftran exiftags jhead id3v2
    sudo apt-get install vlc mplayer
    sudo apt-get install git gitk colordiff automake cmake

### Change default editor from 'nano' to 'vi':


    sudo update-alternatives --config editor


### Allow 'sudo' without password for yourself:

    sudo visudo

Add line (at the end of the file - AFTER any 'include') with:

    myusername  ALL=(ALL) NOPASSWD: ALL


### Set a password for root to have a fallback if your user can't log in any more:

    sudo passwd root


### Enable Ctrl-Alt-Backspace to kill the X server:

    sudo dpkg-reconfigure keyboard-configuration

Hit [Return] 5 times until the "enable Ctrl-Alt-Backspace" dialog appears.


### Problem with ReiserFS partitions at boot time:

Maybe fsck.reiserfs missing

    sudo apt-get install reiserfsprogs


-----

## Booting


### Change display manager back to lightdm default (unity) theme:

    sudo dpkg-reconfigure --force lightdm
    sudo service lightdm restart

(config: `/etc/lightdm/lightdm.conf`)


### Change boot screen:

    sudo update-alternatives --config default.plymouth
    sudo update-alternatives --config text.plymouth
    sudo update-initramfs -u -k all

http://wiki.ubuntuusers.de/Plymouth



### Show boot messages:

    sudo vi /etc/default/grub

change    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`
to        `GRUB_CMDLINE_LINUX_DEFAULT=""`

    sudo update-grub


### Change console / grub resolution:

    sudo vi /etc/default/grub

    GRUB_GFXPAYLOAD_LINUX=1600x900

    sudo update-grub

If that doesn't work:

    sudo vi /etc/grub.d/00_header

locate the line with

    set gfxmode=${GRUB_GFXMODE}

and add a new line with

    set gfxpayload=keep


Reference:

- http://askubuntu.com/questions/127851/change-boot-screen-resolution
- http://www.ubuntu-forum.de/artikel/48665/auflösung-der-konsole-ändern.html
- http://www.intdblog.com/2009/09/grub-2-graphical-boot-tips-to-set.html


### Fix Plymouth boot screen after installing NVidia drivers:

    sudo vi /etc/default/grub

    GRUB_GFXPAYLOAD_LINUX=1920x1200
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset \
      video=uvesafb:mode_option=1920x1200-24,mtrr=3,scroll=ywrap"

    sudo update-grub
    sudo update-initramfs -u -k all


### Change console font:

    sudo dpkg-reconfigure console-setup

For 1600x900, use "Terminus Bold" in 20x10.
Otherwise, "Fixed" (the standard VGA font) is a good choice.


### Change Grub font:

    sudo grub-mkfont -s 24 -o /boot/grub/deja.pf2 \
      /usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf

Add that font to /etc/grub.d/00_header:

    sudo vi /etc/grub.d/00_header

    GRUB_FONT=/boot/grub/deja.pf2

Build complete grub.cfg from all the snippets in /etc/grub.d:

    sudo grub-mkconfig -o /boot/grub/grub.cfg


### Debug and configure suspend-to-RAM:

https://www.kernel.org/doc/Documentation/power/basic-pm-debugging.txt
http://wiki.ubuntuusers.de/pm-utils

Sony Vaio VGX-TP1E: Use method 'shutdown'.
Add to `/etc/rc.local`:

    echo shutdown >/sys/power/disk




-----

## Desktop Environment


### Load ~/.Xdefaults etc.:

Create `~/.xprofile` with

    USRRESOURCES=$HOME/.Xdefaults

(yes, USRRESOURCES, not USERRESOURCES)


### Get rid of unwanted directories in $HOME after each login:

    vi ~/.config/user-dir.dirs


### Disable X screensaver completely:

    cat noscreensaver
    #!/bin/sh
    xset -dpms s off s noblank s noexpose

Add this to Autostart (Xfce: settings -> session -> autostart)


### Use function keys directly on Logitech K400 keyboard:

    sudo apt-get install solaar


### Send middle mouse click upon key press:

    sudo apt-get install xdotool

map key to

    xdotool click 2

in XFce: Map Windows key (Super_L)


### Fix video output tearing for Intel graphics

    cd /usr/share/X11/xorg.conf.d

Create file `20-intel.conf`:

    Section "Device"
        Identifier "Intel Graphics"
        Driver "intel"
        Option "AccelMethod" "SNA"
        Option "TearFree"    "True"
    EndSection

Restart X11 (Ctrl-Alt-Backspace)

Check in Xorg.log:

    grep -i TearFree /var/log/Xorg.0.log

    [  8758.350] (**) intel(0): Option "TearFree" "True"
    [  8758.354] (**) intel(0): TearFree enabled



-----

## KDE 4


### Avoid excessive (128 MB) Akonadi logging

in `~/.local/share/akonadi/db_data/ib_logfile*`:

    akonadictl stop
    sudo vi /etc/akonadi/mysql-global.conf

Search for

    innodb_log_file_size=64M

Replace with

    innodb_log_file_size=1M

then

    akonadictl start


### Fix: KMail / Kontact refuses to start because of Akonadi problems

    sudo apt-get remove --purge apparmor


### Fix: KMail / Kontact refuses to start because of "resource collection" problems

    akonadictl stop
    rm -rf ~/.config/akonadi
    akonadictl start


### Change Gwenview fullscreen background to black instead of dirty brown:

    sudo apt-get install imagemagick

    cd /usr/share/kde4/apps/gwenview/images
    sudo mv background.png background-bullshit.png
    sudo convert -size 256x256 xc:black black-background.png
    sudo ln -s black-background.png background.png

Using a symlink prevents the created black background from being overwritten
upon the next package update of Gwenview: It will just clobber the symlink
which can easily be restored.




-----

## Package management


### Find fastest Ubuntu mirror:

- Start `muon`
- Settings -> software sources
- Select mirror "other"
- "Find fastest mirror"


### Get rid of messages about available updates:

    sudo rm /var/lib/update-notifier/updates-available


### Get rid of "reboot required" notice after apt-get upgrade:

    sudo rm /var/run/reboot-required


### Get rid of complaints about flashplugin-installer failure:

    sudo rm /var/lib/update-notifier/user.d/data-downloads-failed-permanently


### Exclude package during apt-get upgrade:

    sudo apt-mark hold <pkg>

Later:

    sudo apt-mark unhold <pkg>

Show:

    sudo apt-mark showhold



### Find out installation candidate package

    sudo apt-cache policy <pkg-name>



### Find out what packages were manually installed:

    aptitude search '?installed ?not(?automatic)' --disable-columns -F '%p' | \
    sort \
    >/tmp/aptitude-manual-pkg.txt

Remove those from the installation media manifest (that were part of the
default installation): Insert the installation USB stick / CD-ROM and:

    find /media/.../...ubuntu...  -name "*manifest*"
    cd <that directory>

    awk '{ print $1 }' filesystem.manifest | \
    sed -e 's/:amd64//' | \
    sort \
    >/tmp/manifest-pkg.txt

    comm -13 /tmp/manifest-pkg.txt /tmp/aptitude-manual-pkg.txt


### Fix complaints about invalid repo key for Opera:

http://www.ubuntuupdates.org/ppa/opera

    wget -O - http://deb.opera.com/archive.key | sudo apt-key add -

Check `/etc/apt/sources.list.d` for duplicates!



-----

## Kernel / hardware


### Enable keyboard beeper:

- Un-blacklist `pcspkr` from `/etc/modprobe.d/blacklist.conf`

- Start `alsamixer` in terminal
  - Move right to column 'beep'
  - Un-mute the channel ('m')
  - Turn channel volume up (~2-5%)


### Fix: Laptop brightness control not working

    sudo vi /etc/X11/xorg.conf

    Section "Device"
        ...
        Option "RegistryDwords" "EnableBrightnessControl=1"
    EndSection


or create a file in `/usr/share/X11/xorg.conf.d`, e.g. 
`12-nvidia-brightness.conf`:

    Section "Device"
        Identifier  "Default Device"
        Driver      "nvidia"
        Option      "RegistryDwords" "EnableBrightnessControl=1"
    EndSection

The section **must** have an _Identifier_ line.


Restart X11: Ctrl-Alt-Backspace, if enabled, otherwise log out and in, or

    service lightdm restart

(Kubuntu also uses LightDM!)

If there is no `/etc/X11/xorg.conf`, `nvidia-settings` can create one.

Manual check if it worked:

    su
    echo 15 >/sys/class/backlight/acpi_video0/brightness
    echo 3  >/sys/class/backlight/acpi_video0/brightness

(values 0..15 are accepted)

If that doesn't work:

    sudo vi /etc/default/grub

      GRUB_CMDLINE_LINUX_DEFAULT="quiet splash ...  acpi_backlight=vendor"

    sudo update-grub
    sudo reboot




-----

## Applications


### Force MythTV to prompt for language again:

    mythfrontend -p


### Fix Emacs complains about:

    ** (emacs:6834): WARNING **: Couldn't register with accessibility bus: Did not
       receive a reply. Possible causes include: the remote application did not
       send a reply, the message bus security policy blocked the reply, the reply
       timeout expired, or the network connection was broken.

Fix:

    export NO_AT_BRIDGE=1



-----

## Removable media


### Assign volume label to VFAT partition / USB stick:

    fatlabel /dev/sdc1 'MyLabel'

(from package `dosfstools`)


Alternative:

    sudo mlabel -i /dev/sdc1 ::'MYLABEL'

Caution: This forces the label to be uppercase.


-----

## Multimedia

### Get DVDs to play

https://help.ubuntu.com/community/RestrictedFormats

Don't forget to activate `libdvdcss`:

    sudo /usr/share/doc/libdvdread4/install-css.sh


-----

## Network

### Simple graphical network monitoring:


_EtherApe_ is a GUI to show network connections in a graph with a line for each
connection that corresponds to the amount of traffic.

![Etherape screenshot]
(http://etherape.sourceforge.net/images/v0.9.3.png)


Install:

    sudo apt-get install etherape

Using:

    xhost +
    sudo etherape
