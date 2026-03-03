# 1. Cesty
$base = "C:\porteus\porteus"
$rcDir = "$base\rootcopy\etc\rc.d"
if (!(Test-Path $rcDir)) { New-Item -ItemType Directory -Force -Path $rcDir }

# 2. Skript pro ČEŠTINU a PROPOJENÍ (rc.local)
$linuxScript = @"
#!/bin/bash
loadkeys cz
localectl set-x11-keymap cz
export LANG=cs_CZ.UTF-8

# Pripojeni Windows disku
mkdir -p /mnt/win_c
mount -t ntfs-3g -o uid=1000,gid=1000 /dev/sda2 /mnt/win_c 2>/dev/null || mount -t ntfs-3g -o uid=1000,gid=1000 /dev/nvme0n1p3 /mnt/win_c

# Propojeni plochy
W_PATH="/mnt/win_c/Users/$env:USERNAME"
for d in Desktop Downloads Documents; do
    rm -rf /home/guest/`$d
    ln -s "`$W_PATH/`$d" /home/guest/`$d
done
chown -R guest:guest /home/guest
"@

$linuxScript | Out-File -FilePath "$rcDir\rc.local" -Encoding ascii
Write-Host "Příprava hotova! Teď restartuj do Porteusu." -ForegroundColor Green
