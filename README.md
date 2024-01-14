# RaspiBlitz im Doppelpack

# RAID-1 für maximale Sicherheit beim Betrieb deiner Lightning⚡ Fullnode

## Zielbild

- Proxmox (Virtualisierungs Host System)
- ZFS RAID-1 Setup: SSD wird komplett gespiegelt

### Vorteile RAID-1

Beim Ausfall einer SSD gibt es eine komplette Redundanz.  
Eure Channels bleiben also selbst beim total Ausfall einer SSD offen.  
Bei den aktuellen hohen Transaktionsgebühren (TX, ~ 180 sats/VByte),
spart ihr euch somit eine Menge Sats (₿), zudem ist es einfach ein  
sehr beruhigendes Gefühl, zu wissen dass die Platte mit euren Funds  
komplett gespiegelt und somit abgesichert ist.

**Exemplarischer aktueller Block**  
![Alt text](image.png)

## Danger Zone ⚠️

## Potentielle Gefahren und Trouble Shoot

Ok, Full Disclosure, der Prozess von einem Standard Setup (RaspiBlitz auf einem Raspberry 4 oder 5)  
auf ein Setup umzustellen, dass RAID-1 erlaubt ist nicht trivial und birgt Risiken.  
Im Sinne maximaler Transparenz und damit ihr die richtige Entscheidung für euch
treffen könnt sind hier einige Gefahren aufgelistet

## Hardware Setup

### Getestet für diesen Blog

Funktion | Setup 1 (bvolution) | Setup 2 (to be annouced)
---------|----------|---------
 Host System (Proxmox) | - HP EliteDesk 800 G3 mini 35W, Desktop-Mini, Core i5 7500T 2,7GHz, 16GB RAM, 512GB SSD <br> ([Link](https://www.computeroutlet24.de/pc-systeme/hp-elitedesk-800-g3-mini-35w-desktop-mini-core-i5-7500t-27ghz-16gb-ram-512gb-ssd-windows-10-pro.html?cache=1705252819)), **179€**) | -
 Speicher für Raid | - 2 x SanDisk 1TB SSD Plus ([Link](https://www.idealo.de/preisvergleich/OffersOfProduct/201902833_-ssd-plus-1tb-sdssda-1t00-g27-sandisk.html), **~60€**),<br> - angeschlossen über 2 x UGREEN SATA-III zu USB3.0 Adapter ([Link](https://www.amazon.de/dp/B06XWSDGP6?psc=1&ref=ppx_yo2ov_dt_b_product_details), 14€) | -

## Nächste Sicherheitsausbaustufen

Wir nähern uns hier einer extrem hohen Aufallssicherheitsgüte.  
Ein letztes Risiko bleibt der Stromausfall / Blackout. Im worst case,  
geht hier dein komplettes Setup in die Knie, mitten im Schreibprozess auf  
den RAID-1 (die beiden SSDs). Durch die Art und Weise wie RAID-1 und ZFS funktioniert  
ist es nicht ausgeschlossen, das dabei nicht beide SSDs gleichzeitig ausfallen.  

### USV / UPS als Lösung

Um auch im Falle des gefüchteten Blackouts best möglich geschützt zu sein
(und im übrigen auch gegen gelgentliche vorkommende Schwankungen im Netz),  
empfiehlt es sich als eine weitere Sicherhehits-Ausbaustufe über eine  
USV (**U**nunterbrechbare **S**trom**v**ersorgung) nachzudenken.
Mehr dazu in unserem nächsten Blog Beitrag.
