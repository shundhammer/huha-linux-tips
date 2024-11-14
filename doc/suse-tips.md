# HuHa's SUSE tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License

(considerably less than the Ubuntu counterpart because I've been using SUSE for
so long that most things in a SUSE environment come natural to me)



## Getting a freshly installed SUSE up and running

### Install bare minimum tools

    sudo zypper in mmv zsh emacs
    sudo zypper in exif exiftool exiftran jhead id3v2
    sudo zypper in qt5ct

Add the `PackMan` repo, then

    sudo zypper in vlc

If your machine has a DVD drive and you want to watch video DVDs, add the
`libdvdcss` community repo, then

    sudo zypper in libdvdcss2


### Allow 'sudo' without password for yourself:

    sudo visudo

Add those lines at the end of the file (AFTER any `include`):

    myusername  ALL=(ALL) NOPASSWD: ALL
    Defaults !log_allowed
    Defaults env_keep = "DISPLAY QT_QPA_PLATFORMTHEME LANG LC_ADDRESS LC_CTYPE LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE LC_TIME LC_ALL LANGUAGE LINGUAS XDG_SESSION_COOKIE"

Replace `myusername` with your real user name, of course.

Alternative to the `env_keep` line: Add

    Defaults    !env_reset

Explanation:

- The first line allows you to use `sudo` without entering the password all the time.

  Notice that SUSE needs you to enter the _root_ password, for Ubuntu / Debian you need
  to enter your normal user account's password.

- The `!log_allowed` line does not write every succesful `sudo` command to the log;
  which by itself is a severe privacy intrusion.

- The `env_keep` line lists those environment variables to keep while invoking
  a command with `sudo`. By the (maybe more secure, but extremely
  user-unfriendly) defaults, it simply deletes them all, so you can't even do
  basic things with X11 or Qt programs. So we added `DISPLAY` and
  `QT_QPA_PLATFORMTHEME` here. The others are copied form the original line
  somewhere in the middle of the file.

- The alternative `!env_reset` line doesn't delete the environment variables
  in the first place, but it might open up other security holes with
  environment variables that you might not even think of.


### Alternative: Disable 'sudo' logging

    sudo visudo

Add line

    Defaults !syslog, !pam_session


Alternatively:

Just for user 'kilroy':

    Defaults: kilroy !syslog, !pam_session

Just don't log successful `sudo` commands
(this will still log the `sudo` session start and and the UID):

    Defaults !log_allowed


### Disable forced NumLock

_Whoever came up with the brain-dead idea to force this to "on" should be
shot!_

    sudo zypper rm numlockx
    sudo zypper addlock numlockx


### Get a usable font size for Qt programs

    sudo zypper in qt5ct

Add to `~root/.bashrc`:

    export QT_QPA_PLATFORMTHEME=qt5ct

Source that file for using right now: As root:

    . ~root/.bashrc

Configure the Qt look and feel: As root:

    qt5ct

The result is stored in `~root/.config/qt5ct`.


### Install the Opera Browser

No need to add an extra repo; Opera is available from the Leap Non-OSS repo.

    sudo zypper in opera

But adding the extra codecs from Chromium is strongly advised.
Add the PackMan community repo, then

    sudo zypper in chromium-ffmpeg-extra


### Keeping Opera and Codecs in sync

Notice that the codecs need to match the chrome engine that this version of
Opera was built against. Sometimes it is advisable to lock both packages
because they might get updates out of sync with each other.

In Opera, use the "Help" menu -> "About Opera" to check the Chromium
version. That's the version of `chromium-ffmpeg-extra` that should definitely
work.

If the versions don't match, consider locking and unlocking both packages to
update both when matching versions are available:

    sudo zypper addlock opera chromium-ffmpeg-extra

and

    sudo zypper removelock opera chromium-ffmpeg-extra

Check with

    sudo zypper ll

Notice that in Leap, sometimes you also have to lock specific patches that
bring an Opera update; but if the codes don't also get an update, you'll get a
dysfunctional Opera: Some web videos simply won't play. That typically affects
YouTube (some videos, not all), Facebook, Twitter.

Always check with

    zypper lp

or

    zypper lp | grep opera

if a patch would update Opera.



## Switch off Annoying Emojis on Opera Tabs

Edit the launcher (the .desktop file) on your desktop (right-click, then
"Properties", switch to the "Launcher" tab) end add to the opera command line:

    --with-feature:tab-art=off

(Found somewhere in an Opera user forum, NOT on any official Opera web site)

_Making this the default is annoying enough, but not giving the user the
possibility to disable this is adding insult to injury._

_This is one more step in all the enshittification that Opera has undergone in
recent years. Simply stop adding more and more bullshit that nobody wants!
Colored smileys that just get in the way when you want something as simple as
_closing_ a tab, really? What is this, built-in Candy Crush? What are they
thinking?!_



