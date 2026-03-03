Write-Host "--- Oprava stahování modulů Porteus ---" -ForegroundColor Cyan

# 1. VYNUCENÍ PROTOKOLU (Důležité pro stahování z webu)
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

# 2. Nastavení složky
$modulesDir = "C:\porteus\porteus\modules"
if (!(Test-Path $modulesDir)) { New-Item -ItemType Directory -Force -Path $modulesDir }

# 3. Aktualizované odkazy (přesné verze z v5.0 repozitáře)
$baseUrl = "http://dl.porteus.org/x86_64/Porteus-v5.0/modules"
$downloads = @{
    "qBittorrent" = "$baseUrl/apps/qbittorrent-4.4.2-x86_64-1.xzm"
    "LibTorrent"  = "$baseUrl/apps/libtorrent-rasterbar-1.2.15-x86_64-1.xzm"
    "Kodi"        = "$baseUrl/apps/kodi-19.4-x86_64-1.xzm"
    "Chrome"      = "$baseUrl/browsers/google-chrome-101.0.4951.54-x86_64-1.xzm"
    "VLC"         = "$baseUrl/apps/vlc-3.0.17.4-x86_64-1.xzm"
}

# 4. Spuštění stahování s viditelným průběhem
foreach ($item in $downloads.GetEnumerator()) {
    $fileName = Split-Path $item.Value -Leaf
    $destFile = Join-Path $modulesDir $fileName
    
    Write-Host "Pokouším se stáhnout: $($item.Key)..." -ForegroundColor Yellow
    try {
        # Používáme WebClient, který je někdy spolehlivější než Invoke-WebRequest
        $client = New-Object System.Net.WebClient
        $client.DownloadFile($item.Value, $destFile)
        Write-Host "ÚSPĚCH: $($fileName) stažen." -ForegroundColor Green
    } catch {
        Write-Host "CHYBA: Nelze stáhnout $($item.Key). Odkaz na serveru se pravděpodobně změnil." -ForegroundColor Red
        Write-Host "Zkus to prosím ručně zde: $baseUrl" -ForegroundColor White
    }
}

# Otevře složku pro kontrolu
explorer $modulesDir
