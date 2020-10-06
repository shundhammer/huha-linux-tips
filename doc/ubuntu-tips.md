# HuHa's Ubuntu Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Getting a freshly installed Ubuntu up and running


### Install bare minimum tools

    sudo apt-get install vim ssh aptitude synaptic muon
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


### Don't log every `sudo` command:

    sudo visudo

Add a line

    Defaults !syslog


### Keep the shell environment during `sudo`

    sudo visudo

Locate the line with `env_reset`

    Defaults    !env_reset



### Set a password for root to have a fallback if your user can't log in anymore:

    sudo passwd root


### Enable Ctrl-Alt-Backspace to kill the X server:

    sudo dpkg-reconfigure keyboard-configuration

Hit [Return] 5 times until the "enable Ctrl-Alt-Backspace" dialog appears.


### Disable forced NumLock

_Whoever came up with the brain-dead idea to force this to "on" should be
shot!_

    sudo apt-get remove numlockx


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


### Reduce insane systemd timeout from 30 sec to reasonable value

    cd /etc/systemd
    sudo vi system.conf

Uncomment and change

    DefaultTimeoutStopSec=10s

?? The same values are also in user.conf; maybe change them there as well.

See also

  https://unix.stackexchange.com/questions/227017/how-to-change-systemd-service-timeout-value

-----

## Shell

### zsh: Numeric keypad Enter key not working

Add this to `~/.zshrc`:

    bindkey -s "^[OM" "^M"


How to find out the key codes:

In zsh, hit `Ctrl-V` and then they key.



## Desktop Environment

### SysRq Key

#### Enable SysRq Key

Just for this session:

    su
    echo 1 >/proc/sys/kernel/sysrq

Permanently: Create a new file /etc/sysctl.d/90-sysrq.conf :

    cd /etc/sysctl.d
    sudo vi 90-sysrq.conf

Add this line:

    kernel.sysrq=1


#### Using SysRq

- `Alt SysRq` `R` (_raw_) Switch keyboard from _raw_ mode, i.e. take it away
  from the X server. This will switch to a text console. Switch back to X with
  `Alt` `F7`.

- `Alt SysRq` `S` Sync all mounted filesystems.

- `Alt SysRq` `B` Reboot

- `Alt SysRq` `Space` Show a summary of SysRq keys

- `Alt SysRq` `REISUB` "Gentle" reboot:
   - `R`: Switch keyboard from raw mode
   - `E`: Send SIGTERM to all processes except init
   - `I`: Send SIGKILL to all processes except init
   - `S`: Sync all mounted filesystems
   - `U`: Remount all mounted filesystems in read-only mode
   - `B`: Reboot

See also https://en.wikipedia.org/wiki/Magic_SysRq_key


#### SysRq on a Lenovo X1 Carbon

Use `Fn S` + _command-key_
or `Ctrl Fn S` + _command-key_


### Browser Task Manager (Opera, Chrome)

Hit `Ctrl Esc` to show the sub-processes (including extensions) in Chrome-based
browsers including CPU and memory usage. You can kill individual ones from
there.


### Load ~/.Xdefaults etc.:

Create `~/.xprofile` with

    USRRESOURCES=$HOME/.Xdefaults

(yes, USRRESOURCES, not USERRESOURCES)


### Get rid of unwanted directories in $HOME after each login:

    vi ~/.config/user-dir.dirs


### Set font sizes of Qt (5.x) programs in Xfce or GNOME

Install "qt5ct", set the environment up to use it and set fonts with it:

    sudo apt-get install qt5ct
    export QT_QPA_PLATFORMTHEME=qt5ct
    qt5ct

Restart any running Qt 5.x programs so they are using that environment
variable. They should even change their settings on the fly when you hit the
"apply" button in qt5ct.


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


### Fix: X11 Freezing with NVidia Drivers

(Seen on Xubuntu 20.04 LTS with kernel 5.4.0-48 and nvidia-driver-450)

