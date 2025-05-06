# SUSE Linux Enterprise Server 16 (SLES 16)

## Root Access

https://confluence.suse.com/pages/viewpage.action?spaceKey=SLES16&title=Root+authentication

(SUSE internal)

- Physical (text console / serial / graphical): still allowed by default, no
  changed compared to SLES 15

- ssh:

  - root login is DISABLED when trying to use password authentication.
    To allow it, install the `openssh-server-config-rootlogin` package.

  - root login is still possible when an ssh key is configured in
    `/root/.ssh/authorized_keys*`.


# su / sudo

- su: No change (still allowed) and is usually not recommended

- sudo: On SLES (16 and before), sudo is configured to prompt for password from
  the targetted user. In `/etc/ sudoers`, it is named "targetpw".

New in SLES 16.0:

- The `wheel` group is now used (it was not used on SLES < 16.0, unlike other
  Linux distributions like RHEL/Fedora) to track if a user can become root with
  only its own (user) password

- The first user created by the installer is added to wheel group.

- When running `sudo -i` (or using pkexec or polkit based authentication):

  - If the user is part of the `wheel` group, he is prompted for its own
    password.
  
  - if user is NOT part of wheel group, he is prompt for the root password.

This new policy is implemented with the `sudo-policy-wheel-auth-self` package
which is installed by default on a new SLES 16.0 deployment.