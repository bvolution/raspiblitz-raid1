# RaspiBlitz im Doppelpack - RAID-1 für maximale Sicherheit beim Betrieb deiner Lightning⚡ Fullnode

In diesem Blog Beitrag bekommt ihr eine Schritt für Schritt Anleitung wie ihr die Sicherheit beim Betrieb einer RaspiBlitz weiter maximieren könnt indem ihr RAID-1 für euren RaspiBlitz aufsetzet. Dieser Blog ist von Plebs für Plebs geschrieben und basiert auf unseren Erfahrungen 🌝

## Zielbild RaspiBlitz mit RAID-1

1. [Proxmox VE Installieren](#1-proxmox-ve-installieren)  
Dazu werden wir auf einem Stromsparenden Mini PC einen Virtualisierungsserver (Proxmox) aufsetzen.
2. [ZFS RAID-1 in Proxmox aufsetzen](#2-zfs-raid-1-in-proxmox-aufsetzen)  
Einen RAID-1 (also absoluten Spiegel) für unsere SSD aufsetzen.  
Damit erzeugen wir Redundanz, was im Endeffekt bedeutet, selbst im Wort Case: das uns eine SSD abraucht, läuft unser RaspiBlitz ganz unbeindruckt weiter.
3. Erstellen einer Virtuellen Machine (erzeugt via Proxmox) auf der unser Raspiblitz läuft.
4. (Optional) Migrieren unserer bestehenden Node

Klingt technisch oder zu kompliziert 🫢?  
Ist es überhaupt nicht und wir nehmen dich mit auf die Reise und liefern hier ein Schritt für Schritt Tutorial.

### Warum? Oder was sind die Vorteile von RAID-1 💾?

Beim Ausfall einer SSD gibt es eine komplette Redundanz. Eure Channels bleiben also selbst beim  Totalausfall einer SSD offen. Bei den aktuellen hohen Transaktionsgebühren (TX, ~ 180 sats/VByte), spart ihr euch somit eine Menge Sats (₿), zudem ist es einfach ein sehr beruhigendes Gefühl, zu wissen dass die Platte mit euren Funds komplett gespiegelt und somit abgesichert ist.

**Exemplarischer aktueller Block**  🔎

Einen Channel zu Öffnen kostet bei den Momentanen Fees gerne mal zwischen 15.000 und 30.000 SATs. Ein stolzes Sümmchen was zum Zeitpunkt des schreibens in etwa ~5.72€ - 11.44€ entspricht.
  
![Alt text](image.png)

## Danger Zone ⚠️ - Gefahren & Risiken

Solltet ihr eine **neue Node** aufsetzen, ist der gesamte **Prozess völlig unbedenklich**. Entwas anders sieht das ganze jedoch aus wenn ihr eine bestehende Node migrieren wollt.  

Ok, Full Disclosure, der Prozess von einem Standard Setup (RaspiBlitz auf einem Raspberry 4 oder 5) auf ein Setup umzustellen, dass RAID-1 erlaubt ist nicht trivial und birgt Risiken. Im Sinne maximaler Transparenz und damit ihr die richtige Entscheidung für euchtreffen könnt sind hier einige Gefahren aufgelistet.

Das größte Risiko besteht darin, dass ihr ein Backup eurer LND Channels erstellt (am Ende des Prozesses fährt euer Raspiblitz runter) und es geht in diesem Moment noch ein HLTC (LN Transaktion durch). Nach dem Wiederherstellen eures Migrations File würde dass dazu führen, dass ihr mit dem Backup einen alten Channel Status Broadcastet und somit Gefahr für eine Justice / Penalty Transaktion lauft.

> ℹ️ **Info - Was ist das Risiko?**  
> Eine LN Zahlung geht beim erstellen des Backups durch und führt beim wiederherstellen des Backups zu einer Justice / Penalty Transaktion für diesen einen Channel kommt. Das würde den komplett Verlust aller Funds in diesem einen Channel bedeuten

**Additional Reads and Sources:**

- <https://docs.lightning.engineering/lightning-network-tools/lnd/migrating-lnd>
- <https://blog.bitmex.com/lightning-network-justice/>
- <https://voltage.cloud/blog/lightning-network-faq/penalty-transactions-on-lightning-network/>

### Vorbereitung / Mitigation

Ihr solltet unbedingt sicher gehen, dass ihr

1. Den Seed zu eurer Node habt
2. Ein aktuelles [SCB](https://docs.lightning.engineering/lightning-network-tools/lnd/recovery-planning-for-failure#static-channel-backups-scb) (Static Channel Backup) habt

Damit ihr im worst-case Szenario mit dem Desaster Recover ([Link](https://docs.lightning.engineering/lightning-network-tools/lnd/disaster-recovery)) eure Funds wieder herstellen könnt. Bedenkt aber dass ein disaster recovery mittels SCB auch mit dem Close aller eurer Channel einhergeht. Und entsprechend teuer werden kann. Daher werden wir für dieses Tutorial eine Migration in Angriff nehmen.

Es nicht notwendig für dieses Tutorial aber es kann nicht schaden. am besten die entsprechenden Abschnitte zur Migration

## Hardware Setup

Als _kleiner Bonustipp_: wenn ich Hardware bestelle, schaue ich als erstes bei [satsback.com](https://satsback.com/register/Ezv2VLwRk6Wd8X4Z). Satsback listet diverse Shops (z.B. Büroshop24 bei dem ich die SSDs bestellt habe), wenn ihr über den Link von Satsback auf einen der Shops geht bekommt Satsback eine Komission (auch Kickback genannt), wandelt diese in Satoshis um und zahlt sie direkt an euch aus. Ein wenig wie Payback für Satoshis.

Das schreibt [satsback.com](https://satsback.com/register/Ezv2VLwRk6Wd8X4Z) selbst auf ihrer Webseite

> "We work with online stores that pay us a commission whenever you shop with them. We convert that to bitcoin and share most of it with you. Because we also make a small profit every time you buy something, we can keep our platform free to use while respecting your privacy."

Hab bisher nur positive Erfahrungen gemacht. Full Disclosure der Link oben ist ein Referallink, muss keiner nutzen, kostet euch nichts und unterstützt den Autor dieses Tutorials 🌝

### Getestet für diesen Blog

Funktion | Setup 1 (bvolution) | Setup 2 (to be annouced)
---------|----------|---------
 Host System (Proxmox) | - HP EliteDesk 800 G3 mini 35W, Desktop-Mini, Core i5 7500T 2,7GHz, 16GB RAM, 512GB SSD <br> ([Link](https://www.computeroutlet24.de/pc-systeme/hp-elitedesk-800-g3-mini-35w-desktop-mini-core-i5-7500t-27ghz-16gb-ram-512gb-ssd-windows-10-pro.html?cache=1705252819)), **179€**) | -
 Speicher für Raid | - 2 x SanDisk 1TB SSD Plus ([Link](https://www.idealo.de/preisvergleich/OffersOfProduct/201902833_-ssd-plus-1tb-sdssda-1t00-g27-sandisk.html), **~60€**),<br> - angeschlossen über 2 x UGREEN SATA-III zu USB3.0 Adapter ([Link](https://www.amazon.de/dp/B06XWSDGP6?psc=1&ref=ppx_yo2ov_dt_b_product_details), 14€) | -
 USB Stick / SD Karte zum installieren von Proxmox | Beliebiger USB Stick oder SD Karte | -

<hr>

# Schritt-für-Schritt Tutorial

## 1) Proxmox VE Installieren

Die .iso Datei zur Installation von Proxmox kannst du [hier](https://www.proxmox.com/de/downloads) runterladen.
Wir wählen hier **Proxmox VE**

![Alt text](image-1.png)

> ℹ️ **Info:**  
> Was ist Promox VE? Proxmox Virtual Environment (Proxmox VE) ist eine Open-Source-Plattform für Virtualisierung, die auf dem Kernel-basierten Virtual Machine (KVM) Hypervisor und dem containerbasierten Virtualisierungssystem LXC basiert. Sie bietet eine integrierte Management-Oberfläche für die Bereitstellung und Verwaltung von virtuellen Maschinen und Containern auf einem einzigen Host.

**Wichtig:** Wenn ihr auf sicher gehen wollt, dass die Datei nicht manipuliert ist und identisch zu der auf der Homepage von Proxmox angegebenen datei ist, könnt ihr den SHA256 Hash der Datei abgleichen. Und gerade wenn es um eure hart verdienten Sats geht, lohnt es sich ggf. extra vorsichtig zu sein, oder 🤔?

Falls euch das nicht wichtig ist, könnt ihr getrost das nächste Unterkapitel überspringen

### 1.1) Installtionsdatei Verifizieren

#### Windows 🪟

- Dazu öffnest eine Konsole
  - Tipp: <kbd>Win</kbd> + <kbd>R</kbd> dort `cmd` eingeben <kbd>Enter</kbd>
- Wechsel in den Ordner in den Du die Datei runter geladen hast mittels `cd`
  - In der Regel wirst du die Datei vermutlich im Download Ordner runterladen.  
  `cd C:/Users/<benutzer>/Downloads`  
  Tausche hier < benutzer > gegen deinen Benutzernamen aus
- Mittels dem bei Windows standardmäßig gelieferten certutil bekommst du den Hash
  - ```certutil -hashfile "<dateiname>" SHA256```
  - In meinem Fall wäre diese der folgende Befehl (wenn du in der Zukunft eine neue Version runterlädst kann sich natürlich der Dateiname und natürlich auch der Zielhash ändern)
  - ```certutil -hashfile proxmox-ve_8.1-1.iso SHA256```
- Das Ergebnis (der Hash) gleichst du dann mit dem auf der Webseite angegeben Hash ab

```sh
SHA256-Hash von proxmox-ve_8.1-1.iso:
9018a17307ad50eb9bf32a805d0917d621499363ef87b0b477332ed9f9d7dcc1
CertUtil: -hashfile-Befehl wurde erfolgreich ausgeführt.
```

Hier seht ihr das der Hash aus der Konsole übereinstimmt mit dem von der Webseite

### 1.2) Vorbereiten der Installtion (Flash der .iso Datei)

Wenn Ihr bereits einen RaspiBlitz im Betrieb habt, sollte euch das folgende sehr bekannt vorkommen.
Die Schritte sind quasi identisch zum Setup für den Raspiblitz ([Write the SD Card image to the SD Card](https://github.com/raspiblitz/raspiblitz#write-the-sd-card-image-to-your-sd-card)).
Für alle die bisher noch nie ein anderes Betriebssystem installiert haben, ... ist hier die Definitiv nicht mit ChatGPT (😆) generierte, Begründung, warum
wir die nächsten Schritte begehen.

> ℹ️ **Info:**  
> Das Schreiben eines ISO-Abbilds auf einen USB-Stick ermöglicht es, ein bootfähiges Installationsmedium zu erstellen, das tragbar, schnell, wiederverwendbar und flexibel ist, was die Installation von Betriebssystemen auf verschiedenen Computern erleichtert.

Um ein solches "Bootfähiges Installationsmedium" zu erstellen, empfehle ich unter Windows Balena Etcher. Das Tool ist einfach super intuitiv und selbst erklärend.
Das ganze dauert dann eine kleine Weile

<p align=center>
<img src=image-2.png width=500/>
</p>

Den USB-Stick bzw. die SD-Karte stöpselt ihr jetzt einfach in euren Mini-PC und folgt den Installationsschritten ...

Die Installation ist ziemlich selbt erklärend 🌝. Wer es nochmal genauer nachlesen möchte, es gibt diverse Blogs zu diesem Thema <a href="https://decatec.de/home-server/proxmox-ve-installation-und-grundkonfiguration/"><sup>[1]</suo></a><a href="https://mwiza.medium.com/how-to-install-proxmox-ve-on-a-server-771c9f99933a"><sup>[2]</sup></a>, daher sparen wir uns für dieses Tutorial weitere Details.

### 1.3) Erste Schritte 👣 in Proxmox

Hier sind einige erste Schritte die ich nach dem Setup von Proxmox empfehlen kann

- Macht euch mit der GUI / Webinterface vertraut
  - Dazu gebt ihr im Browser eurer wahl folgendes ein `192.168.178.100:8006`
  - Bedenkt das durch die IP Adresse eures Proxmox Mini-PC zu ersetzen
  - Achtet auf den Port am Ende (8006)
- Loggt euch über SSH auf eurem Proxmox ein
  - Startet ein Terminal (s.o.)
  - `ssh root@192.168.178.100`
  - Auch hier müsst ihr natürlich die IP entsprechend austauschen

> **Info**  
> Ihr könnt die IP von eurem neuen Proxmox Mini-PC über euren Router herausfinden. Dazu loggt ihr euch über den Browser in euren Router ein und lasst euch das Lokale Netz anzeigen.

- Updated die System Packages und installiert was ihr braucht
  - Ich mag meine Bash Konsole gerne in Farbe
  - Außerdem entwickle ich gerne ich in Neovim
  - Und git braucht man eigentlich immer
  - Hier ist ein Micro Repository, mit den Dingen die ich gerne auf einem neuen Proxmox Sytem aufsetze: [`customize-proxmox`](https://github.com/bvolution/customize-proxmox)

## 2) ZFS RAID-1 in Proxmox aufsetzen

Wenn ihr schon wisst was ZFS und RAID-1 sind könnt ihr dieses unterkapitel überspringen und direkt zum Kapitel **2.2 Schritte zur Einrichtung** springen.  
  
Was ist jetzt ein ZFS RAID-1? Zunächst einmal ist ZFS ein Dateisystem (Zettabyte File System) und RAID-1 steht für "Redundant Array of Independent Disks (Redundanter Array unabhängiger Festplatten)" <a href="https://www.westerndigital.com/de-de/solutions/raid"><sup>[3]</sup></a> und bedeutet dass die Daten die normalerweise nur auf eine Platte geschrieben werden permanent gespiegelt und auf eine zweite Platte zusätzliche geschrieben werden.
Das ZFS (Zettabyte File System) ist für seine Robustheit und Datensicherheit bekannt, auch im Falle von Stromausfällen. Es wurde speziell entwickelt, um hohe Datenintegrität und Fehlertoleranz zu bieten.

> ℹ️ **Info:**  
> Ein ZFS RAID-1, auch als Spiegelung bekannt, beinhaltet das Kopieren von Daten auf zwei Festplatten (oder mehr) in Echtzeit. Alle Schreibvorgänge werden auf beide Platten dupliziert, was Redundanz und erhöhte Datensicherheit bietet, da auf die Daten zugegriffen werden kann, selbst wenn eine der Platten ausfällt.

### 2.1) Vorteile von ZFS

Was macht ZFS besonders geeignet (d.h. Sicher) für unser Szenario?
Ich finde Wikipedia fasst es ziemlich gut zusammen

> "ZFS nutzt Copy-On-Write und ein Journal (ZIL, ZFS Intent Log). ZFS kann so zu jeder Zeit auf ein konsistentes Dateisystem zurückgreifen. Sicherungen und Rücksicherungen von Blöcken sowie Dateisystemprüfungen sind so bei Abbrüchen wie einem Stromausfall nicht nötig. Inkonsistenzen in Metadaten und Daten werden bei jedem Lesevorgang automatisch erkannt und bei redundanter Information soweit möglich automatisch korrigiert. Die Leistung von solchen Dateisystemen nimmt allerdings ab ca. 80 % Belegung spürbar ab, wie bei allen anderen Dateisystemen auch."
(Quelle: [Wiki](https://de.wikipedia.org/wiki/ZFS_(Dateisystem)))

Weitere Vorteile finden sich z.B. auch im Proxmox [Wiki](https://pve.proxmox.com/wiki/ZFS_on_Linux)

> - Easy configuration and management with Proxmox VE GUI and CLI.
> - Reliable
> - Protection against data corruption
> - Data compression on file system level
> - Snapshots
> - Copy-on-write clone
> - Various raid levels: RAID0, RAID1, RAID10, RAIDZ-1, RAIDZ-2, RAIDZ-3, dRAID, dRAID2, dRAID3
> - Can use SSD for cache
> - Self healing
> - Continuous integrity checking
> - Designed for high storage capacities
> - Asynchronous replication over network
> - Open Source
> - Encryption

Die Wikipedia Seite zu ZFS in Proxmox (s.o.) ist allgemein sehr informativ und empfehlenswert.

### 2.2) Schritte zum Einrichten

1. Versichert euch mit list block (`lsblk`) das eure Platten auffindbar sind. In meinem Fall sind sie zu finden unter `/dev/sdb` und `/dev/sdc`

<p align=center>
<img src=image-3.png width=400/>
</p>

2. Für die beste Performance empfiehlt es sich zunächst einen GPT (= GUID Partitions Tabelle<a href="https://de.wikipedia.org/wiki/GUID_Partition_Table"><sup>[4]</sup></a>, ausnahmsweise mal nicht die KI 😆) auf die beiden neuen Platten mittels gdisk<a href="https://wiki.ubuntuusers.de/gdisk/#Aufbau-einer-GPT"><sup>[5]</sup></a> zu schreiben  zu schreiben
   1. `gdisk /dev/sdb`
      1. Wichtig für beide Platten und, sdb oder sdc entsprechend durch eure device Buchstaben ersetzen.
   2. Dann eingeben `"gpt"` -> <kbd>O</kbd> -> <kbd>Y</kbd>
   3. Dann noch schreiben auf die Platte mittels <kbd>W</kbd> -> <kbd>Y</kbd>
   4. Rinse and Repeat (Also erneut für die zweite Platte)

2. **Als Tipp**: Holt euch die UUID (Universally Unique Identifier) eurer Platten und notiert diese gemeinsam mit dem /dev/sdX (z.B. `/dev/sdb`) auf einem Aufkleber direkt auf eurer Platte. Die UUID bekommt ihr raus über `blkid /dev/sdb`
   1. Um herauszufinden welche Platte welche ist, könnt ihr diese einzeln abstecken (USB ziehen) und erneut mit `blkid` schauen welche der beiden (`/dev/sdb` oder `/dev/sdc` + welche UUID noch angezeigt wird)
   2. Hier im Beispiel blkid bevor ich eine Platte ziehe
   <p align=center><img src=image-4.png width=450 /></p>
   3. Hier nachdem ich eine gezogen habe um den entsprechenden  Aufkleber anzubringen
   <p align=center><img src=image-5.png width=450 /></p>
   Entsprechend weiß ich, die noch angeschlossene Platte ist `/dev/sdc` mit UUID `cab8 ... 80d` und kann den entspechenden Aufkleber anbringen.
3. Nun geht es ans eingemachte. Wir richten den den eigentlichen ZFS-Pool (RAID-1) ein.
   1. Man kann dies bequem über die Graphische Oberfläsche (GUI) tun, und wird z.B. hier<sup><a href="https://technium.ch/proxmox-zfs-mirror-zfs-raid-1-erstellen-tutorial/">[6]</sup></a> erklärt.
   2. Ihr loggt euch dazu in eurem webui ein (also über die IP eures proxmox mit port 8006). In meinem fall gebe ich `https://192.168.178.100:8006/` im browser meines vertrauens ein.
   3. Links in der Navigation wählt man unter `Datacenter/Nodes/pve`, drückt auf pve (proxmox virtual environment).
   4. In der mittleren leiste geht ihr auf `Disks/ZFS`
   5. Anschließend auf `Create: ZFS`
   ![Alt text](image-6.png)
   6. Hier macht ihr folgende Einstellungen
      1. Wählt einen Namen (traidtionell in online blogs etc häufig als `tank`)
      2. RAID Level: hier wählt ihr Mirror für RAID1
      3. Wählt in der Liste eure SSDs aus
      4. Drückt auf Create
   ![Alt text](image-7.png)
   7. Mit doppelclick auf den neuen ZFS Pool den ihr soeben eingerichtet habt, könnt ihr euch dann auch den status der Platten noch einmal anzeigen lassen
 ![Alt text](image-8.png)

Das war es im Grunde genommen schon. Im nächsten Schritt installieren wir auf einer VM zunächst eine neue Bitcoin und LN Fullnode über den Raspiblitz.

## 3) VM mit RaspiBlitz installieren

Eine sehr gute Anleitung wie man eine Raspiblitz VM in Proxmox aufsetzt bietet bereits cerctrova (einer der Einundzwanzig Jungs) in seinem Blog ([hier](https://cercatrova.blog/raspiblitz-auf-proxmox-installieren/)). Daher gibt es hier nur eine Zusammenfassung der wichtigsten Schritte

1. Debian ISO herunterladen
   1. 12 Bookworm in meinem Fall amd64. Die richtige Architektur könnt ihr mit `dpkg --print-architecture` in der Proxmox Shell (z.B. via SSH login herausfinden)
   2. Empfehle die netinstall (also debian ohne GUI)
   3. <https://www.debian.org/distrib/netinst>
   4. Tipp auch hier könnt ihr wieder die installation mittels SHA512 verifizieren:
      1. `certutil -hashfile debian-12.4.0-amd64-netinst.iso SHA512`
      2. Abgleichen mit <https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS>
2. Debian ISO auf dem Proxmox hochladen
3. VM erstellen
   1. General: Name vergeben  
   2. OS: Das zuvor hochgeladen ISO auswählen
   3. System: QEMU-Agent aktivieren
   4. Disks: ich hab 50 GiB gewählt, ihr könnt es aber auch bei den für den RaspiBlitz üblichen 32 GiB belassen.
   5. CPU: Wähle hier alle verfügbaren kerne aus (bei mir 4). `lscpu`
   6. Starten der VM
   7. In die Konsole der VM

   <!-- 2. Ich bevorzuge es auf der Konsole zu arbeiten für dieses Tutorial. Dazu verwenden wir `zpool create`.
   8. Im Simme don't trust verify könnt ihr wenn ihr euch noch tiefer einlesen wollt diesen Guide oder schlicht das manual zu zpool create mittels `man zpool create` durchlesen

  
  ```sh
  zpool create -o ashift=12 <mirrorname> mirror /dev/disk/by-id/UUID-angeben /dev/disk/by-id/UUID-angeben
  ``` -->

## Nächste Sicherheitsausbaustufen

Wir nähern uns hier mit dem RAID-1 Betrieb einer extrem hohen Ausfallsicherheit. Ein letztes Risiko bleibt der Stromausfall / Blackout. Im worst case, geht hier dein komplettes Setup in die Knie, mitten im Schreibprozess auf den RAID-1 (die beiden SSDs). Durch die Art und Weise wie RAID-1 und ZFS funktioniert ist die gefahr von data corruption ist etwas höher bei ZFS

### USV / UPS als Lösung

Um auch im Falle des gefüchteten Blackouts best möglich geschützt zu sein (und im übrigen auch gegen gelgentliche vorkommende Schwankungen im Netz), empfiehlt es sich als eine weitere Sicherhehits-Ausbaustufe über eine USV (**U**nunterbrechbare **S**trom**v**ersorgung) nachzudenken. Mehr dazu in unserem nächsten Blog Beitrag.
