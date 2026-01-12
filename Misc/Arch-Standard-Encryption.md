# Alternative Guide how to Encrypt the Root Partition with encrypt instead of systemd encrypt

This method is less reliable and will not work on every Laptop (e.g. in my case an ASUS Laptop -> not working)

**Edit mkinitcpio.conf and generate**

```bash
nano /etc/mkinitcpio.conf
```

On Hooks: Add these words, separated with a space, between *block* and *filesystem*:

* encrypt

**Bootloader**

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux-Grub --recheck
```

**Configure Root Partition Decryption on Startup**

Use to get the UUID of the Encrypted and Decrypted device (Take a photo or note)

```bash
blkid -o value -s UUID /dev/<diskpartitionwithencryption> >> /etc/default/grub
blkid -o value -s UUID /dev/mapper/cryptroot >> /etc/default/grub
```

Now edit:

```bash
nano /etc/default/grub
```

Go to the Bottom and use the first UUID (encrypted id) with `CTRL + K` to cut and between GRUB_CMDLINE_LINUX_DEFAULT and GRUB_CMDLINE_LINUX Press `CTRL + U` . Do the Same with the other UUID (Decrypted id) and paste it after the first UUID and add after it a `"` to close the command.

Now format the Command as such:

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet cryptdevice=UUID=<ENC-UUID>:cryptroot root=UUID=<DEC-UUID>"
```

Continue on the Main Readme under: **Continue here if you used the Arch-Standard-Encryption**

(Search it)