mini-nas
========

Ansible playbook and vagrantfile to deploy simple NAS OS based on ubuntu with advanced btrfs features.



Introduction
------------

This is an example of a simple NAS installation targeted on private networks
where private data should be centralized
and backups from client machines stored on.

This example shows the capabilities of the used technologies
and how to setup a simple but powerfull setup.
Even though this setup is not complete and should considered *incomplete* for real-world usages.

The whole point is to save all the important data in a secure place that is resilient to
hardware, software and user failures. This covers e.g. the following use cases:

* Harddisk reaches end of live and generates write/read errors or dies instantly.
* Natural bit-flips (bitrot) occur in some file (where the corresponding application cannot handle this false data).
* User deletes or overwrites data that is still needed.

To not need to implement mechanisms against such data loss scenarios on multiple PCs/Servers
we collect all the data of all such clients on one machine
which provides us with the full blown feature set against data loss.


### Used technologies

* advanced filesystem: `btrfs`
  * atomic snapshots (created by 3rd party script: [`btrbk`](https://github.com/digint/btrbk))
  * checksum over each file (protects against bitrot)
  * RAID0 or RAID1 (do not use 5/6)
* widespread fileservice: `smb`
  * used in windows, mac and linux environments
  * `samba` implements this protocol
  * provide snapshots to users via `shadow-copy`
* collecting backups: `rsync`
  * mature and simple tool to copy data between hosts
  * synchronize a destination folder (backup) with the state of the source (client)
  * fast because of multiple algorithms/mechanisms to transfer only changed memory blocks
* monitoring: `S.M.A.R.T.`
  * hardware feature of all kind of disks
  * provides statistic data (e.g. age, power on hours, read error rate, etc.)
  * some self made scripts to get a fast overview of the current state of the filesystem/hardware



Getting started
---------------

This example project consists of mainly two parts:

* `vagrantfile`: definition of a virtual machine with all needed hardware parts to simulate a NAS
* `ansible` playbook: setup definition which installs all services/tools/configs used to provide the described features.

First you need to install `vagrant` and `virtualbox`.
The `Vagrantfile` is not compatible with other VM provides because of `virtualbox` specific configuration.

~~~~~~
# windows (using Chocolatey)
choco install -y git.install vagrant virtualbox

# linux
sudo apt install git vagrant virtualbox
~~~~~~

Then you need to clone this project somewhere on your disk:

~~~~~~
git clone https://github.com/langchr86/mini-nas
~~~~~~

And startup` vagrant` to setup the virtual machine and install all the tools using `ansible`:

~~~~~~
cd mini-nas
vagrant up --provision
~~~~~~

After that step you have a full working NAS running in a VM.



Customization
-------------

If you want to customize the setup you can change the following files:

* `Vagrantfile`
* `ansible/playbook.yml`

If you only changed the playbook you mostly only need to re-run ansible with:

~~~~~~
vagrant provision
~~~~~~

If you changed some VM settings or more intrusive ansible things you will need to destroy the VM and start from scratch:

~~~~~~
vagrant halt
vagrant destroy -f
vagrant up --provision
~~~~~~



Usage
-----


### Connect samba share

Now your virtual NAS is running and it is time to access the SMB share.
This can be done by opening an explorer and access the URL: `\\mini-nas`.
If this does not work you need to use the IP of the VM.
Evaluate it with connecting to the VM with `vagrant ssh`
and then get the IP address with `ip a`.

The share is password protected.
Use one of the defined users in the playbook (or your own defined).
All users are set by default with the exact same password as the user name.
If you change the password of the samba user later the ansible role will not change it.


### Access old snapshots of data

The samba client allows to access so called `shadow-copy` versions of your files.
These can be accessed e.g. in windows directly by using the context-menu: `Restore previous versions`.
This is possible on each file or subfolder existing in your share.
At least if there is a snapshot with a previous version of this file/folder.
Snapshots are created with the `btrbk` utility by creating read-only snapshots of a `btrfs` subvolume.
All the created snapshots are directly accessable under `/mnt/pool-main/snapshots`.
The snapshots are automatically cleaned-up, each time the tool runs.
When the tool is executed is controlled by: `timer_OnCalendar: "*-*-* *:00,10,20,30,40,50:00"`.
In this case it is run each 10 minuntes.
This means we get a new snapshot each 10 minutes.
The cleanup is controlled by:

~~~~~~
snapshot_preserve_min: "1h"
snapshot_preserve: "12h 14d 5w 3m"
~~~~~~

In this case we keep all snapshots not longer then 1 hour.
Note that cleanup will only happen at full hours.
This means intermediate snapshots may exist at most 1 hour and 59 minutes.
After that we keep 12 hourly, 14 daily, 5 weekly and 3 monthly snapshots.


### TODO(clang)

* setup backups with rsync/qtdsync
* overwatch monitoring files
* show some maintenance steps/tools/commands (scrub, temps, free space, disk replace/add)
* cockpit



Presentation
------------

This example was part of a presentation that shows the danger of personal data loss
and a simple and inexpensive way to create a save place for all private data in a private network.

TODO(clang): add presentation PDF



Real hardware
-------------

If you want build up a real NAS with all these features this should be really simple.
Install an ubuntu distribution
(e.g. [ubuntu-server](https://ubuntu.com/download/server) or [armbian](https://www.armbian.com/))
on your hardware,
install newest ansible,
clone this repository,
customize some things in the playbook (mainly the harddisc paths)
and let ansible do its magic:

~~~~~~
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible -y

git clone https://github.com/langchr86/mini-nas
cd mini-nas

nano ansible/playbook.yml
# remove line: include_tasks: tasks/virtualbox_additions.yml
# make own customization

./run-local.sh
~~~~~~



Known issues / future features
------------------------------

* playbook is not idempotent because the samba role does create its
  config file via `template` mechanism and the btrbk role does manipulate this.



References
----------

Some parts of my personal setup are published in other github projects:

* [hostcontrold](https://github.com/langchr86/hostcontrold): Small daemon that overwatches servers in a network and shuts them down to save power.



License
-------

MIT



Versioning
----------

There exists no version numbers, releases, tags or branches.
The master should be considered the current stable release.
All other existing branches are feature/development branches and are considered unstable.



Author Information
------------------

Christian Lang
[lang.chr86@gmail.com](mailto:lang.chr86@gmail.com)
