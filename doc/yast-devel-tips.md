# HuHa's Tips for YaST Development

Author: Stefan.Hundhammer@gmx.de

License: GNU Free Documentation License


## Use YaST2 Classes in irb (Interactive Ruby):


    irb
    require "yast"
    Yast.import "UI"

Finding out YaST UI built-ins:

    Yast::UI.methods


## GitHub

### Suppress Whitespace Diffs in GitHub

Add `?w=1` to URL:

    https://github.com/blog/967-github-secrets?w=1


## Debugging YaST Installation

- Start VM:

      xhost +
      virt-manager

- Create new VM
- Boot VM - use kernel parameter

      ssh=1

- Qt installation: In xterm:

      ssh -X root@192.168...
      yast.ssh

- NCurses installation: In xterm:

      ssh root@192.168...
      yast.ssh

- Send to background:
      ^Z
      bg

- Bind-mount files to edit:

      cd /tmp
      mkdir work
      cp /usr/share/YaST2/modules/StorageProposal.rb .
      mount -o bind StorageProposal.rb /usr/share/YaST2/modules/StorageProposal.rb

- Preferred way: Edit on host PC and copy file into VM:

      scp StorageProposal.rb root@192.168..../tmp/work/

- Alternative: After completion: Copy changed files out of VM: (don't forget!)

      scp /tmp/work/StorageProposal.rb 192.168.122.1:/tmp


## Updated files in /y2update

Prepare /y2update directory on the target machine (e999 here):

    ssh root@e999
    mkdir /y2update

On the development machine, the directory structure in each project below src/
should match the directory structure in /usr/share/YaST2, so you can simply
recursively copy that entire tree to the target machine:

    cd yast-myproject
    scp -r src/* root@e999:/y2update


## DUDs and ISOs

### Creating a Driver Update CD (DUD)

Once:

sudo zypper in mkdud

Later:

    mkdud -c /tmp/sample.dud  --obs-keys --dist 13.2 yast2-storage-*.rpm

    mkdud -c ../iso/nocow.dud --obs-keys --dist sles12 *.rpm

    mkdud -c ../iso/nocow.dud --obs-keys --dist tw *.rpm



### Creating a DUD with Single Files (instead of RPMs)

    mkdir -p files/usr/share/YaST2/modules
    cp /usr/share/YaST2/modules/StorageProposal.rb files/usr/share/YaST2/modules

    mkdud -c dasd-skip-reuse.dud --name "dasd-skip-reuse" --dist sles12 files/


### Add OBS Keys to RPMs in DUD

    mkdud --obs-keys ...


### List the Contents of a DUD

    mkdud --show mydud.dud

or

    zcat mydud.dud | cpio -t
or
    zcat mydud.dud | cpio -tv



### Integrating a DUD into an ISO

    sudo zypper in mksusecd

    sudo mksusecd --create openSUSE-Tumbleweed-HuHa-x86_64.iso \
         --initrd yast2-storage.dud \
	 openSUSE-Tumbleweed-DVD-x86_64-Snapshot20150525-Media.iso


### Integrating RPMs directly into an ISO

    sudo mksusecd --create openSUSE-Tumbleweed-HuHa-x86_64.iso \
         --initrd yast2-storage.rpm \
         --initrd yast2-installation.rpm \
	 openSUSE-Tumbleweed-DVD-x86_64-Snapshot20150525-Media.iso

**Caveat:** _This works only for the inst-sys; those RPMs will not be installed
to the target._


### Using an Updated linuxrc Package

Do **not** put the linuxrc.rpm into a DUD; specify it explicitly at the
`mksusecd` command line:

    sudo mksusecd --micro --create SLE-12-SP5-ZZZ-HuHa.iso \
        --initrd ../rpms/linuxrc-5.1.14.2-4.1.x86_64.rpm \
        SLE-12-SP5-Server-DVD-x86_64-Build0229-Media1.iso


### Creating a minimal ISO

    mksusecd --micro ...


### Use DUD via http:// from ~/Export

    cp mystuff.rpm ~/Export

At boot prompt: use

    http://w3.suse.de/~myself/mystuff.rpm


## YaST Package Handling


### Build Package

#### Ruby Packages

    rake tarball

#### Pure C++ / Autotools Packages

    make package-local


#### (Ruby Packages) Skip License Check

Edit project toplevel Rakefile:

    Yast::Tasks.configuration do |conf|
      ...
      conf.skip_license_check << /.*/
    end


## Working with OBS and IBS


### Setting up IBS (Internal Build Service)

Edit `~/.oscrc:`

Add alias for OBS:

      [https://api.opensuse.org]
    + aliases = obs
      user = shundhammer
      pass = ...


Add new section for IBS:

    [https://api.suse.de]
    aliases = ibs
    user = shundhammer
    pass = ...



Add shell alias in .zshrc (or .bashrc):

    alias isc='osc -A ibs'

source .zshrc / .bashrc:

    . ~/.zshrc



## Submit a YaST or libyui package to another target

    cd src/yast/myproj
    YAST_SUBMIT="sle_latest"  rake osc:sr

For available targets, see
https://github.com/libyui/libyui-rake/blob/master/data/targets.yml

Make sure that this target is included in `~/.oscrc`:

    trusted_prj = ... SUSE:SLE-15-SP2:GA ...



## Submit libstorage or snapper

Make sure to edit VERSION and write a change log entry:

    vi VERSION
    cd package
    bash
    . /work/src/bin/.profile
    vc

Create tarball with the new version:

    cd ..
    make -f Makefile.cvs
    make package


Check in to OBS Devel:YaST:Head

Check out from OBS
Check out from mirrored IBS (!) YaST:Head
Submit to  SUSE:SLE-12-SP1:GA

For SLE-12-SP1:

    isc submitrequest Devel:YaST:Head libstorage SUSE:SLE-12-SP1:GA
or
    isc sr Devel:YaST:Head libstorage SUSE:SLE-12-SP1:GA


    isc request show 4711


## Prevent Status _blocked_ in OBS

If a package cannot be built because of dependent packages that are in the
process of building, i.e. OBS reports status _blocked_:

Edit the meta data for that project to change to a less restrictive blocking
mode.

Web interface: "Advanced" -> "Meta"

`<repository name="REPO_NAME" `**`block="local">`**

See also https://openbuildservice.org/help/manuals/obs-reference-guide/cha.obs.build_scheduling_and_dispatching.html#id18770


## OBS to IBS Links

libstorage in IBS is a link to libstorage in OBS.

    cd ibs
    isc co Devel:YaST:Head/libstorage
    ls -1

      libstorage-2.25.35.tar.bz2
      libstorage.changes
      libstorage.spec

    isc up -u
    ls -1

      _link

    cat _link

      <link project="openSUSE.org:YaST:Head" package="libstorage">
      <patches>
        <!-- <branch /> for a full copy, default case  -->
        <!-- <apply name="patch" /> apply a patch on the source directory  -->
        <!-- <topadd>%define build_with_feature_x 1</topadd> add a line on the top (spec file only) -->
        <!-- <add name="file.patch" /> add a patch to be applied after %setup (spec file only) -->
        <!-- <delete name="filename" /> delete a file -->
      </patches>
      </link>

    isc up -e
    ls -1

      libstorage-2.25.35.tar.bz2
      libstorage.changes
      libstorage.spec



--------------------------------------------------

## Misc


### Package Version Bump

Once:

    sudo ln -s /mounts/work /work

Later:

    bash
    . /work/src/bin/.profile
    vc


### VNC viewer during Scrum videoconf:

    vncviewer -shared dhcp114.suse.cz:1
    PW: viewonly



--------------------------------------------------

## Unsorted


### Commit libyui-qt-pkg etc.


    cd ...ibs...
    (copy tarball)
    isc ar
    isc ci -n

    isc submitrequest Devel:YaST:Head libyui-qt-pkg SUSE:SLE-12-SP1:GA


    cd ...obs...
    ...obs/YaST:Head/libyui-qt-pkg 96 % rm *.tar.bz2
    ...obs/YaST:Head/libyui-qt-pkg 98 % rsync -av ~/src/yast/libyui-qt-pkg/build/package/ .

    osc ar
    osc ci -n

    osc submitrequest devel:libraries:libyui libyui-qt-pkg


### Download built RPMs from IBS

    cd /space/shundhammer/rpms

    rm libstorage*.rpm
    isc getbinaries -d . home:shundhammer:storage libstorage SLE_12_SP1 x86_64

For maintenance branches:

    cd $HOME/ibs
    cd home:shundhammer:branches:OBS_Maintained:yast2-storage/yast2-storage.SUSE_SLE-11-SP1_Update

    isc getbinaries -d ~/space/rpms SUSE_SLE-11-SP1_Update x86_64


### Show my pending OBS submit requests

    osc request list --mine

    osc request list --mine -s all -D 14


### Show pending OBS submit requests for a specific package

    osc request list openSUSE:Factory libstorage



### Build CMake Project with /usr/bin prefix

    cmake -DCMAKE_INSTALL_PREFIX=/usr/bin


### CMake-based YaST projects

    rake version:bump

    rake tarball

    rake osc:sr

#### All YaST repos:

Make sure to install `ruby2.6-rubygem-yast-rake`

(replace 2.6 with the current Ruby version)


#### libyui-qt* repos:

Make sure to install `ruby-2.6-rubygem-libyui-rake`


### Check pkg-bindings

    irb

    require "yast"
    Yast.import "Pkg"


### Fix Complaints about Missing libboost_* in inst-sys

- Use ssh installation

- Before starting yast.ssh, enable snapper extension:

    enable snapper

- Start installation:

    yast.ssh


### Packages Used in inst-sys

    cat /.packages.root
    cat /.packages.*

    ll /update/000
    ll /update/???



### Debugging "extend" in the inst-sys

    [11:23:06] <ancorgs> snwint: I'm debugging this in a s390x orthos machine
               and "extend cracklib-dict-full.rpm" is returning "extend ok" but
    	   apparently doing nothing
    [11:23:17] <ancorgs> snwint: how can I debug?
    [11:23:56] <ancorgs> in x86 looks like the whole dictionary is actually
               in inst-sys from the very beginning, so loading the extension is
    	   kinda pointless (not sure if it works or not)
    [11:23:57] <snwint> there's a log, I think in /var/log/extend
    [11:24:40] <snwint> you can also use 'extend -v'
    [11:25:12] <snwint>  /etc/instsys.parts (or so) contains the list of extensions
    [11:25:57] <snwint> if boot option debug=ignore helps, it's probably some
               signature checking going wrong


### Check Ruby Syntax

    rake check:syntax

See also .travis.yml in project toplevel dir


### Commit one patch at a time in Git

    git add -p



### Submit to older release (SLES-12-GA etc.)

- check out branch for that release with git

- edit / create pull request / merge

- submit changes:

    rake osc:sr


### Testing Multipath

    qemu-system-x86_64 -enable-kvm -display sdl -device cirrus-vga -m 1024 -boot d -drive
    file=/home/test-1.qcow2,media=disk,format=raw,serial='test70' -drive
    file=/home/test-1.qcow2,media=disk,format=raw,serial='test70' -drive
    file=/usr/local/src/ISOs/SLES-11-SP4-DVD-x86_64-GM-DVD1.iso,media=cdrom,format=raw -redir tcp:10022::22



## Workflow for Testing libstorage in inst-sys

- Create tarball:

      cd /space/shundhammer/src/yast/libstorage

      make package-local


- Check tarball into IBS:

      cd /space/shundhammer/ibs/home:shundhammer:storage/libstorage

      rsync  -av --exclude .gitignore /space/shundhammer/src/yast/libstorage/package/ .
      isc ar
      isc ci -n
      isc rebuild


- Wait for build in IBS to finish


- Fetch RPMs from IBS:

      cd /space/shundhammer/rpms

      isc getbinaries -d . home:shundhammer:storage libstorage SLE_12_SP1 x86_64

- Create DUD:

      mkdud  -c ../iso/verbose-libstorage.dud --obs-keys --dist sles12 *.rpm


- Integrate DUD into ISO:

      cd /space/iso

      sudo mksusecd --create SLE-12-SP1-ZZZ-HuHa.iso --initrd \
           verbose-libstorage.dud \
           SLE-12-SP1-Server-DVD-x86_64-Beta2-DVD1.iso


- Start installation VM:

      sudo virt-install-sles

- Start installation in ssh session:

      ssh -X root@192.168.100.230
      yast.ssh


- Watch log:

      ssh -X root@192.168.100.230
      cd /var/log/YaST2
      tail -F y2log



### Testing Multiversion Packages

      sudo vi /etc/zypp/zypp.conf

search for "multiversion" and change that line to:

      multiversion = provides:multiversion(kernel), provides:multiversion(skelcd)


### UEFI in QEMU

Install packages:

- ovmf
- qemu-ovmf-x86_64

For virt-install:

    OVMF_DIR=/usr/share/qemu

    virt-install \
      --connect=qemu:///system \
      --location=$ISO \
      --name=$VM_NAME \
      --disk=$STORAGE \
      --memory=$VM_MEM \
      --boot loader=$OVMF_DIR/ovmf-x86_64-opensuse-code.bin,loader_ro=yes,loader_type=pflash,nvram_template=$OVMF_DIR/ovmf-x86_64-opensuse-vars.bin \
      $EXTRA_ARGS


### Restart OBS checkout service

    osc service rr


### Run YaST Ruby script from the command line

    ruby -ryast -e 'puts Yast::Builtins.xy()'



### Build for Debian

    osc build mypackage.dsc

Working example:

Nasty, using debian* files:

    https://build.opensuse.org/package/show/YaST:Head:Travis/yast2

Nice and shiny:

    https://build.opensuse.org/package/show/YaST:Head:Travis/ruby-yast-rake



### Resurrect old LVM to get y2logs
===============================

    vgchange -ay


### Rake

Show targets:

  rake -T

Show where a target is defined:

    rake -w <target>

Verbose ("trace"):

    rake -t <target>


### Jenkins

- Wiki:     https://wiki.microfocus.net/index.php/YaST/jenkins
- Server:   https://ci.suse.de/view/YaST/
- Snapper:  https://ci.suse.de/job/snapper-master/

Jenkins is polling the GitHub repo with

    git -c core.askpass=true ls-remote -h https://github.com/openSUSE/snapper.git

then checks out the latest revision, then does

    rake osc:sr
    make -f Makefile.ci package
    rpmbuild -ba /home/abuild/rpmbuild/SOURCES/snapper.spec

    osc -A https://api.suse.de/ commit -m Updated to git ref a0d96b0
    osc -A 'https://api.suse.de/' --traceback --verbose checkout 'Devel:YaST:Head' snapper
    osc -A 'https://api.suse.de/' cat 'SUSE:SLE-12-SP2:GA' 'snapper' 'snapper.spec' > /tmp/yast-rake20160419-11797-1mbe4bf
    yes | osc -A 'https://api.suse.de/' submitreq --no-cleanup 'Devel:YaST:Head' 'snapper' 'SUSE:SLE-12-SP2:GA' -m 'submit new version 0.3.2' --yes


Configure Jenkins what to do:

    https://ci.suse.de/

- Log in (!)

- Search project

- Click "configure" on the left,
  scroll all the way down to the "Command" multi line edit


Fetch config:

    https://ci.suse.de/view/YaST/job/yast-snapper-master/config.xml


### RSpec

Verbose and colorized output:

    rspec -cf d  *.rb

Always add color:

    vi ~/.rspec
add
    --color


### Disable RSpec tests

Prefix "it" or "describe" with "x"

Single test:

Change
    it "does something"
to
    xit "does something"

All tests in a test file:


Change
    describe SomeStuff::Someclass do
to
    xdescribe SomeStuff::Someclass do



### Find out maintained versions

    osc maintained libstorage
    -> openSUSE ...

    isc maintained libstorage
    -> SLE-12...

    isc maintained yast2-storage
    -> SLE-11...

(libstorage was part of yast2-storage before SLE-12)



### Security updates

    isc mbranch --checkout libstorage
    isc mbranch --checkout yast2-storage

For each subdirectory there:

    isc sr -m "CVE-2016-5746 bsc#986971"



## QEMU PPC64

    qemu-system-ppc64   \
        -cpu POWER8     \
        -display sdl    \
        -m 1024         \
        -boot d         \
        -drive file=/wrk1/images/test432,media=disk,format=raw  \
        -drive file=/wrk1/iso/SLE-12-....iso,media=cdrom,format=raw,readonly


## Merge feature branch back to master

- Start from master and create a new branch for merging:

    git co master
    git branch my-merge-branch
    git co my-merge-branch

- Merge feature branch to master:

    git merge my-feature-branch

- Fix up version number in .spec file

- Fix up version number(s) in .changes file

- Revert commits from feature branch that shouldn't be in master
  (e.g. Jenkins submit target in Rakefile)

- Push to own fork on GitHub

- Create pull request against master

- Have pull request reviewed

- Merge pull request


## Debug Travis-only Failures

See also http://yastgithubio.readthedocs.io/en/latest/travis-integration/#running-the-build-locally

### Preparation

- If not installed, install Docker:

      sudo zypper in docker

- Make sure the Docker service is running:

      sudo systemctl status docker.service
      sudo systemctl start  docker.service

- Trigger rebuilding the Docker base image from the internal Jenkins instance:

  https://ci.opensuse.org/view/Yast/job/docker-trigger-yastdevel-ruby-latest/

- Get the Docker base image. For SLE-12 SP3:

      sudo docker pull yastdevel/ruby:sle12-sp3

  See also
  - https://hub.docker.com/u/yastdevel/
  - https://hub.docker.com/r/yastdevel/ruby/builds/


### Usage

- Build the Docker image for the current project:

      cd src/yast/$myproject
      sudo docker build -t yast-nfs-client-image .

  (see also the `Dockerfile` in that directory what actually happens)

- Execute locally what Travis would do:

      sudo docker run -it --rm yast-nfs-client-image yast-travis-ruby

  (see also `.travis.yml` in that directory)

- If it fails, just start a shell in Docker and execute the commands manually:

      sudo docker run -it yast-nfs-client-image bash
      yast-travis-ruby

- Install `vi` in the Docker container if needed:

      zypper in vim


More information about the YaST Docker images:

https://github.com/yast/docker-yast-ruby


## Debugging yast-registration

Fake a SLES product:

    su
    FAKE_BASE_PRODUCT=1 yast2 scc

_Just using "sudo" doesn't work because it will remove the variable from the
environment immediately!_

***This will add SLES repos to the system;** you will have to remove them
afterwards.

Check with

    sudo zypper lr | grep SLE


