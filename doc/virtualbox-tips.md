# HuHa's VirtualBox Tips

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Sharing Folders between Host and Guest System

### Scenario:

I wanted to edit my checked-out source code on my host Linux and
build the code in a Linux VM. I wanted to make two directory trees available
from the host system to the guest VM just as if they were local directories
inside the VM with the same permissions and everything.

I used the same user name and ID both in the host and in the guest VM (this
makes things a bit easier).


### Preparations

In the VirtualBox application, select the VM and open the settings details for
that VM. Click on the _Shared folders_ heading. In the pop-up dialog, add the
desired folders that you want to share and give each one a name for use inside
the VM:

```
work_src  /work/src
work_tmp  /work/tmp
```

`work_src` is the name that I chose, `/work/src` is the path on the host
machine.


### The Naive Approach

For each shared folder, also check the _Auto-mount_ option.

Once you start the VM, the folders are instantly available.


### Problem

The mount options are bizarre and completely useless. You basically don't have
any reasonable access inside the guest VM; you can only use them as _root_ (and
then only with limitations).

You _can_ add your normal user account inside the guest VM to the `vboxsf` user
group in `/etc/group`:

```
vboxsf:x:463:myusername
```

But still, all shared files are owned by the _root_ user and the _vboxsf_
group, and all files have execute permissions.


### The Correct Approach

Make sure to leave the _Auto-mount_ option unchecked in the settings for the
shared folders in the VirtualBox application.  Instead, Mount the shared
folders manually with the correct options.

**The mount.vboxsf command uses those bizarre mount options by default!**

Yes, it mounts everything with mount options `uid=0,gid=0`. It will only stop
doing that when you explicitly specify different values for those options.


### Manually Mounting the Shared Folders

Inside the guest VM:

```
sudo mount -t vboxsf -o uid=1000,gid=100 work_src /work/src
sudo mount -t vboxsf -o uid=1000,gid=100 work_tmp /work/tmp
```

This assumes that your user ID is 1000 and your group ID is 100 which is the
default for modern Linux systems for the first user account that is created.
For different IDs, use the correct numeric values accordingly.


### Adding the Shared Folders to /etc/fstab

Inside the guest VM:

```
sudo vi /etc/fstab
```

Add those lines:

```
# VirtualBox shared folders

work_src  /work/src  vboxsf  uid=1000,gid=100  0  0
work_tmp  /work/tmp  vboxsf  uid=1000,gid=100  0  0
```

Apply the changes:

```
sudo mount -a
```

Check if it worked:

```
mount | grep vbox
```

or

```
df -x tmpfs -x devtmpfs
```

You should see your shared folders in both cases.



## Enable Symlinks for Shared Folders

They disabled symbolic links for shared folders for security reasons. But that
also makes shared folders pretty much unusable for most real uses, such as
software development. Here is how to enable them:

On the host system (!), for each shared folder execute this command:

```
VBoxManage setextradata "$MY_VM_NAME" VBoxInternal2/SharedFoldersEnableSymlinksCreate/$SHARED_FOLDER_NAME 1
```

I.e. in the above example for shared folders _work_src_ and _work_tmp_:

```
VBoxManage setextradata "MyVmName" VBoxInternal2/SharedFoldersEnableSymlinksCreate/work_src 1
VBoxManage setextradata "MyVmName" VBoxInternal2/SharedFoldersEnableSymlinksCreate/work_tmp 1
```

_Notice that this will happily accept any random bullshit without any error._


Check:

```
VBoxManage getextradata "MyVmName" enumerate
```

Now shut down your guest VM and **restart the VirtualBox application** (!!) for
the changes to take effect.

You should now be able to create symlinks in the guest VM in those shared
folders.



## No Graphics in Guest after VirtualBox Downgrade

_This happened to me after going back to Xubuntu 18.04 LTS from 20.04 LTS
because of constant NVidia / X11 crashes._

