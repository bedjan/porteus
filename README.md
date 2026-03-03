
ps-update


V Porteusu 5.0 (a obecně v Slackware-based systémech) se v příkazovém řádku (CLI) aplikace neinstalují jako ve Windows nebo Ubuntu. Tady se pracuje s moduly (.xzm), které se neinstalují na disk, ale "připojují" do paměti RAM.

Zde je přehled programů a postupů, jak to v CLI (terminálu) dělat:
1. get-app (Nejjednodušší a nejrychlejší)

Toto je hlavní skript Porteusu pro stahování aplikací z repozitářů Slackwaru a jejich automatický převod na Porteus moduly.

    Příkaz: get-app název_programu

    Příklad: get-app qbittorrent

    Jak to funguje: 1.  Skript najde balíček na serverech Slackware.
    2.  Stáhne ho do /tmp.
    3.  Stáhne všechny chybějící závislosti (knihovny).
    4.  Všechno to zabalí do jednoho .xzm souboru.
    5.  Zeptej se tě, jestli ho chceš aktivovat.

2. usm (Unified Slackware Manager)

Starší, ale velmi silný nástroj pro správu balíčků. Umí prohledávat pět různých repozitářů najednou.

    Aktualizace databáze: usm -u all (udělej vždy jako první)

    Hledání: usm -s qbittorrent

    Stažení: usm -g qbittorrent

    Výhoda: Najde i velmi specifické knihovny, které get-app někdy mine.

3. txz2xzm (Ruční převod)

Pokud si stáhneš balíček pro Slackware (koncovka .txz nebo .tgz) ručně z webu (třeba ze slackware.uk), musíš ho převést na formát Porteusu.

    Příkaz: txz2xzm balik.txz balik.xzm

    Postup: Tímto vytvoříš modul, který pak stačí přesunout do složky /porteus/modules.

4. activate / deactivate

Když už máš .xzm soubor na disku, nemusíš restartovat počítač, aby se program objevil v menu.

    Aktivace: activate /cesta/k/programu.xzm (program se okamžitě objeví v menu a můžeš ho spustit).

    Deaktivace: deactivate /cesta/k/programu.xzm (program zmizí a uvolní místo v RAM).

Přesný postup instalace qBittorrentu v CLI:

Pokud jsi v terminálu jako root (su -> toor), udělej toto:

    Stáhni a vytvoř modul:
    Bash

    get-app qbittorrent

    (Sleduj obrazovku, stahování může chvíli trvat, nakonec se tě zeptá na vytvoření .xzm balíčku).

    Ulož ho natrvalo:
    Aby ti qBittorrent nezmizel po restartu, musíš ho přesunout z dočasné složky na tvůj disk (flashku):
    Bash

    mv /tmp/qbittorrent*.xzm /mnt/sda2/porteus/porteus/modules/

    (Cestu /mnt/sda2/... uprav podle toho, kde máš Porteus nainstalovaný).

    Aktivuj ho hned teď:
    Bash

    activate /mnt/sda2/porteus/porteus/modules/qbittorrent*.xzm

Shrnutí:

V Porteusu se v CLI používá hlavně get-app. Ten za tebe vyřeší všechnu těžkou práci – najde program, stáhne ho a vyrobí z něj modul .xzm. Pokud ten modul dáš do složky modules na svém disku, Porteus ho při každém startu automaticky "nainstaluje" (připojí).

#####################################################


Co teď uvidíš v Porteusu?

Po spuštění skriptu rc.local (příkazem su a pak /etc/rc.d/rc.local) se stane toto:

    Plocha: Uvidíš své ikony a soubory z Windows plochy přímo na ploše Linuxu.

    qBittorrent: Sám se spustí a díky prolinkování složek můžeš ukládat přímo do "Stažených souborů", které uvidíš i po návratu do Windows.

    Kodi & VLC: Budou mít okamžitý přístup k tvým filmům v /mnt/win_c/Users/$winUser/Videos.

    Double Commander: Spustíš ho z menu a uvidíš v něm dva panely – v jednom můžeš mít Linux a v druhém Windows disk pro snadné kopírování.

Důležitá poznámka k "Nerozbitnosti":

Všechny ty .xzm soubory, co jsi teď stáhl, Porteus při startu načte do RAM. Pokud by se stalo, že Porteus po přidání nějakého modulu nenaběhne, stačí se vrátit do Windows a ten poslední přidaný .xzm ze složky modules smazat. To je ta největší krása Porteusu – systém nelze "rozbít" trvale.
