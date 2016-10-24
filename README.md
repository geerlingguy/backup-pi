# Backup Raspberry Pi Server

Ansible configuration to set up a Raspberry Pi as a backup server for storing backups of data from other servers (e.g. webroots, files, and databases).

## Backstory

For many years, I've been using one or more cloud VPSes configured as a semi-secure backup server—I configure my other web and app servers to backup webroots, databases, and other files to the VPS(es) and everything is wonderful.

But as Raspberry Pis have gotten faster and cheaper, and my home Internet connection has gotten faster, I've realized I can cut the costs of hosting these backups in the cloud, and bring them home. For better peace-of-mind, I can still dump periodic snapshots (or live copies) of backup data in the Cloud (e.g. on Amazon Glacier or S3 for something that would need faster or more frequent access).

Since I had a spare 'spinning rust' 2 TB USB hard drive and Raspberry Pi 3 at my house, I configured them as a local backup server using Ansible, and now have many of my personal servers dumping backup data to this Pi-based server.

## Choosing a Raspberry Pi

I used a Raspberry Pi 3, but any model will do, as backup servers don't usually need to be fast—they just need a lot of storage and as good of I/O as possible. USB 2.0 is okay, and the 100 Mbps built-in LAN is adequate for most Internet connections. Hopefully someday the Pi will have USB 3.0 and 1 Gbps networking, but for now, we live with what we have :)

This configuration has been tested with a Raspberry Pi 2, 3, Zero, A+ and B+ (with slight modifications sometimes for A+ or Zero usage, due to port layout and lack of onboard LAN).

## Prepare the Pi

Just like any Raspberry Pi project, you first need to get an OS running and configure the basics on the Pi itself:

  1. Download [Raspbian Jessie Lite](https://www.raspberrypi.org/downloads/raspbian/).
  2. Write the img to your microSD card (`pv yyyy-mm-dd-raspbian-jessie.img | sudo dd of=/dev/rdisk2 bs=1m`).
  3. Put the microSD card in your Pi, boot it up, log in with `pi` / `raspberry`, then configure the basic settings with `sudo raspi-config`.
  4. After configuring things, reboot the Pi, and take note of it's IP address (e.g. `sudo ip addr show`). You'll use this later to connect to the Pi and configure it from another computer.

> I generally give my Pis static IP addresses, configured via their MAC addresses in my network router's settings. This way I know my Pi always gets the same IP.

## Prepare an external USB hard drive

You could technically back up to the microSD card you boot from, but for operations like backups, which write a lot of data to the drive over and over, it's best to use an external drive (SSD or HDD) instead for reliability and speed (especially for random access—[microSD cards are often very slow](http://www.pidramble.com/wiki/benchmarks/microsd-cards)!).

> If you're powering a small USB hard drive using USB only (e.g. it's not plugged into a separate AC adapter), make sure you have a very good 2A power supply driving your Pi. Otherwise the Pi might not get enough power and it will write corrupt data during backup operations!

To use a USB hard drive, you need to plug it into the Pi's USB port, then do the following to partition, format, and mount it:

### Partition the drive with `fdisk`

**Note**: These instructions presume you're using a single external USB hard drive (which Linux should see as `/dev/sda`). If you have multiple drives, you need to partition, format, and mount the correct one, otherwise you could inadvertently wipe out the contents of the wrong drive. _You've been warned!_

  1. Run `sudo fdisk /dev/sda` to open fdisk.
  2. Enter `p` to list partitions (and verify you're operating on the correct disk!).
  3. Use `d` to delete any existing partitions (e.g. `1`, `2`, etc.).
  4. Use `n` to create a new partition, using all the defaults by pressing 'Enter' (if you want to use the whole disk and have one partition).
  5. Verify the partition settings with `p` again.
  6. Once satisfied, press `w` to write the partition table.

### Create a filesystem with `mkfs`

  1. Run `sudo mkfs.ext4 -E lazy_itable_init=0,lazy_journal_init=0 /dev/sda1`, and wait for this operation to complete.

> Note that you can initialize the drive more quickly without the `-E` options. If you do so, the disk will be initialized, but for a long time you might notice activity on the hard drive. This is due to `ext4lazyinit` finishing the initialization process in the background.

### Create a mount point and mount the disk

  1. Create a directory where you'd like to mount the disk (e.g. `sudo mkdir /backup`).
  2. Mount the disk: `sudo mount /dev/sda1 /backup`.
  3. Use `df` to verify the disk is mounted.
  4. If everything's working correctly, make sure the mount is persistent by adding the following line to `/etc/fstab`:
        
        /dev/sda1       /backup         ext4    defaults,noatime  0       3

Before the `pi` user can use the mounted backup volume, though, you'll need to change ownership (otherwise you'll get a lot of 'permission denied' warnings unless you log in as `root`!). Run the command `sudo chown -R pi:pi /backup`.

## Configure the Pi using Ansible

TODO.

## License

MIT

## Author

This project was created in 2016 by [Jeff Geerling](http://www.jeffgeerling.com/), author of [Ansible for DevOps](http://www.jeffgeerling.com/).