Check the syslog for this message:
_NVRM: GPU ...: GPU has fallen off the bus._

    sudo journalctl | grep "fallen off the bus"
    
    Okt 06 17:59:27 balrog kernel: NVRM: Xid (PCI:0000:01:00): 79, pid=1122, GPU has fallen off the bus.
    Okt 06 17:59:27 balrog kernel: NVRM: GPU 0000:01:00.0: GPU has fallen off the bus.

Try setting _persistent mode_ for the GPU. 

Preferred method: Use NVidia's _persistenced_ (from package _nvidia-compute-utils_).

- Check if it's already running:

    sudo systemctl status nvidia-persistenced

- Start it only once (won't auto-start after reboot with this method):

    sudo systemctl start nvidia-persistenced

- If it's not running, enable it:

    cd /etc/systemd/user/default.target.wants
    sudo ln -s /lib/systemd/system/nvidia-persistenced.service .

  As of 2020/10, this does _not_ work (systemd complains that it's missing a
  _wanted_ line):
  
    sudo systemctl enable nvidia-persistenced     # DOESN'T WORK

    

Old and deprecated method (won't survive a reboot):

    sudo nvidia-smi -pm 1

    


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

## Xfce

### Use a Better File Manager

Install PCManFM:

    sudo apt install pcmanfm



### Problem: PCManFM doesn't Show Thumbnails for Video Files

Install a thumbnailer for those files.

Check `/usr/share/thumbnailers` what MIME types are already supported:

    grep MimeType /usr/share/thumbnailers/*

If there is none for videos, install `ffmpegthumbnailer`:

    sudo apt install ffmpegthumbnailer



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

    cd /usr/share/gwenview/images
    sudo mv background.png background-bullshit.png
    sudo convert -size 256x256 xc:black black-background.png
    sudo ln -s black-background.png background.png

Using a symlink prevents the created black background from being overwritten
upon the next package update of Gwenview: It will just clobber the symlink
which can easily be restored.




-----

## Package management


### Find out what packages are installed:

    dpkg-query -f '${Package} ${db:Status-Want}\n' | grep install

with pattern:

    dpkg-query -f '${Package} ${db:Status-Want}\n' --show "*xfce4*" | grep install | sed -e 's/ install//'

(see `man dpkg-query` and then search _showformat_ for more variables)


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


### Selectively Disable Meltdown/Spectre Kernel Patches

https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/SpectreAndMeltdown/MitigationControls
https://askubuntu.com/questions/991874/how-to-disable-page-table-isolation-to-regain-performance-lost-due-to-intel-cpu
https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/mds.html

Disable all (**CAUTION!**):

    sudo vi /etc/default/grub

      GRUB_CMDLINE_LINUX_DEFAULT="... pti=off spectre_v2=off mds=off"

    sudo update-grub
    sudo reboot


-----

## Applications


### Fix Emacs complains about:

    ** (emacs:6834): WARNING **: Couldn't register with accessibility bus: Did not
       receive a reply. Possible causes include: the remote application did not
       send a reply, the message bus security policy blocked the reply, the reply
       timeout expired, or the network connection was broken.

Fix:

    export NO_AT_BRIDGE=1


### Emacs doesn't mark a region anymore with Ctrl-Space

_ibus_ stole that key. Remove that keybinding from ibus so it is available
again for Emacs:

    ibus-setup

(as normal user, not as root!)

Tab "General", "Next input method"; click on "...", then "Delete".

**Nobody needs that bullshit!**

What fucking moron came up with such a brain-dead idea?

Seriously: It is already hard enough to advocate a Linux desktop in the sorry
state it's in. We surely don't need anybody fucking around with it every
couple of weeks to make it even harder to use. And breaking the most basic and
single most important key combination of one of the oldest and most widely-used
Linux editors like Emacs is adding insult to injury.


### Emacs keeps marking regions in bright yellow, and it won't ever go away

This is the "secondary selection", a remnant of the very early days of the X
Window System, going back to the early nineties (i.e. the last millenium). This
is obsolete and braindead and just a PITA. It was of little use back in the
days of xterm, xload and xclock, and it is completely useless in this day and
age of KDE, GNOME, Xfce. Why after so many years somebody decided to enable
that by default is beyond me; it shows a complete disconnect from the user
base.

I have been using Emacs since 1992 or so, and I had never come across this -
until two years ago or so. Suddenly, sometimes for seemingly no reason at all,
I got blocks of text highlighted in bright yellow, and there was no way to get
it back to normal other than restarting Emacs. What a PITA. And for the life of
me I could not find out what was going on and how to fix it.

Emacs uses mouse operations in combination with the Meta key for that secondary
selection. Normally, window managers tend to eat those key combinations:
Alt-drag-mouse-1 for moving windows around, Alt-drag-mouse-3 for resizing them.

I am not sure what other keys also cause this; some genius might have found one
of those other completely superfluous keys on the keyboard (some of the Windows
keys?) to be "useful" for that, thus breaking my favourite editor.

May he rot in hell forevermore. May a thousand camels crap on his grave.

Anyway, here is how to get rid of it: Add to one of your Emacs startup files
(e.g. `~/.emacs`) those lines:

    (global-set-key [M-mouse-1]      nil)
    (global-set-key [M-drag-mouse-1] nil)
    (global-set-key [M-down-mouse-1] nil)
    (global-set-key [M-mouse-2]      nil)
    (global-set-key [M-mouse-3]      nil)

This simply undefines those completely braindead key combinations.

If you have an .elc counterpart (a byte-compiled version) of that file, don't
forget to byte-compile it (`M-x byte-compile-file`).

See also

  https://emacs.stackexchange.com/questions/8225/clear-secondary-selection-without-using-mouse


### Fix: Evince (the PDF reader) segfaults with weird "permission denied" problems

_Experienced with Xubuntu 18.04 LTS Bionic Beaver_

Uninstall apparmor.

    sudo apt-get remove --purge apparmor

Reboot to make sure it isn't still active in the running system:

    sudo reboot

It took me quite a while to figure that one out. It might just be an apparmor
profile that isn't set up 100% correctly, but seriously, **I don't give a
shit**. It was yet another denial of service by this "security enhancement"
tool. Why anybody would bother with this piece of ~~shit~~ software is beyond
me; it never did anything for me, only just against me. Good riddance.


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

![Etherape screenshot](http://etherape.sourceforge.net/images/v0.9.3.png)


Install:

    sudo apt-get install etherape

Using:

    xhost +
    sudo etherape


### Set up Automounter

Install:

    sudo apt-get install autofs

Edit auto.master:

    sudo vi /etc/auto.master

Add a line for a new map file:

    /nas  /etc/auto.nas  --timeout=180

The first field (`/nas`)is the parent directory for the mount points,
the second field is the name of the new map file.
`--timeout` is optional.

Create the new map file:

    sudo vi /etc/auto.nas

    work  -fstype=nfs,soft,intr  nas:/work
    sh    -fstype=nfs,soft,intr  nas:/sh

Each line describes one mount. The first field is the mount point below the
parent directory specified in the master file, i.e. in this example it will
become `/nas/work` and `/nas/sh`.

The optional second field are the mount options. The first mount option must
start with a `-` to identify the field as mount options.

For Nfs4, use

    -fstype=nfs4

The last field is the server and the exported mount.


#### Testing and Debugging

Test:

    ls -l /nas/work

This should already mount the NFS directory and show content.
The `mount` command should also show the new automount map:

    mount | grep autofs

    /etc/auto.nas on /nas type autofs (rw,relatime,...

Discover mounts on the server (here: `nas`) with

    showmount -e nas

To test what the automounter does, shut it down:
Ubuntu 16.04 and later with systemd:

    sudo systemctl stop   autofs.service
    sudo systemctl status autofs.service

Earlier Ubuntu versions with upstart:

    sudo /etc/init.d/autofs stop

Open a new shell and restart the automounter in the foreground (!) with verbose
and debug output:

    sudo automount -f -v -d

Watch the output while you try to access the NFS mounts in another shell:

    ls -l /nas/work

When everything works well, don't forget to restart the automounter:
Ubuntu 16.04 and later with systemd:

    sudo systemctl start  autofs.service
    sudo systemctl status autofs.service

Earlier Ubuntu versions with upstart:

    sudo /etc/init.d/autofs start

