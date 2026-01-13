# Gnome Setup

Gnome:

```bash
sudo pacman -S gnome gdm gnome-shell-extensions gnome-terminal gnome-tweaks extension-manager
sudo pacman -S gnome-shell-extension-caffeine
# Install and Remove not useful software
sudo pacman -S gnome-circle gnome-extra
sudo pacman -Rns amberol gnome-calls epiphany gnome-console audio-sharing health komikku warp webfont-kit-generator mousai polari junction forge-sparks 
sudo systemctl enable gdm.service
```

## HiDPI Support of Desktop Environments

[Source](https://archived.forum.manjaro.org/t/hidpi-support-in-manjaro/26088)

**Gnome**

HiDPI Support in Gnome is excellent. It auto-scales by default and is completely usable out of the box

To install gnome reference the appropriate Wiki page 679

Gnome autoscales the interface by default. If it doesn't you can run the run the "dconf Editor" and go under org->gnome->desktop->interface->scaling-factor and change it manually

To change the cursor size run the "dconf Editor" and go under org->gnome->desktop->interface->cursor-size

**GDM**

Like Gnome, GDM auto-scales by default

**Activating NumLock on Bootup**

```bash
gsettings set org.gnome.desktop.peripherals.keyboard numlock-state true
```

In order to remember last state of numlock key (whether you disabled or enabled), use:

```bash
gsettings set org.gnome.desktop.peripherals.keyboard remember-numlock-state true
```

Note: The key *numlock-state* was moved from *org.gnome.settings-daemon.peripherals.keyboard* since GNOME 3.34

Alternatively, you can use add *numlockx* on (from numlockx to a startup script, like */.bashrc* (if using Bash) or */.profile.*

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

#### Install Awesome Nautilus Extensions

```bash
sudo pacman -S nautilus-terminal
```