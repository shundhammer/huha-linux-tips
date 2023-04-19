# HuHa's SUSE ALP Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Root Login

Root login via `ssh` is disabled by default. Either log into the system console
as root, or log in via `ssh` as a normal user (which you _should_ really create
during installation) and then use `su`.

Notice that you can also use a root shell in Cockpit in the "Terminal" tab.


## Console Keyboard Layout

The US keyboard layout is the default. To get another one, use `localectl`.

    localectl status
    localectl list-keymaps

For a German keyboard layout:

    localectl set-keymap de


## Use Cockpit for Administration via Web

- Find out the machine's IP address:

      ip a

  For IPv4 only:

      ip a | grep 'inet ' | grep -v '127.0.0.1'

- Start a browser tab with that IP address followed by `:9090`:

      https://192.168.178.44:9090