### Change Default Browser for URL Handling

Symptom: Hyperlinks (http, https) are opened in Firefox, not in your favourite
browser. In this example: _Opera_.

- Make sure that your favourite browser is selected as the default in your desktop. In _Xfce_:

  Settings -> Default Applications -> Web Browser

- Find the .desktop file for that browser from its package file list. For Opera:

      rpm -ql opera | grep '\.desktop'

      /usr/share/applications/com.opera.opera.desktop

- Check what handler is currently used for the relevant MIME types:
  - `x-scheme-handler/https`
  - `x-scheme-handler/http`

  with:

       xdg-mime query default x-scheme-handler/https
       xdg-mime query default x-scheme-handler/http

- Set your favourite browser as the new handler:

      xdg-mime default com.opera.opera.desktop x-scheme-handler/https
      xdg-mime default com.opera.opera.desktop x-scheme-handler/http

- Check the new handler:

      xdg-mime query default x-scheme-handler/https
      xdg-mime query default x-scheme-handler/http

**Notice that just setting the MIME association `text/html` to that browser might not work.**


## Firefox

### Don't Quit Firefox when Closing the Last Tab

Go to `about:config`, search for `closeWindow` and change

    browser.tabs.closeWindowWithLastTab

to `false`.



### X11 Mouse Pointer Grabs

Use `Ctrl` + `Alt` + `NumPad/` to stop a grabbed X11 mouse pointer.

If that doesn't work, check with `xev` (`sudo zypper in xev`) if that key
combination indeed sends the _XF86Ungrab_ key symbol.


### Disable Web Pages Confirmation Popup when Closing

Go to `about:config`, search for `dom.disable_beforeunload`, change to `false`.

-> No more impertinent asking by web pages such as YouTube's comment history if
   you are sure you want to leave the page.
   
   **None of your fucking business, YouTube!**
   I am the boss on my machine, not some impertinent web programmer!



## Xfce Desktop

### Usable Font Size on Full HD Laptop

14" Laptops with a 1920x1200 or 1920x1080 resolution have 160 dpi, but setting
that is too much: Fonts become huge, and you'd have to select a 7 or 8 point
font which does not leave any room for smaller fonts on the desktop etc., and
UI elements in Gtk become too large.

Workaround: Select 128 dpi instead of the real 160 dpi.

Go to _Settings_ -> _Appearance_ -> Tab "Fonts", check "Custom DPI setting",
enter "128", then select the fonts you like.

On my laptop, I have "Sans Regular 9.5" (yes, you can enter fractional numbers
here) and "Monospace Regular 9".

Don't forget: 

-The xfce4-terminal settings: Context menu -> "Preferences" -> Tab
"Appearance"; "Dejavu Sans Mono Book 9" is a good choice.

- The desktop settings: Right-click on the desktop, "Desktop Settings", tab
  "Icons"; if you have "Use custom font size", an 8 point font is a good choice
  here.

- "Settings" -> "Window Manager" -> tab "style"; "Title font" -> "Sans Regular 9".

- Firefox fonts

- Thunderbird fonts

- Qt programs (qt5ct; see that section here)

The larger UI elements are also used by Firefox and Thunderbird.


### Xfce NumLock State

- Open _Settings_ -> _Keyboard_ -> Tab "Behavior"

- Set or unset "Restore num lock state on startup"

Or check with:

    xfconf-query -c keyboards -lv


### Get Rid of Duplicate SysTray Icons

    xfce4-panel --restart


### Open Desktop Folder Links in a New Window in PcManFM

- Open _Settings_ -> _Default Applications_ -> Tab "Utilities"

- Combo box "File Manager" -> "Other"

- Select "pcmanfm" and add option `-n` (`/usr/bin/pcmanfm -n "%s"`)


### Install Additional Xfce Packages

    sudo zypper in xfce4-places-plugin xfce4-cpugraph-plugin xfce4-mount-plugin xfce4-systemload-plugin
    sudo zypper in xfwm4-themes

Check out `xfwm4-theme-*`.

For laptops:

    sudo zypper in xfce4-battery-plugin


Personal preference: Replace `xfce4-*-branding-openSUSE` with `xfce4-*-branding-upstream`.


## Change Tiny Bootloader and Console Font

### Change Grub Font

Create a .pf2 font in the size you like from a True Type Font (ttf):

    sudo grub2-mkfont -s 24 -o /boot/grub2/deja.pf2 \
      /usr/share/fonts/truetype/DejaVuSansMono.ttf

Add that font to `/etc/grub.d/00_header`:

    sudo vi /etc/grub.d/00_header

    GRUB_FONT=/boot/grub2/deja.pf2

