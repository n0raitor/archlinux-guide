# XFCE

```bash
sudo pacman -S xfce4 xfce4-goodies lightdm lightdm-gtk-greeter network-manager-applet 
sudo systemctl enable lightdm
```

## My Taste of XFCE
For Kali-Style use:
- Cantarell Regular 11
- Cantarell Regular 11
- Fira Code Medium 10
- Cantarell Bold 11
- Hinting: Medium
- Gray
- Scaling: 1,35

**Themes**
Use Flat-Remix

```bash
sudo pacman -S xfce4-whiskermenu-plugin xfce4-session xfce4-pulseaudio-plugin sound-theme-freedesktop sound-theme-smooth 
yay -S sound-theme-smooth xfce4-volumed-pulse xfce4-mixer xfce4-screensaver
```

**Popup Menu on Windows-Icon**
In Keyboard Shortcuts set xfce4-popup-whiskermenu run on windows-key

```bash
sudo pacman -S redshift
sudo systemctl enable redshift
```

Optional:
```bash
sudo pacman -S blackarch-config-xfce
```

Other: https://www.kali.org/docs/general-use/xfce-faq/
