Write-Host "--- Porteus & qBittorrent Generator ---" -ForegroundColor Cyan

# 1. Zjištění identity a disku
$winUser = $env:USERNAME
$winDrive = Get-Partition | Where-Object { $_.DriveLetter -eq 'C' }
$linuxDevice = if ((Get-Disk -Number $winDrive.DiskNumber).BusType -eq 'NVMe') { "nvme0n1p" } else { "sda" }
$finalDev = "/dev/$($linuxDevice)$($winDrive.PartitionNumber)"

# 2. Cesty pro Porteus (rootcopy)
$basePath = "C:\porteus\porteus\rootcopy"
$rcPath = "$basePath\etc\rc.d"
$qbConfPath = "$basePath\home\guest\.config\qBittorrent"
if (!(Test-Path $rcPath)) { New-Item -ItemType Directory -Force -Path $rcPath }
if (!(Test-Path $qbConfPath)) { New-Item -ItemType Directory -Force -Path $qbConfPath }

# 3. Kopírování nastavení qBittorrentu z Windows do Porteusu
Write-Host "Kopíruji nastavení qBittorrentu..." -ForegroundColor Yellow
$winQB = "$env:APPDATA\qBittorrent"
if (Test-Path $winQB) {
    Copy-Item "$winQB\*" $qbConfPath -Recurse -Force
}

# 4. GENEROVÁNÍ LINUX SKRIPTU (rc.local)
$linuxScript = @"
#!/bin/bash
# --- ČÁST PRO LINUX (Automaticky spouštěno Porteusem) ---

# Připojení Windows disku
mkdir -p /mnt/win_c
mount -t ntfs-3g -o uid=1000,gid=1000,dmask=022,fmask=133 $finalDev /mnt/win_c

# Propojení uživatelských složek
USER_HOME="/home/guest"
WIN_PATH="/mnt/win_c/Users/$winUser"

rm -rf `$USER_HOME/Downloads `$USER_HOME/Desktop `$USER_HOME/Documents
ln -s "`$WIN_PATH/Downloads" `$USER_HOME/Downloads
ln -s "`$WIN_PATH/Desktop"   `$USER_HOME/Desktop
ln -s "`$WIN_PATH/Documents" `$USER_HOME/Documents

# Oprava práv pro qBittorrent konfigy
chown -R guest:guest `$USER_HOME/.config

# Automatický start qBittorrentu pod uživatelem guest
export DISPLAY=:0
su guest -c "qbittorrent &"
"@

$linuxScript | Out-File -FilePath "$rcPath\rc.local" -Encoding ascii

Write-Host "Hotovo! Vypni Fast Boot a restartuj do Porteusu." -ForegroundColor Green
