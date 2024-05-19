# personal-guide-fedora

This repository aims to keep track of every installation/process I make in my fedora setup. This way, I can always come here and check how I did something.

## General commands

### Set pcmanfm as default file manager:
```
xdg-mime default pcmanfm.desktop inode/directory application/x-gnome-saved-search
```

### Mount partition

The command ```fdisk -l``` lists all disk partitions available. When desired partition is chosen, e.g. /dev/sdb2, create directory where it will be mounted: ```sudo mkdir /mnt/disk1```; then mount it with ```sudo mount -t auto -v /dev/sdb2 /mnt/disk1/```.

Unmount it with ```sudo umount /dev/sdb2 -l```

### Create symbolic link of all files in a directory (if you're in the actual directory):

Update `ln -s -r * ~/.local/bin/`

This will link all files in directory to the path ~/.local/bin.

### Install ueberzug to preview images in ranger:

Install dependencies: `sudo dnf install python3-devel libXxf86vm-devel`

Install ueberzug: `pip install ueberzug`

Now that it is unmaintained, this is how to install it:
```bash
sudo apt install libxext-dev libx11-dev -y
git clone --depth 1 --branch 18.1.9 https://github.com/seebye/ueberzug.git
cd ueberzug
sudo python3 setup.py install
```

### Qalculate use period (.) instead of comma (,):

In file `~/.config/qalculate/qalc.cfg`, write `decimal_comma=0`. Set it to `-1` to ignore setting (use system's locale) or to `1` (to enable .).

### Rename Enter key

Enter key was named `KP_Enter`, instead of `Return`, so with command (requires xmodmap)
```
xmodmap -e "keysym KP_Enter = Return"
```
it's now solved. 

### Wayland

#### Waybar

To install waybar git version, `git clone` waybar repository, `meson build -Dexperimental=true` and `ninja build` +  install`.  

## Arch Linux

### Update pacman mirrors

```
sudo reflector --country 'Spain' --latest 5 --age 2 --fastest 5 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

### Replace two colors in svg

```
cp calendar.svg calendar2.svg; sed -i 's/#ff0000/#5EAAE8/g' calendar2.svg; sed -i 's/#00ffff/#AAC690/g' calendar2.svg
```

### Enable hw acceleration for NVIDIA:

Use the link:

https://danilw.github.io/blog/nvidia_linux_gpu_video_accceleration_webbrowsers/#short-insrtuction-to-nvidia-vaapi-driver

### Troubleshoot `xdg-desktop-portal` issues

For example, no file chooser dialog opens in flatpak apps:
```
/usr/lib64/xdg-desktop-portal -rv
```

### No wayland sessions show in gdm

Probably related to NVIDIA card and using `optimus-manager`. To fix it (source https://www.reddit.com/r/debian/comments/149jx0y/comment/jo62jsk/?utm_source=share&utm_medium=mweb3x&utm_name=mweb3xcss&utm_term=1&utm_content=share_button):

1. check `/etc/gdm3/custom.conf`, if there's a line `#WaylandEnable=false`, it needs to be uncommented and set to `true`.

2. check `/usr/lib/udev/rules.d/61-gdm.rules`, comment these lines: `#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"` and `#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"`
3. 

### Enable launch using discrete graphics in GNOME

When done, only right-clicking in application is needed to launch with `prime-run`. First, install `switcheroo-control`:
```
sudo pacman -S switcheroo-control
```
Then, just run
```
systemctl enable switcheroo-control.service
systemctl start switcheroo-control.service
```
