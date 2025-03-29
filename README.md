# Personal guide

This repository aims to keep track of every installation/process I make in my setups. This way, I can always come here and check how I did something.

## Set pcmanfm as default file manager

Execute

```
xdg-mime default pcmanfm.desktop inode/directory application/x-gnome-saved-search
```

## Mount partition

The command `fdisk -l` lists all disk partitions available. When desired partition is chosen, e.g. `/dev/sdb2`, create directory where it will be mounted:

```bash
sudo mkdir /mnt/disk1
```

then mount it with

```bash
sudo mount -t auto -v /dev/sdb2 /mnt/disk1/
```

Unmount it with

```bash
sudo umount /deb/sdb2 -l
```

## Create symbolic link of all files in a directory (if you're in the actual directory)

The command

```bash
ln -s -r * ~/.local/bin/
```

will link all files in directory to the path `~/.local/bin`.

## Install ueberzug to preview images in ranger (Fedora)

- When it was maintained:
  Install dependencies:

```bash
sudo dnf install python3-devel libXxf86vm-devel
```

Install ueberzug:

```bash
pip install ueberzug
```

- Now that it is unmaintained, this is how to install it: (replace `apt` and its packages with the required ones)

```bash
sudo apt install libxext-dev libx11-dev -y
git clone --depth 1 --branch 18.1.9 https://github.com/seebye/ueberzug.git
cd ueberzug
sudo python3 setup.py install
```

## Qalculate use period (.) instead of comma (,)

