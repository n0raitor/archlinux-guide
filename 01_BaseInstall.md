## !!!Preparation!!!

**Run the Arch Linux Disc**

Burn and Run the ArchLinux ISO until the command prompt appears.

**Optional Font Size Increase**

```bash
setfont -d
```

**Load Keyboard Layout**

```bash
loadkeys de
# Use localectl list-keymaps if you want to see all possible layouts
```

**Internet Connection**

```bash
ip a # check internet connection (alternative: ip link)
```

**On Wifi**

```bash
iwctl

# Use help for support
device list  # list network devices
station <device> scan  # scan for networks
station <device> get-networks  # get all networks
station <device> connect SSID  # login into your wifi

quit
```

**On LAN**

```bash
dhcpcd <ethernet-adapter>  # "ip a" to get the available adapter 
```

**Test Connection**

```bash
ping google.com
```

**System Clock Check**

Check if the output of this command is representing the current UTC Time:

```bash
timedatectl
```

On Error:

```bash
timedatectl set-ntp true
```

**Update Database**

```bash
pacman -Syyy
```

## !!!Base Install!!!

```bash
export EDITOR=nano
```

**Configure Partitions**

Untested Alternative with Gui: cfdisk

I use *gdisk* for GPT Tables

```bash
lsblk  # Print Devices

gdisk /dev/<os-device>  

# n, 128, -1000M, Enter, ef00  # Create EFI Partition
# n, 1, Enter, Enter, Enter
# p 
# w
```

**Encrypt Root Partition**

```bash
cryptsetup luksFormat /dev/<os-device>1  # Root Partition Device
# Confirm and Enter Passphrase.

cryptsetup open /dev/<os-device>1 cryptroot  # Open Created LVM Device
# Enter Passphrase
```

**Format Logical Volumes and Partitions**

```bash
mkfs.fat -F32 /dev/<os-device>128
mkfs.ext4 /dev/mapper/cryptroot
```

**Mount Partitions**

```bash
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<os-device>128 /mnt/boot
```

**(Optional) Create Local Mirror List**

Just for more download speed

*Note: This is for Server in Germany, feel free to tweak this command with your location*

```bash
reflector --verbose --country 'Germany' -l 20 -p https --sort rate --save /etc/pacman.d/mirrorlist

# Alternativ
nano /etc/pacman.d/mirrorlist
```

**Install Base Packages**

```bash
pacstrap /mnt base base-devel linux-lts linux linux-headers linux-lts-headers linux-firmware nano networkmanager dhcpcd lvm2 reflector git cryptsetup grub efibootmgr dosfstools os-prober mtools iwd openssh
```

## !!!Configure System!!!

*Note: Again, this is for Germany and German, feel free to use your timezone / language / locale informations*

**Gererate fstab**

```bash
genfstab -U /mnt >> /mnt/etc/fstab  # (alt. change -U to -L to use Label instead of UUID
cat /mnt/etc/fstab  # check gen
```

**Switch to Arch-Chroot**

```bash
arch-chroot /mnt  # Change to Root Directory
```

**Set Time**

```bash
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime  # Set Timezone
hwclock --systohc # Sync Hardware-Clock - Run to generate /etc/adjtime
```

**Set Locale**

```bash
echo LANG=de_DE.UTF-8 > /etc/locale.conf  # Set Language
nano /etc/locale.gen
# Uncomment this:
#de_DE.UTF-8 UTF-8
#de_DE ISO-8859-1
#de_DE@euro ISO-8859-15
#en_US.UTF-8
```

```bash
locale-gen
```

**Set Computer Hostname**

```bash
echo <myhost> > /etc/hostname
```

**Set Keyboard Layout (optional)** 

```bash
echo KEYMAP=de-latin1 > /etc/vconsole.conf
```

**Set Font (optional)**

```bash
echo FONT=lat9w-16 >> /etc/vconsole.conf
```

**Set Root Password**

```bash
passwd
```

**Edit Hosts file (optional)** 

```bash
nano /etc/hosts  # set Hosts file
```

```
# <ip-address>    <hostname.domain.org>        <hostname>
127.0.0.1         localhost.localdomain        localhost
::1               localhost.localdomain        localhost
127.0.0.1         <hostname>.localdomain       <hostname>
```

**Edit Mirror List** (Optional -> Later possible too, recommended for better download speed)

```bash
reflector --verbose --country 'Germany' -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist
```

Alternative:

```bash
nano /etc/pacman.d/mirrorlist
```

**Edit Visudo (Sudo)**

