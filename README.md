# RaspiBlitz im Doppelpack - RAID-1 für maximale Sicherheit beim Betrieb deiner Lightning⚡ Fullnode

In diesem Blog Beitrag berichten wir, all das was wir gelernt haben und geben euch eine Schritt für Schritt Anleitung wie ihr die Sicherheit beim Betrieb einer RaspiBlitz weiter maximieren und einen RAID-1 aufsetzen könnt.

## Zielbild

1. Dazu werden wir auf einem Stromsparenden Mini PC einen Virtualisierungsserver (Proxmox) aufsetzen.
2. Einen RAID-1 (also absoluten Spiegel) für unsere SSD aufsetzen.  
Damit erzeugen wir Redundanz, was im Endeffekt bedeutet, selbst im Wort Case: das uns eine SSD abraucht, läuft unser RaspiBlitz ganz unbeindruckt weiter.
3. Erstellen einer Virtuellen Machine (erzeugt via Proxmox) auf der unser Raspiblitz läuft.
4. (Optional:) Migrieren unserer bestehenden Node

Klingt technisch und zu kompliziert? Ist es überhaupt nicht und wir nehmen dich mit auf die Reise und liefern hier ein Schritt für Schritt tutorial.

### Warum? Oder die Vorteile von RAID-1 💾

Beim Ausfall einer SSD gibt es eine komplette Redundanz. Eure Channels bleiben also selbst beim total Ausfall einer SSD offen. Bei den aktuellen hohen Transaktionsgebühren (TX, ~ 180 sats/VByte), spart ihr euch somit eine Menge Sats (₿), zudem ist es einfach ein sehr beruhigendes Gefühl, zu wissen dass die Platte mit euren Funds komplett gespiegelt und somit abgesichert ist.

**Exemplarischer aktueller Block**  🔎
  
![Alt text](image.png)

## Danger Zone ⚠️ - Gefahren & Risiken

Solltet ihr eine **neue Node** aufsetzen, ist der gesamte **Prozess völlig unbedenklich**. Entwas anders sieht das ganze jedoch aus wenn ihr eine bestehende Node migrieren wollt.  

Ok, Full Disclosure, der Prozess von einem Standard Setup (RaspiBlitz auf einem Raspberry 4 oder 5) auf ein Setup umzustellen, dass RAID-1 erlaubt ist nicht trivial und birgt Risiken. Im Sinne maximaler Transparenz und damit ihr die richtige Entscheidung für euchtreffen könnt sind hier einige Gefahren aufgelistet.

## Hardware Setup

### Getestet für diesen Blog

