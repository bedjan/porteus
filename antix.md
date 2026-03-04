Gru2Win, Ext2volume manager, Diusklinternal linux reader, antix 64 bit full iso


###########################################

Grub2Wim:


nsmod part_gpt
insmod part_msdos
insmod ext2

search --no-floppy --file --set=root /antix/antiX/vmlinuz

linux /antix/antiX/vmlinuz bdir=antiX from=hd toram persist_all quiet

initrd /antix/antiX/initrd.gz

#############################################

insmod part_gpt
insmod part_msdos
insmod ext2

search --no-floppy --file --set=root /antix/antiX/vmlinuz

linux /antix/antiX/vmlinuz bdir=antix/antiX buuid=BC57DE46-26AB-DC01-3041-DE4626ABDC01 from=hd toram persist_all

initrd /antix/antiX/initrd.gz

###########################################

sudo sh /media/sdb1/antix/antix.sh


#!/bin/bash

# 1. Pripojeni Windows (sda3)
sudo mkdir -p /media/win_c
sudo ntfsfix -d /dev/sda3
sudo mount -t ntfs-3g /dev/sda3 /media/win_c -o ro,uid=1000

# 2. Prenos qBittorrentu (bedri)
mkdir -p ~/.config/qBittorrent
mkdir -p ~/.local/share/qBittorrent/BT_backup
cp /media/win_c/Users/bedri/AppData/Roaming/qBittorrent/qBittorrent.ini ~/.config/qBittorrent/qBittorrent.conf
cp -r /media/win_c/Users/bedri/AppData/Local/qBittorrent/BT_backup/. ~/.local/share/qBittorrent/BT_backup/

# 3. HDMI a Oprava disku
xrandr --output HDMI-1 --auto --primary --right-of eDP-1
sudo blkid -t TYPE=ntfs -o device | xargs -L1 sudo ntfsfix -d

echo "HOTOVO - RESTARTUJ QBITTORRENT"
