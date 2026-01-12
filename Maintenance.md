# Maintenance

Maybe fixes pacman issues

```bash
pacman-key --refresh-keys  # Refresh pacman keys
```

**Remove Orphans**

```bash
sudo pacman -Rns $(pacman -Qtdq)  # Removes unused Packages
yay -Sc  # Removes Cache of YAY
# Also: https://ostechnix.com/recommended-way-clean-package-cache-arch-linux/
```

**Optimize pacman's database access speeds**

```bash
sudo pacman-optimize
```

**Make Arch Stable (Golden Rules)**

1. Do Backups with Timeshift and BtrFS

2. Update only every Week

3. Install Linux and Linux-LTS but us the LTS Kernal. If the System breaks, you can switch to the non LTS Kernal when the system is not booting. (use: `uname -sr` to check the current used kernel)

4. Ignore Linux Packages on Update (pacman.conf -> ignorepkg = Linux)

5. Do not install out of date packages! Use the AUR Page and search under package actions: *Flagged out-of-date (2019 )*. You can also see this in the aur-tool logs!

**Check for errors**

```bash
sudo systemctl --failed sudo journalctl -p 3 -xb
```

**(optional) Backup your System manually**

```bash
sudo rsync -aAXvP --delete --exclude=/dev/* --exclude=/proc/* --exclude=/sys/* --exclude=/tmp/* --exclude=/run/* --exclude=/mnt/* --exclude=/media/* --exclude=/lost+found --exclude=/home/.ecryptfs / /mnt/backupDestination/
```

AUR Checksum Skip

```bash
makepkg --skipchecksums -si**
```

**Set Route with net-tools manually**

```bash
# e.g.
sudo route add -net 172.16.0.0/16 tun0
```

**Cleanup Tips**

Clean PKG:

- sudo pacman -Sc

- or install *pacman-contrib*
  
  - paccache -h # help page
  
  - ... -d #Dry Mode (just list packages to remove)
  
  - .... -r #Remove Packages from Dry Mode

- oder mit paccache.timer jeden Monat automatisch bereinigen
  
  - sudo nano /etc/systemd/system/paccache.timer
  
  - For this enter:
    
    ```
    [Unit]
    Description=Clean-up old pacman pkg
    
    [Timer]
    OnCalendar=monthly
    Persistent=true
    
    [Install]
    WantedBy=multi-user.target
    ```

**Remove Orphans**

```bash
sudo pacman -R $(pacman -Qtdq)
```

**Clean Cache in /home**

```bash
du -sh ~/.cache/    # Size of Cache
rm -rf ~/.cache/*
```

**Remove old config files**

-> ~/.config und ~/.local/share

Remove them manualy! (only uninstalled files recommended)

**Find and remove duplicates, empty files, empty dirs, broken symlinks**

```bash
sudo pacman -S rmlint
rmlint --help
rmlint /home/user     #Scans folder and saves special script to remove findings (read results)
```

**Find the largest files and directories**

```bash
sudo pacman -S ncdu
ncdu   # Run tool (command line)
```

### Improve Performance

[https://wiki.archlinux.org/title/improving_performance](https://wiki.archlinux.org/title/improving_performance)