(Optional) If you are using a separate home drive, use this to create an encrypted home partition (LVM, luks added beneath):

```bash
gdisk /dev/<home-device>

# n, 1, Enter, Enter, 8e00  
# Create Linux Filesystem Partition for the entire driveNow copy your Backups to the Home directory (sudo recommended)
```

```bash
# (Optional) For the Home Partition Encryption (if created above)
cryptsetup luksFormat /dev/<home-device>
cryptsetup open /dev/<home-device> crypt_home
# (Optional) If you are using the above created encrypted home partition
mkfs.ext4 /dev/mapper/crypt_home
# If you are using the above created encrypted home partition
mkdir /mnt/home
mount /dev/mapper/crypt_home /home

```

**(optional) If you are using the encrypted home partition**

```bash
lsblk -f  # Note down the UUID of your home-partition (not the mapper!)

nano /etc/crypttab   # Edit this file and insert the following row
# crypt_home     UUID=<UUID enter here>    none                    luks,timeout=180

nano /etc/fstab      # Check if this line is in place
# /dev/<home-device>
/dev/mapper/crypt_home    /home    ext4    rw,relatime     0 2

```

Edit `/etc/default/grub` if using Crypt_Home:

1. Remove the *#* in front of *GRUB_ENABLE_CRYPTODISK=y**
