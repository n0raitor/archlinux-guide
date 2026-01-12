# Arch Linux Guide 2026

This is my 2026 created guide of all things archlinux I learned over the past 7 Years. It is based on my old installation guide and tested with the recent archlinux image (note the Readme notice). 

Tested Release: 2026.01.01

Feel free to Follow the Markdown Files by starting from the lowest number to the highest. On DE (Desktop Environments): Only select one is recommended.

If any error occures, feel free to create an issue.

Have fun and go the Arch way!

## !!!Welcome!!!

In this Installation Guide, I will explain my default Setup for ArchLinux with LUKS Drive-Encryption and EFI.

You can follow the full installation guide and choose a desktop environment to install.

For a much simpler installation using a Desktop Environment from a Live Arch Linux OS, feel free to use my Prebuild Arch Linux Live IMAGES on SourceForge.

[![Download ArchLinux Live ISO Prebuild](https://a.fsdn.com/con/app/sf-download-button)](https://sourceforge.net/projects/archlinux-live-iso-prebuild/files/latest/download)

**How to Copy Paste / Install these Commands**

If you downloaded my ArchLinux Live ISO with Desktop Environment:

* If you wish to copy them via copy and paste, feel free to use my installation live iso. (Burn it via Rufus or Balena Etcher and plug in the usb and boot from it.)

* Open this github page and open a terminal to start the process and follow the guide.

* Feel free to skip to Update Database (if you are connected  to the Internet)

On Original ArchLinux Iso:

* Until the Desktop Environment is installed, you need to type all commands by hand.

* After the desktop environment is installed, feel free to copy and paste the code snippes.

Continue with **01_BaseInstall**

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

If you want to stick to the default way, continue to the file: **Arch-Standard-Encryption.md** and hope the best ;-). This method is by far the most reliable way of decrypting the root device. Starting the Systemd method below:

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

### Desktop Environment

Gnome:

```bash
sudo pacman -S gnome gdm gnome-shell-extensions gnome-tweaks extension-manager
sudo systemctl enable gdm.service
```

## Final Tweaks before Reboot

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

## !!!Post Installation!!!

**Updates**

```bash
# Update System
pacman -Syu

nano /etc/pacman.conf
# Uncomment Color
# (dep) Uncomment ParallelDownloads
# Add ILoveCandy (underneath ParallelDownloads)
# uncomment multilib Repo (2 Lines) (32bit Software Repo)+

pacman -Sy

### Segnature Security ###
pacman -S archlinux-keyring  # Update keyring
pacman-key --refresh-keys  # Refresh pacman keys

# If it fails (lots of red lines at once): Make sure your System Time is right
```

Feel free to reboot at this point to fix some user right issues

***!!!NOW STAY IN USER MODE!!!** to get the commands working

**Booting**

Num Lock activation: [Activating numlock on bootup - ArchWiki](https://wiki.archlinux.org/title/Activating_numlock_on_bootup)

**(Optional / Dep) Sound System**

```bash
sudo pacman -S alsa-plugins alsa-utils pulseaudio pulseaudio-alsa
sudo pacman -S pulseaudio-bluetooth  # Bluetooth Support
sudo pacman -S pulseaudio-equalizer  # Optional
```

**Tools / Utilities / Recommended Packages**

Configuration tools for Linux networking

```bash
sudo pacman -S xdg-user-dirs net-tools wget curl ufw gufw dialog usbutils zip unzip unrar p7zip lzop ntfs-3g gvfs timeshift gst-plugins-base gst-plugins-good gst-plugins-ugly
```

**Fonts**

These Fonts are for System Appearance and Document Creation Use-Cases

```bash
sudo pacman -S ttf-bitstream-vera ttf-dejavu ttf-liberation adobe-source-sans-pro-fonts ttf-anonymous-pro ttf-droid ttf-ubuntu-font-family ttf-roboto ttf-roboto-mono ttf-font-awesome ttf-fira-code
yay -S ttf-hackgen ttf-gentium-basic ttf-ms-fonts ttf-nerd-fonts-hack-complete-git ttf-mac-fonts
```

**Printer**

```bash
sudo pacman -S cups cups-pdf
sudo systemctl enable cups
sudo systemctt disable systemd-resolved.service
sudo systemctl enable avahi-daemon
```

**(optional / dep) Bluetooth**

```bash
sudo pacman -S bluez bluez-utils
sudo systemctl enable bluetooth
```

**YAY (AUR Installer CLI)**

Change to User - This is required, because yay can't run as root as it uses the fake root (which is non-root user context only) to install packages from AUR.

```bash
su <username>
```

```bash
# Install YAY - simple AUR Package Installation
cd ~
git clone https://aur.archlinux.org/yay.git
cd yay/
makepkg -si PKGBUILD
```

**Alternative Power Management Tools:**

[auto-cpufreq]([https://github.com/AdnanHodzic/auto-cpufreq)

```bash
yay -S auto-cpufreq
sudo systemctl enable auto-cpufreq
```

Powertop:

* **Powertop** is a tool provided by Intel to enable various powersaving modes in userspace, kernel and hardware. It is possible to monitor processes and show which of them are utilizing the CPU and wake it from its Idle-States, allowing to identify applications with particular high power demands.

```bash
sudo pacman -S powertop
```

### ZSH

```bash
sudo pacman -S zsh zsh-syntax-highlighting zsh-autosuggestions zsh-completions
yay -S zsh-theme-powerlevel10k
```

## Next Steps in the Desktop Environment

Create a Snapshot with **Timeshift**

* Open Timeshift and create a Snapshot Task

* Press Create and wait until the backup / snapshot is done

* Rename the Snapshot (add a Comment) "INIT"

For HiDPI (choose one):

- Open Settings -> Display and Adjust Scaling to 200%

- Open Gnome Tweaks -> Fonts and Adjust Scaling to 1.25

General INIT Maintenance:

- Go through the settings of Gnome and adjust it to your choice.
  
  - We will edit the default Applications later
  
  - Add Keyboard shortcut:
    
    - gnome-terminal (on CTRL + ALT + T)
      
      - nautilus (on CTRL + ALT + E)
        
        - xkill (on CTRL + ALT + ESC)

- Enable Firewall

- Gnome Num-Lock:

Set NumKey Enabled:

```bash
gsettings set org.gnome.desktop.peripherals.keyboard numlock-state true
```

**Get a Transparent Terminal**

```bash
yay -S gnome-terminal-transparency
#-> Type YES to remove the old terminal
```

**Extensions in the Browser**:

Prepare (Chromium Based Browser):

```bash
yay -S chrome-gnome-shell
```

Visit [https://extensions.gnome.org/](https://extensions.gnome.org/)

* User Themes

* Places Status Indicator

* AppIndicator and KStatusNotifierItem Support

* Removable Drive Menu

* Sound Input & Output Device Chooser

* Caffeine

* Arch Linux Updates Indicators

* Bluetooth Quick Connect

* OpenWeather

* Favourites in AppGrid

* ddterm

* Resource Monitor or System Monitor

* [Blur my Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/)
  
  * IMPORTANT: The default dock (or Dash-to-Dock Ext) can get a shadow effect by enabling this extension. Tweak the settings (especially: Panel and Dock, disable it) to get rid of it

* DEP: Dynamic Panel Transparency

* Frippery Move Clock

* DEP: Panel OSD

* Gnome 4x UI Improvements

* DEP: Shell Configurator

* Workspace Indicator

* NEW:
  
  * Desktop Cube
  
  * #Useless Gaps
  
  * #Fly-Pie
  
  * #Compiz windows effect
  
  * Toggle Night Light
  
  * Night Theme Switcher
  
  * Window Is Ready - Notification Remover

**Styles in Gnome**

For Kali-Style use:

- Cantarell Regular 11

- Cantarell Regular 11

- Fira Code Medium 10

- Cantarell Bold 11

- Hinting: Medium

- Gray

- Scaling: 1,35

**Themes**

Use Flat-Remix, Orchis, Nordic or Skeuos

**Arch Artworks**

```bash
yay -S archlinux-artwork
```

**Arch Wallpaper**

```bash
yay -S arch-logo-dark-wallpapers
yay -S arch-linux-2d-wallpapers
```

**Terminal Color Scheme**

[Gogh](http://mayccoll.github.io/Gogh/)

```bash
bash -c "$(curl -sLo- https://git.io/vQgMr)"gpar
```

**Wallpaper**

https://wallhaven.cc/

## !!!Programms!!!

This Section contains the most recommended programms to use on your OS

### (optional) Snap

```bash
yay -S snapd
sudo systemctl enable snapd.socket
sudo systemctl start snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install snap-store
```

### FlatPak

```bash
sudo pacman -S flatpak
# integriert das Flathub Repositorium, das eine zentrale Sammelstelle für flatpaks darstellt.
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

**Recommended Tools and Programs in one Block**

```bash
sudo pacman -S wireless_tools qbittorrent gnome-keyring evolution vlc obs-studio handbrake brasero audacity gimp gimp-help-de darktable krita calibre gnome-builder python-pip ipython gparted bleachbit xreader libreoffice-still libreoffice-still-de hunspell hunspell-en_us hunspell-de mythes aspell aspell-de aspell-en mythes mythes-en  mythes-de languagetool enchant libmythes speech-dispatcher protonmail-bridge-core kdenlive signal-desktop filelight baobab bleachbit glances htop gnome-disk-utility gparted hwinfo nano-syntax-highlighting neofetch gnome-firmware obsidian firefox gnome-builder pycharm-community-edition gnome-boxes

yay -S jdownloader2 openshot balena-etcher guitar-pro yed vscodium-bin github-desktop-bin zotero-bin libreoffice-extension-languagetool marktext-bin

pip3 install pylint
```

******

**(optional) Haskell**

```bash
pacman -S ghc
pacman -S cabal-install
pacman -S haskell-language-server
```

GHCup

```bash
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

To start a simple repl, run:

```bash
ghci
```

To start a new haskell project in the current directory, run:

```bash
cabal init --interactive
```

To install other GHC versions and tools, run:

```bash
ghcup tui
```

If you are new to Haskell, check out [First steps - GHCup](https://www.haskell.org/ghcup/steps/)

**VirtualBox**

Follow the instructions on the following [Site](https://wiki.archlinux.org/title/VirtualBox).

German: [VirtualBox – wiki.archlinux.de](https://wiki.archlinux.de/title/VirtualBox)

IMPORTANT: Arch module for linux and dkms for linux-lts.

Use modprobe vboxdrv.

```bash
sudo pacman -S virtualbox virtualbox-host-dkms linux-lts-headers
sudo modprobe vboxdrv
sudo nano /etc/modules-load.d/vboxdrv.conf
# Add vboxdrv
yay -S virtualbox-ext-oracle
```

**virt-manager**

```bash
sudo pacman -S virt-manager
```

**Bottles**

```bash
flatpak install flathub com.usebottles.bottles
```

**Kali Linux Shell**

If you want a more "Kali" like shell, feel free to install [this](https://wiki.archlinux.org/index.php/zsh) shell.

**Ubuntu System Optimizer - Stacer**

```bash
yay -S stacer-bin
```

## The Package Manager of Arch Linux

[Link](https://wiki.archlinux.de/title/Graphische_Paketmanager)

**Pamac**

A Package Manager for Arch Linux Pacman Packages

```bash
sudo pacman -S appstream-glib
yay -S archlinux-appstream-data-pamac
yay -S snapd-glib

git clone https://aur.archlinux.org/libpamac-aur.git
cd libpamac
nano PKGBUILD  # Edit PKGBUILD
# SET
ENABLE_FLATPAK=1
ENABLE_SNAPD=1
makepkg -si

yay -S pamac-aur
```

**Gnome Software**

```bash
sudo pacman -S gnome-softwarte
```

## Other Tools

**No Adobe - Use the Libre Graphics Suite**

[Follow this link](https://github.com/AppImage/AppImageKit/wiki/Libre-Graphics-Suite)

#### Install Awesome Nautilus Extensions

```bash
sudo pacman -S nautilus-terminal
```

## PEN and DFIR Tools

Requirement

```bash
git clone https://github.com/WqyJh/qwingraph_qt5.git
cd qwingraph_qt5
sudo ./install.sh
```

Installation:

```bash
sudo pacman -S nmap wireshark-qt hashcat
yay -S ida-free ghidra edb-debugger-git maltego burpsuite
```

**Blackarch**

```bash
curl -O https://blackarch.org/strap.sh
echo 00688950aaf5e5804d2abebb8d3d3ea1d28525ed strap.sh | sha1sum -c
chmod +x strap.sh
sudo ./strap.sh
sudo pacman -Syu
```

```bash
# To list all of the available tools, run
sudo pacman -Sgg | grep blackarch | cut -d' ' -f2 | sort -u

# To install a category of tools, run
sudo pacman -S blackarch-<category>

# To see the blackarch categories, run
sudo pacman -Sg | grep blackarch

# To search for a specific package, run
pacman -Ss <package_name>

# Note - it maybe be necessary to overwrite certain packages when installing blackarch tools. If
# you experience "failed to commit transaction" errors, use the --needed and --overwrite switches
# For example:
sudo pacman -Syyu --needed --overwrite='*' <wanted-package> 
```

---

**.RST Reader**

Restview:

```bash
sudo pacman --needed -S python-virtualenv
mkdir -p ~/venv/restview
cd ~/venv/restview
virtualenv venv # The second system I used just used 'virtualenv venv'
./venv/bin/pip install restview
./venv/bin/restview ~/path/MANUAL.rst
```

****

### Zim

Knowledge Database [Zim - ArchWiki](https://wiki.archlinux.org/title/Zim)

## Gaming

**Steam**

```bash
sudo pacman -S steam steam-native-runtime
```

**Minecraft**

```bash
yay -S minecraft-launcher

yay -S multimc-bin  # MC Mod Manage Launcher
```

[Gaming - ArchWiki](https://wiki.archlinux.org/title/gaming)

**Lutris**

```bash
sudo pacman -S lutris
```

[itch.io](https://en.wikipedia.org/wiki/itch.io) — Indie game store. [https://itch.io/](https://itch.io/) || [itch](https://aur.archlinux.org/packages/itch/)AUR

```bash
yay -S itch-bin 
```

## !!!Final Steps!!!

**Reduce Swappiness**

Forces System to use as much RAM as possible and reduces hard drive access

```bash
cat /proc/sys/vm/swappiness  # 60 by DEFAULT
sudo nano /etc/sysctl.d/100-archlinux.conf
-> vm.swappiness=10
reboot

cat /proc/sys/vm/swappiness  # should be 10 now
```

**Trim SSD**

Enable Trim for SSD - optimize performance of ssd-drive

```bash
sudo systemctl status fstrim.timer  # should be inactive, if not, skip to next heading
sudo systemctl start fstrim.timer
sudo systemctl status fstrim.timer  # should be active now
```

**Bash**

*use the dotfiles!*

```bash
sudo nano /etc/profile.d/editor.sh  # set default global editor
```

**Man Page - German Edition**

```bash
sudo pacman -S man-pages-de
alias man="LANG=de_DE.UTF-8 man"  # Write to ~/.bashrc or ~/.zshrc
```

**Most**

```bash
#Install Most for Manual Page highlighting
sudo pacman -S most
export PAGER=most
# See results on "man mv"
```

**Generate MIrrorlist of your Country**

```bash
reflector --verbose --country 'Germany' -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist
```

###### Generate SSH-Key

```bash
ssh-keygen -t rsa -b 4096 -o -a 100
```

**(optional) Enable SSH**

```bash
sudo systemctl enable sshd
sudo systemctl start sshd
# -> Already done, if not, run this commands
```

**OpenVPN**

```bash
sudo pacman -S openvpn networkmanager-openvpn
sudo systemctl restart NetworkManager
```

**OpenConnect**

```bash
sudo pacman -S openconnect networkmanager-openconnect
sudo systemctl restart NetworkManager
```

**Thumbnailer**

```bash
# Lightweight video thumbnailer that can be used by file managers.
sudo pacman -S ffmpegthumbnailer  
```

**Setup CUPS**

```bash
sudo pacman -S system-config-printer
```

**Install Your Brother Printer Driver**

```bash
yay -S <Printer Driver ID> (Search yay -Ss or in Pamac)  # e.g. brother-dcpj315w
sudo lpadmin -p dcpj315w -v lpd://${printer-ip}/BINARY_P1

# opt-dep

yay -S brscan3
```

Finally:

* get IP-Addresse of your Printer:

* Open CUPS web interface and use "search new Printer"

* Select your Network Printer and click "add"

* Follow the instructions and use the driver you just installed

* Use the "system-config-printer" application to remove the printer "dcpj315w"

* Print :)

NOTE: You may test this multiple times: Use the Test-Print Button to test the connectivity

## Set up Dual Boot with Windows

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

### Improve Performance

[https://wiki.archlinux.org/title/improving_performance](https://wiki.archlinux.org/title/improving_performance)