Funktion | Setup 1 (bvolution) | Setup 2 (to be annouced)
---------|----------|---------
 Host System (Proxmox) | - HP EliteDesk 800 G3 mini 35W, Desktop-Mini, Core i5 7500T 2,7GHz, 16GB RAM, 512GB SSD <br> ([Link](https://www.computeroutlet24.de/pc-systeme/hp-elitedesk-800-g3-mini-35w-desktop-mini-core-i5-7500t-27ghz-16gb-ram-512gb-ssd-windows-10-pro.html?cache=1705252819)), **179€**) | -
 Speicher für Raid | - 2 x SanDisk 1TB SSD Plus ([Link](https://www.idealo.de/preisvergleich/OffersOfProduct/201902833_-ssd-plus-1tb-sdssda-1t00-g27-sandisk.html), **~60€**),<br> - angeschlossen über 2 x UGREEN SATA-III zu USB3.0 Adapter ([Link](https://www.amazon.de/dp/B06XWSDGP6?psc=1&ref=ppx_yo2ov_dt_b_product_details), 14€) | -
 USB Stick / SD Karte zum installieren von Proxmox | Beliebiger USB Stick oder SD Karte -.. | -

<hr>

# Schritte

1. Proxmox (Virtualisierungs Host System)
2. ZFS RAID-1 Setup: SSD wird komplett gespiegelt
3. Virtuelle Machine (RaspiBlitz)
4. (Migration einer bestehenden Node)

Ok auf gehts

## 1) Proxmox VE Installieren

Die .iso Datei zur Installation von Proxmox kannst du [hier](https://www.proxmox.com/de/downloads) runterladen.
Wir wählen hier **Proxmox VE**

![Alt text](image-1.png)

> ℹ️ **Info:**  
> Was ist Promox VE? Proxmox Virtual Environment (Proxmox VE) ist eine Open-Source-Plattform für Virtualisierung, die auf dem Kernel-basierten Virtual Machine (KVM) Hypervisor und dem containerbasierten Virtualisierungssystem LXC basiert. Sie bietet eine integrierte Management-Oberfläche für die Bereitstellung und Verwaltung von virtuellen Maschinen und Containern auf einem einzigen Host.

**Wichtig:** Wenn ihr auf sicher gehen wollt, dass die Datei nicht manipuliert ist und identisch zu der auf der Homepage von Proxmox angegebenen datei ist, könnt ihr den SHA256 Hash der Datei abgleichen. Und gerade wenn es um eure hart verdienten Sats geht, lohnt es sich ggf. extra vorsichtig zu sein, oder 🤔?

Falls euch das nicht wichtig ist, könnt ihr getrost das nächste Unterkapitel überspringen

### Installtionsdatei Verifizieren

#### Windows 🪟

- Dazu öffnest eine Konsole
  - Tipp: <kbd>Win</kbd> + <kbd>R</kbd> dort `cmd` eingeben <kbd>Enter</kbd>
- Wechsel in den Ordner in den Du die Datei runter geladen hast mittels `cd`
  - In der Regel wirst du die Datei vermutlich im Download Ordner runterladen.  
  `cd C:/Users/<benutzer>/Downloads`  
  Tausche hier < benutzer > gegen deinen Benutzernamen aus
- Mittels dem bei Windows standardmäßig gelieferten certutil bekommst du den Hash
  - ```sh certutil -hashfile "<dateiname>" SHA256```
  - In meinem Fall wäre diese der folgende Befehl (wenn du in der Zukunft eine neue Version runterlädst kann sich natürlich der Dateiname und natürlich auch der Zielhash ändern)
  - ```sh certutil -hashfile proxmox-ve_8.1-1.iso SHA256```
- Das Ergebnis (der Hash) gleichst du dann mit dem auf der Webseite angegeben Hash ab

```sh
SHA256-Hash von proxmox-ve_8.1-1.iso:
9018a17307ad50eb9bf32a805d0917d621499363ef87b0b477332ed9f9d7dcc1
CertUtil: -hashfile-Befehl wurde erfolgreich ausgeführt.
```

Hier seht ihr das der Hash aus der Konsole übereinstimmt mit dem von der Webseite

### Flash der .iso Datei

Wenn du bereits einen RaspiBlitz im Betrieb hast

## 2.) ZFS RAID-1 in Proxmox aufsetzen

Ipsum Dorusm

> ℹ️ **Info:**  
> Ein ZFS RAID-1, auch als Spiegelung bekannt, beinhaltet das Kopieren von Daten auf zwei Festplatten (oder mehr) in Echtzeit. Alle Schreibvorgänge werden auf beide Platten dupliziert, was Redundanz und erhöhte Datensicherheit bietet, da auf die Daten zugegriffen werden kann, selbst wenn eine der Platten ausfällt.

für die beste performance aber noch die ssds mit gdisk auf gpt formatieren

- dann o (gpt schreiben)
- dann w
- macht die änderungn wirksam

```sh
zpool create -o ashift=12 <mirrorname> mirror /dev/disk/by-id/UUID-angeben /dev/disk/by-id/UUID-angeben
```

## Nächste Sicherheitsausbaustufen

Wir nähern uns hier mit dem RAID-1 Betrieb einer extrem hohen Ausfallsicherheit. Ein letztes Risiko bleibt der Stromausfall / Blackout. Im worst case, geht hier dein komplettes Setup in die Knie, mitten im Schreibprozess auf den RAID-1 (die beiden SSDs). Durch die Art und Weise wie RAID-1 und ZFS funktioniert ist es nicht ausgeschlossen, das dabei nicht beide SSDs gleichzeitig ausfallen.  

### USV / UPS als Lösung

Um auch im Falle des gefüchteten Blackouts best möglich geschützt zu sein (und im übrigen auch gegen gelgentliche vorkommende Schwankungen im Netz), empfiehlt es sich als eine weitere Sicherhehits-Ausbaustufe über eine USV (**U**nunterbrechbare **S**trom**v**ersorgung) nachzudenken. Mehr dazu in unserem nächsten Blog Beitrag.
