# Set up Dual Boot with Windows

In general

```bash
 sudo pacman -S os-prober
```

```bash
 sudo pacman -S ntfs-3g  #opt: If target Os is Windows
```

**Mount Dir of OS**

```bash
sudo mkdir -p /mnt/os-mount

sudo mount /dev/<partition-of-os-to-dual-boot> /mnt/os-mount
```

**run os-prober**

```bash
os-prober
```

**recreate Grub Config**

```bash
 grub-mkconfig -o /boot/grub/grub.cfg
```

If a message appears: *Os-prober will not be executed*

Solution: Set the named constant to the default/grub file as False (don't forget to uncommand the line)
