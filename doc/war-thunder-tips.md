# HuHa's War Thunder on Linux Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Update 2020-11-19:

As of today with version 2.1.0.36, War Thunder is now back to working again for
me.

Thanks to everybody involved; and _many_ thanks to the continued Linux support
by the Gajin developers!


## Problem (now fixed)

With the November 17, 2020 update _New Power_, War Thunder didn't run anymore
on (some?) Linux systems: After hitting the "Play" button from the game
launcher, all you got was a black screen.


## Workaround (no longer necessary)

### Overview

- Shut down your graphical desktop for now

- Go to the War Thunder game directory

- Start a fresh X server with the War Thunder binary ("aces") directly

- Play


### Preparations

Save your War Thunder settings so you can restore them later:
Open a shell window and type

    cd ~/.config
    tar cjvf ~/WarThunder-config.tar.bz2 --exclude .game_logs WarThunder

You can later restore that with:

    cd ~/.config
    tar xjvf ~/WarThunder-config.tar.bz2


### Details

- Shut down your graphical desktop for now:

  - Shut down all your running applications to avoid losing data;
    we will kill them in a moment.

  - Use Ctrl-Alt-F2 to switch to the text console.
  
  - Log in there (with your normal user account).

  - Shut down your desktop session (including the graphical login) that is
    still running on console 7:

        sudo systemctl isolate multi-user.target


- Go to the directory that you installed War Thunder to; in my case:

      cd /work/games/WarThunder

- Go to the linux64/ subdirectory there:

      cd linux64

- Start a new X server with the binary War Thunder as the only client:

      startx ./aces
  
- **Play**



### After Playing

Just reboot:

    sudo reboot


## My System

- Xubuntu 18.04.5 LTS
- Xfce on X11
- NVidia Drivers 455.38
- NVidia GeForce GTX 1050 Ti
- Intel Core i7 870 @ 2.93 GHz
- 16 GB RAM
- Samsung 256 GB SSD, 2 * 1 TB HD
