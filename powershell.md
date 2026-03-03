Write-Host "--- Porteus 2026: MEGA Auto-Downloader ---" -ForegroundColor Cyan

# 1. Příprava složek
$porteusDir = "C:\porteus\porteus"
$modulesDir = "$porteusDir\modules"
$rcPath = "$porteusDir\rootcopy\etc\rc.d"
if (!(Test-Path $modulesDir)) { New-Item -ItemType Directory -Force -Path $modulesDir }
if (!(Test-Path $rcPath)) { New-Item -ItemType Directory -Force -Path $rcPath }

# 2. Rozšířený seznam modulů (URL repozitář v5.0)
$baseUrl = "http://dl.porteus.org/x86_64/Porteus-v5.0/modules"
$downloads = @{
    "qBittorrent" = "$baseUrl/apps/qbittorrent-4.4.2-x86_64-1.xzm"
    "LibTorrent"  = "$baseUrl/apps/libtorrent-rasterbar-1.2.15-x86_64-1.xzm"
    "Kodi"        = "$baseUrl/apps/kodi-19.4-x86_64-1.xzm"
    "Chrome"      = "$baseUrl/browsers/google-chrome-101.0.4951.54-x86_64-1.xzm"
    "VLC"         = "$baseUrl/apps/vlc-3.0.17.4-x86_64-1.xzm"
    "DoubleCmd"   = "$baseUrl/system/doublecmd-gtk-1.0.5-x86_64-1.xzm"
    "PCManFM"     = "$baseUrl/system/pcmanfm-1.3.1-x86_64-1.xzm"
}

# 3. Automatické stahování
foreach ($item in $downloads.GetEnumerator()) {
    $fileName = Split-Path $item.Value -Leaf
    $destFile = Join-Path $modulesDir $fileName
    
    if (!(Test-Path $destFile)) {
        Write-Host "Stahuji $($item.Key)..." -ForegroundColor Yellow
        try {
            Invoke-WebRequest -Uri $item.Value -OutFile $destFile -ErrorAction Stop
        } catch {
            Write-Host "Chyba při stahování $($item.Key). Verze na serveru se možná změnila." -ForegroundColor Red
        }
    } else {
        Write-Host "$($item.Key) již máme." -ForegroundColor Gray
    }
}

# 4. Detekce Windows prostředí
$winUser = $env:USERNAME
$winDrive = Get-Partition | Where-Object { $_.DriveLetter -eq 'C' }
$linuxDevice = if ((Get-Disk -Number $winDrive.DiskNumber).BusType -eq 'NVMe') { "nvme0n1p" } else { "sda" }
$finalDev = "/dev/$($linuxDevice)$($winDrive.PartitionNumber)"

# 5. Finální Linux Script (rc.local)
$linuxScript = @"
#!/bin/bash
# Připojení Windows disku C:
mkdir -p /mnt/win_c
mount -t ntfs-3g -o uid=1000,gid=1000,dmask=022,fmask=133 $finalDev /mnt/win_c

USER_HOME="/home/guest"
WIN_PATH="/mnt/win_c/Users/$winUser"

# Propojení VŠECH klíčových složek Windows -> Linux
for dir in Downloads Desktop Documents Pictures Videos Music; do
    rm -rf "`$USER_HOME/`$dir"
    ln -s "`$WIN_PATH/`$dir" "`$USER_HOME/`$dir"
done

# Oprava práv a automatický start qBittorrent
chown -R guest:guest `$USER_HOME
export DISPLAY=:0
(sleep 6 && su guest -c "qbittorrent") &
"@

$linuxScript | Out-File -FilePath "$rcPath\rc.local" -Encoding ascii

Write-Host "`nVŠE PŘIPRAVENO!" -ForegroundColor Green
Write-Host "Složka s moduly obsahuje: $( (Get-ChildItem $modulesDir).Count ) souborů."