VirtualBox 6.x introduced a new setting for graphics card type and defaults to
another one: `VMSVGA` instead of the old `VBoxVGA`. There is no way to set that
in the VirtualBox GUI in older versions, so you have to manually edit the XML
file for each affected virtual machine.

See also

https://superuser.com/questions/1403123/what-are-differences-between-vboxvga-vmsvga-and-vboxsvga-in-virtualbox


Go to your VirtualBox vms directory where the `*.vbox` XML files are and edit
them.

- **Make sure to close the VirtualBox GUI**, or it will overwrite your changes!

- Look for lines like this:

  ```XML
    <Display controller="VMSVGA" VRAMSize="16" accelerate3D="true"/>
  ```

  and change them to look like this instead:

  ```XML
    <Display controller="VBoxVGA" VRAMSize="16" accelerate3D="true"/>
  ```

  **There is one such line for every snapshot that the VM has, so make sure to
  edit all of them or at least the one for the most recent snapshot!**

- Or use this perl command to bulk-edit them all at once:

      cd /work/virtualbox/vms      # or wherever your VMs are

      perl -p -i'*.bak' -e 's/VMSVGA/VBoxVGA/g' */*.vbox

  This leaves behind a `.bak` file for every changed file, so you can safely go
  back.

Starting such a VM or editing its settings may remove that `<Display
controller...>` line entirely, leaving the defaults (which is `VBoxVGA`).


## Newer VirtualBox (6.x) on older Ubuntu (18.04 LTS)

From some point on, the older VirtualBox 5.2 that comes with Ubuntu 18.04 LTS
is no longer a viable option: A VM like Tumbleweed or Leap 15.3 boots fine, but
when it starts X11, all you get is a black screen:

The VirtualBox 6.x guest additions in the VM are no longer compatible with the
older VirtualBox 5.2 on the host. While this is clearly a bug, there is no fix
in sight.

But you can install the official VirtualBox 6.x from
https://www.virtualbox.org/ .



### Add the VirtualBox APT Repo

See also https://tecadmin.net/install-virtualbox-on-ubuntu-18-04/

- Uninstall the old VirtualBox 5.2 packages that came with Ubuntu 18.04:

      sudo apt remove

- Import the repo's keys:

      wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
      wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -

- Add the repo. You _could_ simply do this with `sudo add-apt-repository ...`
  as described in that (very good and very helpful) tecadmin.net article, but
  then it is added to all the other repos in `/etc/apt/sources.list`. For
  clarity and better organization, it is recommended to put this into a separate file:

      cd /etc/apt/sources.list.d
      sudo vi virtualbox-org.list

  content of that file:

      deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian bionic contrib
      # deb-src http://download.virtualbox.org/virtualbox/debian bionic contrib

- Whichever way you choose, make sure to add that `[arch=amd64]` part, or you
  will very likely get a warning

  _Skipping acquire of configured file 'main/binary-i386/Packages' as
  repository 'xxx' doesn't support architecture 'i386'_

  every time you do `sudo apt update` from now on which is very annoying, so
  tell _apt_ that this repo only supports the _amd64_ architecture; that's what
  that `[arch=amd64]` is for .

  See the _askubuntu.com_ article below for more background.


- Update the repo information, including the one we just added:

      sudo apt update

- Check what newer versions are now available:

      sudo apt-search "virtualbox-6"

  Or, more generally, but with more irrelevant output:

      sudo apt-search virtualbox

- Install the newer version. **DO NOT** install just any random version; be
  more specific by adding the version number:

      sudo apt install virtualbox-6.1

  Don't be surprised if that takes a while: It will have to compile a kernel
  module.



### Reference

- https://www.virtualbox.org/
- https://tecadmin.net/install-virtualbox-on-ubuntu-18-04/
- https://askubuntu.com/questions/741410/skipping-acquire-of-configured-file-main-binary-i386-packages-as-repository-x
