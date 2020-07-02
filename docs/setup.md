# Basic Setup

In order to create a personal archiving system, we need to first setup the disks to be used for storage and backup.

Everybody has their own hardware limitations and budget, but for this specific tutorial, we shall be creating a mirror setup between _two_ 2TB disks in a ZFS poll, with a backup to another hard disk, which will be in another pool. All disks will have a initial encryption, on top of which, ZFS will live.

Tools for this setup:

* LUKS - encryption
* ZFS - backup and redundancy

## Step 0.

In order to create an encrypted ZFS pool, a well known encryption method (LUKS) can be used.

The idea is to encrypt all disks before creating a ZFS pool. In that way we will have ZFS pool created on encrypted devices that will not be usable without decrypting them first.

_Pros_

* LUKS is well known encryption mechanism, passed the test of time.
* Security firs: anyone with physical access to disks will not be able to access the data without decrypt them first.
* Easy to implement.

_Cons_

* As any other encryption from the same type the data is encrypted at rest.
* Zpool need to be manually imported after each reboot.

## Step 1.

Identify disks for encryption.

``` bash
lsblk

NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda            8:0    0 223.6G  0 disk
├─sda1         8:1    0     1G  0 part /boot
└─sda2         8:2    0   222G  0 part
  ├─vg0-root 252:0    0    32G  0 lvm  /
  ├─vg0-swap 252:1    0    64G  0 lvm  [SWAP]
  └─vg0-home 252:2    0   100G  0 lvm  /home
sdb            8:16   0   9.1T  0 disk
sdc            8:32   0   9.1T  0 disk
sdd            8:48   0   9.1T  0 disk
sde            8:64   0   9.1T  0 disk
sdf            8:80   0   9.1T  0 disk
sdg            8:96   0   9.1T  0 disk
```

After we have picked what we want to encrypt, we can execute the following command:

`cryptsetup -v --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-urandom --verify-passphrase luksFormat /dev/sd*`

Of course, you can specify disks separately, by putting instead of the `*`, the disk last character, like `/dev/sdb /dev/sdc`, etc.

Then decrypt the devices before creating a ZFS pool on them.

`cryptsetup open /dev/sd* disk#`

_Note:_ Each disk need to be decrypted separately OR a simple loop can be used to go threw all identified disks and to decrypt them. Decrypted device file will be located under `/dev` folder.

## Step 2.

Create the ZFS pool:

`zpool create pas-poll mirror /dev/disk1 /dev/disk2`
