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
that VM. Click on the _Shared folders_ heading. I the pop-up dialog, add the
desired folders that you want to share and give each one a name for use inside
the VM:

```
work_src  /work/src
work_tmp  /work/tmp
```

`work_src` is the name that I chose, `/work/src` is the path on the host machine.


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
VBoxManage getextradata Tumbleweed enumerate
```

Now shut down your guest VM and **restart the VirtualBox application** (!!) for
the changes to take effect.

You should now be able to create symlinks in the guest VM in those shared folders.

