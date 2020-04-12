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
* Natural bit-flips (bitrot) occurs in some file (where the corresponding application cannot handle this false data).
* User deletes or overwrites data that is still needed.

To not need to implement mechanisms against such data loss scenarios on multiple PCs/Servers
we collect all the data of all such clients on one machine
which provides us with the full blown feature set against data loss.


### Used technologies

* advanced filesystem: `btrfs`
  * atomic snapshots (created by 3rdp party script: `btrbk`)
  * checksum over each file (protects against bitrot)
  * RAID (do not use 5/6)
* widespread fileservice: `smb`
  * used in windows, mac and linux environments
  * `samba` implements this protocol
  * provide snapshots to user via `shadow-copy`
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

If you want to customize the setup you can customize the following files:

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

TODO(clang):

* SMB access
* shadow-copy
* setup backups with rsync/qtdsync
* overwatch monitoring files
* show some maintenance steps/tools/commands (scrub, temps, free space, disk replace/add)



Presentation
------------

This example was part of a presentation that shows the danger of personal data loss
and a simple and inexpensive way to create a save place for all private data in a private network.

TODO(clang): add presentation PDF



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