In file `~/.config/qalculate/qalc.cfg`, write `decimal_comma=0`. Set it to `-1` to ignore setting (use system's locale) or to `1` (to enable .).

## Rename Enter key

Enter key was named `KP_Enter`, instead of `Return`, so with command (requires `xmodmap`)

```bash
xmodmap -e "keysym KP_Enter = Return"
```

it's now solved.

## Install `waybar` from source

To install `waybar` git version, `git clone` waybar repository, `meson build -Dexperimental=true` and `ninja build` +  `install`.  

## Arch Linux specific commands

### Update pacman mirrors

```bash
sudo reflector --country 'Spain' --latest 5 --age 2 --fastest 5 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

### Uninstall orphan packages

```bash
sudo pacman -Qdtq | sudo pacman -Rns -
```

### Clean pacman cache

```bash
sudo paccache -r
```

### Clean yay cache

```bash
yay -Sc --aur
```

## Replace two colors in svg

```bash
cp calendar.svg calendar2.svg; sed -i 's/#ff0000/#5EAAE8/g' calendar2.svg; sed -i 's/#00ffff/#AAC690/g' calendar2.svg
```

## Enable hw acceleration for NVIDIA

Use this [link](https://danilw.github.io/blog/nvidia_linux_gpu_video_accceleration_webbrowsers/#short-insrtuction-to-nvidia-vaapi-driver)

## Troubleshoot `xdg-desktop-portal` issues

For example, no file chooser dialog opens in flatpak apps:

```bash
/usr/lib64/xdg-desktop-portal -rv
```

## No wayland sessions show in gdm

Probably related to NVIDIA card and using `optimus-manager`. To fix it ([source](https://www.reddit.com/r/debian/comments/149jx0y/comment/jo62jsk/?utm_source=share&utm_medium=mweb3x&utm_name=mweb3xcss&utm_term=1&utm_content=share_button)):

1. check `/etc/gdm3/custom.conf`, if there's a line `#WaylandEnable=false`, it needs to be uncommented and set to `true`.
2. check `/usr/lib/udev/rules.d/61-gdm.rules`, comment these lines: `#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"` and `#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"`

## Enable launch using discrete graphics in GNOME

When done, only right-clicking in application is needed to launch with `prime-run`. First, install `switcheroo-control`:

```bash
sudo pacman -S switcheroo-control
```

Then, just run

```bash
systemctl enable switcheroo-control.service
systemctl start switcheroo-control.service
```

In fedora, it is included by default

## Black wallpaper after suspend on GNOME

As stated [here](https://forums.developer.nvidia.com/t/fixed-suspend-resume-issues-with-the-driver-version-470/187150/3) the workaround is:

```bash
sudo systemctl stop nvidia-suspend.service
sudo systemctl stop nvidia-hibernate.service
sudo systemctl stop nvidia-resume.service

sudo systemctl disable nvidia-suspend.service
sudo systemctl disable nvidia-hibernate.service
sudo systemctl disable nvidia-resume.service

sudo mv /lib/systemd/system-sleep/nvidia ~/nvidia.bak
```

and reboot. Original content of `nvidia.bak` is

```bash
#!/bin/sh

case "$1" in
post)
/usr/bin/nvidia-sleep.sh "resume"
;;
esac
```

Thing above didn't work always.

## NVIDIA as primary GPU in GNOME

One can use NVIDIA as primary GPU in GNOME (`mutter`) using a `udev` rule: in file `/usr/lib/udev/rules.d/61-mutter-primary-gpu.rules`

```
ENV{DEVNAME}=="/dev/dri/card1", TAG+="mutter-device-preferred-primary"
```

Alternatively, one can choose the card by pci (so that it doesn't fail). The `tag` is given by

```bash
udevadm info --query=property --property=ID_PATH_TAG /dev/dri/card1
```

and then just set it to `ENV{DEVLINKS}=="/dev/dri/by-path/pci-<card>-card", TAG+="mutter-device-preferred-primary"`, which would be

```
ENV{DEVNAME}=="/dev/dri/by-path/pci-0000\:00\:02.0-card",
TAG+="mutter-device-preferred-primary"
```

## using NVIDIA as primary GNOME GPU

Suspend works really bad in this scenario, so I tried the following:

`gnome-shell` is trying to talk to the NVIDIA driver after it has already gone into suspend, so it can’t respond. Linux tries to freeze the task, but fails because `gnome-shell` is waiting for a response from the driver and can’t be frozen.

The solution is to manually suspend `gnome-shell` using the STOP signal before the NVIDIA driver goes to suspend. Then use the `CONT` signal on resume.

`/usr/local/bin/suspend-gnome-shell.sh`

```bash
#!/bin/bash

case "$1" in
suspend)
killall -STOP gnome-shell
;;
resume)
killall -CONT gnome-shell
;;
esac
```

As this is a script it needs executable permissions:

```bash
sudo chmod +x /usr/local/bin/suspend-gnome-shell.sh
```

`/etc/systemd/system/gnome-shell-suspend.service`

```ini
[Unit]
Description=Suspend gnome-shell
Before=systemd-suspend.service
Before=systemd-hibernate.service
Before=nvidia-suspend.service
Before=nvidia-hibernate.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/suspend-gnome-shell.sh suspend

[Install]
WantedBy=systemd-suspend.service
WantedBy=systemd-hibernate.service
```

`/etc/systemd/system/gnome-shell-resume.service`

```ini
[Unit]
Description=Resume gnome-shell
After=systemd-suspend.service
After=systemd-hibernate.service
After=nvidia-resume.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/suspend-gnome-shell.sh resume

[Install]
WantedBy=systemd-suspend.service
WantedBy=systemd-hibernate.service
```

Then just enable the two new systemd units:

```bash
systemctl daemon-reload
systemctl enable gnome-shell-suspend
systemctl enable gnome-shell-resume
```

This should interrupt gnome-shell in time so it’s not trying to access the graphics hardware. It worked for me.

## Crear una máquina virtual (VM) de Nixos

Al crear la máquina virtual de `Nixos` en `virt-manager`, darle a personalizar antes de crear la VM. En el menú recién abierto, poner el arranque en UEFI (NO EN BIOS).

## Winapps

Algunos errores:

- El servicio `iptables` no estaba activado, así que la conexión no funcionaba. Para habilitarlo,

```bash
sudo systemctl enable iptables.service
```

- Rendimiento muy malo: usar las mejoras de [farmerbb](https://www.reddit.com/r/ChromeOSFlex/comments/ucno4b/comment/i6cviv8/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

# Compartir una carpeta con la VM de Windows

Seguir los pasos de [askubuntu](https://askubuntu.com/a/1513697).

# Nuevo intento de suspender en NVIDIA

- [Fuente](https://www.reddit.com/r/kde/comments/131jyog/comment/ji2vrsx/?utm_source=share&utm_medium=web2x&context=3).
- Ejecutar

```bash
sudo systemctl enable nvidia-resume
```

- Crear el archivo `/etc/modprobe.d/nvidia-power-management.conf` y escribir

```
options nvidia NVreg_PreserveVideoMemoryAllocations=1 NVreg_TemporaryFilePath=/var/tmp  
```

# NixOS: crear/instalar un paquete propio

- Seguir [este enlace](https://unix.stackexchange.com/a/717169).

# Repair mp4 video to send it over whatsapp

```bash
ffmpeg -i original_video.mp4 -c:v libx264 -profile:v baseline -level 3.0 -pix_fmt yuv420p sendable_video.mp4
```
