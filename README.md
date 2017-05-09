# build-ubuntu-vm

This repository contains a couple of scripts to create a VirtualBox image
for a number of Ubuntu flavours. The scripts create a new install ISO
and then start a VM and boot the ISO. The ISO is set to run unattended,
and by the end of the process you have a running, fully configured Ubuntu
VM. You can then shut it down and you have your new VM.

# Pre-requisites

You will need already installed:

* VirtualBox
* These packages: whois, p7zip-full, genisoimage
* A copy of the [Minimal Ubuntu CD](https://help.ubuntu.com/community/Installation/MinimalCD)

It is also a good idea to install the *apt-cacher-ng package*. The Minimal
Ubuntu CD has very few packages; most of them are installed from the Internet.
The *apt-cacher-ng* package will act as a caching proxy, so you can build the
customised ISO several times and only have to download the packages once.

# Making Your Customisations

The `test1` directory has a file called `config` which holds the customisations
that we want to apply to the Minimal Ubuntu CD. You can make your own
directories, so that you can build multiple, different, customised ISO files.

Here are the contents of the `config` file:

```
# Config file used by the customise_iso script
username=fred
password=abc123
rootpassword=abc123
hostname=ubuntu
timezone=Australia/Brisbane
proxy=http://172.17.120.172:3142
base_system=lubuntu-desktop
orig_iso=/tmp/mini_17.04.iso
new_iso=/tmp/custom.iso
packages="rcs, build-essential"
late_command='sh /copyit'
```

The compulsary lines are *username*, *password*, *hostname*, *timezone*,
*proxy*, *base_system*, *orig_iso*, *new_iso* and *packages*.

The *username*, *password*, *hostname* and *timezone* should be obvious. The
*proxy* is the URL of your caching proxy. The *orig_iso* is the absolute
location of the Minimal Ubuntu CD on your system. The *new_iso* is the
absolute location of the customised ISO that will be created.

You need to choose a specific Ubuntu distribution to customise. This is
done with the *base_system* line. According to
[this web page](https://help.ubuntu.com/16.04/installation-guide/amd64/apbs04.html#preseed-pkgsel),
the available options are:
*    standard (standard tools)
*    ubuntu-desktop
*    kubuntu-desktop
*    edubuntu-desktop
*    lubuntu-desktop
*    ubuntu-gnome-desktop
*    xubuntu-desktop
*    ubuntu-mate-desktop
*    lamp-server
*    print-server (print server)

If you want extra packages installed, list them as a comma-separated
names on the *packages* line.

In the `test1` directory, there is a subdirectory called `cdrom`. Anything
in this directory is included at the top-level of the `initrd` on the
customised ISO image. These files will be visible in the root directory
during the installation process.

The *late_command* line contains a command which runs during the installation
process. You can use this to do extra customisations which are not already
covered. Have a look at the `test1/cdrom/copyit` and `test1/cdrom/rc.local`
scripts. These copy the VirtualBox Guest Additions to the VM's hard disk,
and then install the Guest Additions on the first boot from the hard disk.

# Running the Scripts

Assuming that you have downloaded the Minimal Ubuntu CD to
`/tmp/mini_17.04.iso`, you can now run the `customise_iso` script
to build the customised ISO at `/tmp/custom.iso`:

```
$ ./customise_iso test1
Unpacking /tmp/mini_17.04.iso
Adding test1/preseed.cfg: 8 blocks
Adding extra files from test1/cdrom: 15902 blocks
Generating /tmp/custom.iso
```

With the customised ISO now generated, you can start a VirtualBox VM and
install the system with the `vbox_boot` script:

```
$ ./vbox_boot test1
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Virtual machine 'test1' is created and registered.
UUID: 1d36dc56-06dc-4184-bfc1-2c63936c460c
Settings file: '/d/VirtualBox/test1/test1.vbox'
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: 10d89167-76ee-49b3-88c0-cf239fa95f7f
Name:            test1
Groups:          /
Guest OS:        Ubuntu (64-bit)
UUID:            1d36dc56-06dc-4184-bfc1-2c63936c460c
Config file:     /d/VirtualBox/test1/test1.vbox
Snapshot folder: /d/VirtualBox/test1/Snapshots
Log folder:      /d/VirtualBox/test1/Logs
Hardware UUID:   1d36dc56-06dc-4184-bfc1-2c63936c460c
Memory size:     512MB
< lots more details of the VM omitted >

Waiting for VM "test1" to power on...
VM "test1" has been successfully started.
```

A new VM window will appear, the ISO will boot and the Ubuntu system
will be installed.

# Things to Do

This is still a work in progress, but it's probably a good starting-off
point for you to make your own changes. Some things to add:

* Separate scripts for launching the install in VMware etc.
* More customisations to add the user, key etc. as required by Vagrant
* Another script to shut down the VM and package up a Vagrant box

Yes, I know that this could all be done with Packer. Feel free to show me
how! I just wanted to try doing it with a couple of simple shell scripts :-)