Disable graphical booting and restore plain text mode for Grub:

    sudo vi /etc/default/grub

and comment out the lines with those variables:

    # GRUB_TERMINAL="gfxterm"
    # GRUB_GFXMODE="auto"
    # GRUB_THEME=...

Build a complete grub.cfg from all the snippets in `/etc/grub.d`:

    sudo grub2-mkconfig -o /boot/grub2/grub.cfg


### Change Console Font

Install the `terminus-bitmap-fonts`:

    sudo zypper in terminus-bitmap-fonts

Check the available fonts in that package with

    rpm -ql terminus-bitmap-fonts | grep console

To preview the font, change to a text console (Ctrl-Alt-F1) and enter

    setfont ter-v24b

Configure an appropriate font:

    sudo vi /etc/vconsole.conf

    FONT=ter-v24b.psfu

(original: `FONT=eurlatgr.psfu`)
See also `man vconsole.conf`.

Add this font as a default kernel parameter:

    sudo vi /etc/default/grub

    GRUB_CMDLINE_LINUX_DEFAULT="... vconsole.font=ter-v24b.psfu ..."

and recreate the grub configuration:

    sudo grub2-mkconfig -o /boot/grub2/grub.cfg

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


### Ignore a Disk / Block Device

    echo 1 >/sys/block/sdX/device/delete

See also https://bugzilla.suse.com/show_bug.cgi?id=1195894#c22



----

## ssh

### Can't ssh as root Anymore

This affects mostly openSUSE Tumbleweed since mid-2021.

- Log in to that remote machine as a normal user

- Edit `/etc/ssh/sshd_config` (notice the `d`! not `/etc/ssh/ssh_config`,
  `/etc/ssh/sshd_config`!) with root privileges:


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

**I am the boss on my machine, and I get root access to VMs on it whenever I
please. Do you read me, security folks?** The same applies to all my machines
in my home network.

It's nobody's business to make life even harder for users even in their home
networks where nobody can log in from the outside to begin with. All you do is
encourage people to migrate to BSD or to MacOS X or back to Windows.


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

    sudo vi /etc/zypp/zypp.conf

Add

    solver.allowVendorChange = true


### Disable Delta RPMs

If your Internet connection is fast, but delta RPMs take a long time to apply,
simply tell zypper to download the complete new RPM instead of just a delta and
apply the delta to the old RPM:

    sudo vi /etc/zypp/zypp.conf

Locate this line, uncomment it and set the value to _false_:

    download.use_deltarpm = false


### Manually Updating

#### Leap

    sudo zypper refresh
    sudo zypper lp
    sudo zypper patch

(`lp` is equivalent to `list-patches`)


#### Tumbleweed

    sudo zypper refresh
    sudo zypper dup -dRy
    sudo zypper dup -dRy

(`-dRy` is equivalent to `--download-only --no-force-resolution --no-confirm`)

until nothing is downloaded anymore; then:

    sudo zypper dup

Downloading all pending updates before actually applying them minimizes the
risk of getting into an inconsistent state where some packages were updated,
some were not, and some may already be available again in an even newer
version.


### VPN

    zypper in NetworkManager-openvpn




### Disable Automatic Daily Updates Check

Configuring that only in the Xfce/GNOME systray applet has no effect:

That applet will STILL alert you at the most inconvenient times. And it will
never realize when you already did today's updates manually (still screaming
its ALARM!! at you); and when you use the GNOME PackageKit app, that thing will
always be in a totally antisocial fullscreen mode (which cannot be configured
away), so you have to resize it manually to a reasonable size to make your
computer usable while it is running.

It's much better to update manually (see previous section) when it's convenient
for you, not at some random time when some systemd timer runs out, and then
that alarm icon keeps nagging you in the systray.

So, disable that timer-based auto-update with:

    sudo systemctl disable packagekit-background.timer

Check with:

    sudo systemctl status packagekit-background.timer

Check all related systemd units:

    sudo systemctl list-unit-files "packagekit*"

The units are defined in `/usr/lib/systemd/system/packagekit*`
or (if there are user-defined ones) in `/etc/systemd/`.


### Zypper Documentation

How-To:

  https://en.opensuse.org/SDB:Zypper_usage

SLES Manual:

  https://documentation.suse.com/sles/15-SP1/html/SLES-all/cha-sw-cl.html#sec-zypper
  https://documentation.suse.com/sles/15-SP1/html/SLES-all/cha-upgrade-online.html#sec-upgrade-online-zypper-plain



### Expand an RPM Macro

The macros used in .spec files are defined in `/usr/lib/rpm` and its
subdirectories.

To see a specific one expanded, e.g. `_sbindir`:

    rpm -E '%{_sbindir}'

or

    rpm --eval '%{_sbindir}'


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