```bash
EDITOR=nano visudo
--> Uncomment: # %wheel ALL=(ALL) ALL 
```

**Add your Accounts**

```bash
useradd -m -G wheel -s /bin/bash <username>
passwd <name>

# Optional
gpasswd -a <username> audio
gpasswd -a <username> video
gpasswd -a <username> games
gpasswd -a <username> power
gpasswd -a <username> optical
```

**Configure Root Drive Decryption using Systemd-Encrypt**

Important: This is a method that works for me and i had to figure this out the hard way (2 Days of error solving), due to some Laptops like Asus Laptops have issues with the Arch Default Encryption Method in the mkinitcpio *encrypt* hook due to the device will not get recognized fast enough and an error will happen in the boot process and it is nearly impossible to get the real reason other than trying other methods.

If you want to stick to the default way, continue to the file: **Misc/Arch-Standard-Encryption.md** and hope the best ;-). This method is by far the most reliable way of decrypting the root device. Starting the Systemd method below:

----

**Edit mkinitcpio.conf and generate**

```bash
nano /etc/mkinitcpio.conf
```

On Hooks: Add these words, separated with a space, between *block* and *filesystem*:

* sd-encrypt

**Bootloader**

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux-Grub --recheck
```

**Configure Root Partition Decryption on Startup**

Use to get the UUID of the Encrypted and Decrypted device (Take a photo or note)

```bash
blkid -o value -s UUID /dev/<diskpartitionwithencryption> >> /etc/default/grub
blkid -o value -s UUID /dev/<diskpartitionwithencryption> >> /etc/crypttab
```

Now edit:

```bash
nano /etc/default/grub
```

Go to the Bottom and use the UUID (encrypted id) with `CTRL + K` to cut and between GRUB_CMDLINE_LINUX_DEFAULT and GRUB_CMDLINE_LINUX Press `CTRL + U` and add after it a `"` to close the command.

Now format the Command as such:

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet rd.luks.uuid=<UUID>"
```

Now edit:

```bash
nano /etc/crypttab
```

Format it like this:

```
cryptroot   UUID=<UUID>  none  luks
```

----------------

**Continue here if you used the Arch-Standard-Encryption**:

**Retaining boot messages (optional)**

If you wish to get more detailed boot messages on startup, configure this:

```bash
nano /etc/systemd/system/getty.target.wants/getty@tty1.service

# Set
[Service]
TTYVTDisallocate=no
```

Fix Quiet Log in Grub Config:

```bash
nano /etc/default/grub
# Remove the "quiet" on GRUB_CMDLINE_LINUX_DEFAULT
```

**Generate mkinitcpio Config**

Run both for bleeding edge and lts kernel support / integration

```bash
mkinitcpio -p linux-lts
mkinitcpio -p linux
```

**Generate Grub Configuration**

```bash
grub-mkconfig -o /boot/grub/grub.cfg  # Generate Grub config file
```

## Display and Desktop Environment

### GPU / Display drivers

Find out your Device:

```bash
sudo pacman -Sy pciutils
lspci -k | grep -EA3 'VGA|3D'
```

NVIDIA - Open source:

```bash
sudo pacman -S xf86-video-nouveau mesa mesa-utils

# optional # c
# virtualbox-guest-utils  # (For Virtualbox)
```

NVIDIA - Prop:

https://wiki.archlinux.org/title/NVIDIA

```bash
lspci -k -d ::03xx
# In my case: GMXXX -> nvidia-580xx-dkmsA
git clone https://aur.archlinux.org/nvidia-580xx-dkms.git
cd nvidia-580xx-dkms
makepkg -si PKGBUILD
```

Intel or AMD:

```bash
sudo pacman -S mesa mesa-utils intel-media-driver
```

AMD:

```bash
sudo pacman -S mesa mesa-utils libva-mesa-driver
```

VM:

```bash
sudo pacman -S virtualbox-guest-utils  # (For Virtualbox)
```

**Microcode**

*Note: Replace the intel with amd, if your cpu is an amd-cpu*

```bash
sudo pacman -S intel-ucode
```

**Power management - Laptops (Choose One)**

TLP:

```bash
sudo pacman -S tlp tlp-rdw
sudo systemctl enable tlp --now
```

**Optional: Network-Manager**

```bash
systemctl enable NetworkManager
systemctl enable sshd   # Optional
```


**Reboot**

```bash
exit  # leave arch-chroot
umount -a  # Unmount all devices
reboot
```

Continue to **02_DesktopEnvironment**