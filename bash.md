su
# heslo je: toor
chmod +x /etc/rc.d/rc.local
/etc/rc.d/rc.local

##################################################

cat << 'EOF' > /tmp/setup.sh
#!/bin/bash

echo "--- Spouštím Porteus 5.0 CZ Konfigurátor ---"

# 1. NASTAVENÍ ČEŠTINY (okamžitě)
loadkeys cz
localectl set-x11-keymap cz
export LANG=cs_CZ.UTF-8
echo "Klávesnice přepnuta na CZ."

# 2. SLOŽKY (Najdeme, kde je Porteus nainstalován na disku)
# Hledáme složku modules na tvém Windows disku (sda2)
MOD_PATH=$(find /mnt/sda2 -name "modules" -type d | grep "porteus/modules" | head -n 1)

if [ -z "$MOD_PATH" ]; then
    echo "Složka modules nebyla na sda2 nalezena. Zkouším nvme..."
    MOD_PATH=$(find /mnt/nvme0n1p3 -name "modules" -type d | grep "porteus/modules" | head -n 1)
fi

echo "Cílová složka pro moduly: $MOD_PATH"

# 3. STAHOVÁNÍ PROGRAMŮ (přes get-app)
# get-app je chytrý stahovač Porteusu, který si najde funkční servery sám.
programs=("qbittorrent" "vlc" "kodi")

for prog in "${programs[@]}"; do
    echo "Stahuji $prog..."
    get-app $prog
    
    # Přesuneme stažený .xzm soubor z /tmp do tvé složky modules na disku
    mv /tmp/$prog*.xzm "$MOD_PATH/" 2>/dev/null
    echo "$prog stažen a uložen do $MOD_PATH"
done

# 4. AKTIVACE rc.local (aby čeština fungovala i po restartu)
chmod +x /etc/rc.d/rc.local
echo "Startovací skript aktivován."

echo "--- HOTOVO! ---"
echo "Restartuj Porteus a vše bude česky a s programy."
EOF

# Spuštění skriptu
chmod +x /tmp/setup.sh
bash /tmp/setup.sh
