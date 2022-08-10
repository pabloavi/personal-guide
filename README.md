# personal-guide-fedora

This repository aims to keep track of every installation/process I make in my fedora setup. This way, I can always come here and check how I did something.

## General commands

- Set pcmanfm as default file manager:
```
xdg-mime default pcmanfm.desktop inode/directory application/x-gnome-saved-search
```

- Mount partition
The command ```fdisk -l``` lists all disk partitions available. When desired partition is chosen, e.g. /dev/sdb2, create directory where it will be mounted: ```sudo mkdir /mnt/disk1```; then mount it with ```sudo mount -t auto -v /dev/sdb2 /mnt/disk1/```.

Unmount it with ```sudo umount /dev/sdb2 -l```
