Write-Host "--- Porteus 2026: Ultimate Czech Configurator ---" -ForegroundColor Cyan

# 1. Příprava složek
$porteusDir = "C:\porteus\porteus"
$modulesDir = "$porteusDir\modules"
$rcPath = "$porteusDir\rootcopy\etc\rc.d"
if (!(Test-Path $modulesDir)) { New-Item -ItemType Directory -Force -Path $modulesDir }
if (!(Test-Path $rcPath)) { New-Item -ItemType Directory -Force -Path $rcPath }

# 2. Seznam modulů ke stažení (URL repozitář v5.0)
$baseUrl = "http://dl.porteus.org/x86_64/Porteus-v5.0/modules"
$downloads = @{
    "qBittorrent" = "$baseUrl/apps/qbittorrent-4.4.2-x86_64-1.xzm"
    "LibTorrent"  = "$baseUrl/apps/libtorrent-rasterbar-1.2.15-x86_64-1.xzm"
    "Kodi"        = "$baseUrl/apps/kodi-19.4-x86_64-1.xzm"
    "Chrome"      = "$baseUrl/browsers/google-chrome-101.0.4951.54-x86_64-1.xzm"
    "VLC"         = "$baseUrl/apps/vlc-3.0.17.4-x86_64-1.xzm"
    "DoubleCmd"   = "$baseUrl/system/doublecmd-gtk-1.0.5-x86_64-1.xzm"
}

# 3. Automatické stahování modulů
foreach ($item in $downloads.GetEnumerator()) {
    $fileName = Split-Path $item.Value -Leaf
    $destFile = Join-Path $modulesDir $fileName
    if (!(Test-Path $destFile)) {
        Write-Host "Stahuji $($item.Key)..." -ForegroundColor Yellow
        try { Invoke-WebRequest -Uri $item.Value -OutFile $destFile -ErrorAction Stop } catch { Write-Host "Chyba u $($item.Key)" -ForegroundColor Red }
    }
}

# 4. Detekce Windows disků
$winUser = $env:USERNAME
$winDrive = Get-Partition | Where-Object { $_.DriveLetter -eq 'C' }
$linuxDevice = if ((Get-Disk -Number $winDrive.DiskNumber).BusType -eq 'NVMe') { "nvme0n1p" } else { "sda" }
$finalDev = "/dev/$($linuxDevice)$($winDrive.PartitionNumber)"

# 5. Zápis finálního rc.local s ČEŠTINOU
$linuxScript = @"
#!/bin/bash
# --- NASTAVENÍ ČEŠTINY ---
loadkeys cz
localectl set-x11-keymap cz
echo "LANG=cs_CZ.UTF-8" > /etc/profile.d/lang.sh
export LANG=cs_CZ.UTF-8

# Připojení Windows disku C:
mkdir -p /mnt/win_c
mount -t ntfs-3g -o uid=1000,gid=1000,dmask=022,fmask=133 $finalDev /mnt/win_c

USER_HOME="/home/guest"
WIN_PATH="/mnt/win_c/Users/$winUser"

# Propojení složek (Downloads, Desktop, atd.)
for dir in Downloads Desktop Documents Pictures Videos Music; do
    rm -rf "`$USER_HOME/`$dir"
    ln -s "`$WIN_PATH/`$dir" "`$USER_HOME/`$dir"
done

# Oprava práv a automatický start programů
chown -R guest:guest `$USER_HOME
export DISPLAY=:0
(sleep 6 && su guest -c "qbittorrent") &
(sleep 8 && su guest -c "vlc") &
"@

$linuxScript | Out-File -FilePath "$rcPath\rc.local" -Encoding ascii

Write-Host "`nVŠE JE PŘIPRAVENO! Porteus bude česky a propojen s Windows." -ForegroundColor Green
