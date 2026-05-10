# Knuts PV Warmwater Heater

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Blueprint-blue?logo=home-assistant)](https://www.home-assistant.io/)
[![HA Version](https://img.shields.io/badge/HA%20Version-2025.1%2B-green)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

### PV-Überschuss Pufferspeicher Heizstab Steuerung

Ein Home Assistant Blueprint zur intelligenten Steuerung von ein oder zwei Heizstäben
im Pufferspeicher oder Warmwasserbereiter — entwickelt von **Knutzen19** mit Unterstützung
von **Claude (Anthropic)**.

Wer eine Primärheizung (Gas, Öl, Pellets) betreibt kennt das Problem: Die Anlage springt
an sobald der Pufferspeicher auskühlt — auch dann wenn tagsüber genug PV-Strom verfügbar
wäre um ihn zu halten. Dieser Blueprint löst genau das. Er steuert Heizstäbe im
Pufferspeicher so, dass die PV-Anlage die Wärmeversorgung vollständig übernimmt.
Weniger Starts bedeuten weniger Brennstoffverbrauch, weniger Verschleiß und eine längere
Lebensdauer der Primäranlage.

Drei aufeinander aufbauende Modi sorgen dafür dass der verfügbare PV-Strom optimal
eingesetzt wird: Der Puffer wird vor einem kritischen Auskühlen geschützt, anschließend
zuverlässig auf Tagestemperatur gebracht — und überschüssiger Strom der sonst ins Netz
ginge heizt den Puffer weiter auf, ohne dabei anderen Verbrauchern Strom wegzunehmen.

---

### Features

- **Drei Betriebsmodi** — BOOST, TAGESBEDARF und ÜBERLADEN greifen automatisch
  je nach Puffertemperatur ineinander (siehe Regellogik)
- **Bis zu zwei Heizstäbe** — frei konfigurierbare Leistung, intelligente Priorisierung
  (größerer zuerst ein, größerer zuerst aus)
- **Batterie-Mindestladestand** — Heizstäbe schalten erst ein wenn die Batterie
  ausreichend geladen ist
- **Temperatur-Abschaltung** — maximale Puffertemperatur schützt vor Überhitzung,
  sofortige Abschaltung
- **Zirkulationspumpe** *(optional)* — läuft automatisch mit dem Heizstab mit und
  verteilt die Wärme im Puffer; konfigurierbarer Nachlauf nach Heizstab-Abschaltung
- **Verzögertes Abschalten** — konfigurierbare Wartezeit verhindert Abschalten bei
  kurzen PV-Schwankungen (Wolkendurchzug)
- **Hysterese** — verhindert ständiges Ein-/Ausschalten an der Schaltschwelle
- **Kein delay-Block** — Verzögerungslogik über input_boolean Helfer, damit
  mode: single nie blockiert wird
- **Ausführliches Logbook** — jeder Schaltvorgang protokolliert Modus, Überschuss/Export,
  Batterie- und Puffertemperatur; einsehbar unter
  *Einstellungen → Protokoll* oder direkt in der Automations-Trace-Ansicht

---

### Voraussetzungen

- Home Assistant 2025.1 oder neuer
- PV-Anlage mit Wechselrichter/Energiemanagementsystem das folgende Sensoren bereitstellt:
  - Batterieladestand (%)
  - Export-Leistung (Netzeinspeisung)
  - Import-Leistung (Netzbezug)
  - Hausverbrauch
  - PV-Erzeugung (DC oder AC)
  - Temperatursensor am Pufferspeicher
- Schaltbare Heizstäbe — **die Schaltung muss für die Nennleistung der Heizstäbe ausgelegt sein**
- Ein input_boolean Helfer (zwei bei aktivierter Zirkulationspumpe)
- *(Optional)* Schaltbare Zirkulationspumpe

---

### Kompatible Wechselrichter / Energiemanagementsysteme

Der Blueprint funktioniert mit jedem System das die benötigten Sensoren in Home
Assistant bereitstellt. Bekannte Entitätsnamen:

| Wechselrichter | Batterie | Export | Import | Verbrauch | PV |
|---|---|---|---|---|---|
| SunGrow | `battery level` | `Export power` | `import power` | `load power` | `total dc power` |
| Fronius | `state_of_charge` | `grid_feed_in_power` | `grid_consumption_power` | `consumption_power` | `pv_power` |
| SMA | `soc` | `power_to_grid` | `power_from_grid` | `house_consumption` | `pv_generation` |
| Huawei | `battery_state_of_capacity` | `power_grid_exported` | `power_grid_imported` | `power_consumption` | `input_power` |
| E3/DC | `battery_soc` | `grid_export_power` | `grid_import_power` | `home_power` | `solar_power` |

Die genauen Entitätsnamen können je nach Integration und Konfiguration abweichen.
Im Zweifel unter *Einstellungen → Geräte & Dienste → Entitäten* suchen.

---

### Einrichtung

#### Schritt 1 — Helfer anlegen

Unter *Einstellungen → Geräte & Dienste → Helfer → Erstellen → Schalter*
einen (bzw. zwei mit Pumpe) Umschalter (input_boolean) anlegen:

| Name | Empfohlene Entity ID | Zweck |
|---|---|---|
| PV Heizer Abschaltung Ausstehend | `input_boolean.pv_heizer_abschaltung_ausstehend` | Abschalt-Countdown |
| PV Heizer Pumpe Nachlauf | `input_boolean.pv_heizer_pumpe_nachlauf` | Pumpen-Nachlauf *(nur mit Pumpe)* |

#### Schritt 2 — Blueprint importieren

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Knutzen19/knuts-pv-warmwater-heater/blob/main/blueprints/automation/pv_heizstab_steuerung.yaml)

Oder manuell: *Einstellungen → Automationen → Blueprints → Blueprint importieren*

```
https://github.com/Knutzen19/knuts-pv-warmwater-heater/blob/main/blueprints/automation/pv_heizstab_steuerung.yaml
```

#### Schritt 3 — Automation konfigurieren

---

### Konfigurationsparameter

| Parameter | Beschreibung | Standard |
|---|---|---|
| Batterie-Ladestand | Sensor Batterieladestand in % | — |
| Export Leistung | Sensor Netzeinspeisung (positiv = Export) | — |
| Import Leistung | Sensor Netzbezug (positiv = Import) | — |
| Gesamtverbrauch | Sensor Hausverbrauch | — |
| PV-Ertrag | Sensor PV-Erzeugung | — |
| Heizstab 1 Schalter | Schalter Heizstab 1 (leistungsgerecht ausgelegt) | — |
| Heizstab 1 Leistung | Nennleistung Heizstab 1 | 2000 W |
| Zweiten Heizstab verwenden | Zweiten Heizstab aktivieren | false |
| Heizstab 2 Schalter | Schalter Heizstab 2 (leistungsgerecht ausgelegt) | — |
| Heizstab 2 Leistung | Nennleistung Heizstab 2 | 3000 W |
| Zirkulationspumpe verwenden | Pumpensteuerung aktivieren | false |
| Zirkulationspumpe Schalter | Schalter Zirkulationspumpe | — |
| Nachlaufzeit Pumpe | Laufzeit nach Heizstab-Abschaltung | 30 min |
| Nachlauf-Hilfsschalter | input_boolean für Pumpen-Nachlauf | — |
| Abschalt-Hilfsschalter | input_boolean für Abschalt-Countdown | — |
| Minimaler Batterieladestand | Unter diesem Wert: Sofortabschaltung | 90 % |
| Abschaltverzögerung | Wartezeit bei Energiemangel | 5 min |
| Hysterese | Puffer unter Einschaltschwelle für Abschaltung | 300 W |
| Temperatur-Sensor (Abschalt) | Sensor für maximale Puffertemperatur | — |
| Abschalt-Temperatur | Maximale Puffertemperatur | 65 °C |
| Boost-Temperatur | Unter diesem Wert: Batteriegrenze ignorieren | 33 °C |
| Zieltemperatur-Sensor | Sensor für den Moduswechsel TAGESBEDARF → ÜBERLADEN | — |
| Zieltemperatur | Unter diesem Wert TAGESBEDARF, darüber ÜBERLADEN | 50 °C |

---

### Regellogik

Der Blueprint arbeitet mit drei Betriebsmodi die automatisch je nach Situation
ineinandergreifen:

#### Modus 1 — BOOST (Primärheizung-Schutz)
*Aktiv wenn: Puffertemperatur unter Boost-Schwelle (z.B. 33 °C)*

Der Puffer ist kritisch kalt — die Primärheizung würde sonst anspringen.
In diesem Modus wird die Batterie-Mindestgrenze ignoriert und der Heizstab
schaltet ein sobald PV-Strom verfügbar ist, unabhängig vom Akkuladestand.
Eine Hysterese von +2 °C verhindert ständiges Ein-/Ausschalten an der Schwelle.

#### Modus 2 — TAGESBEDARF (Puffer auf Zieltemperatur bringen)
*Aktiv wenn: Puffertemperatur zwischen Boost-Schwelle und Zieltemperatur (z.B. 33–50 °C)*

Der Puffer soll auf seine Zieltemperatur gebracht werden — jene Wärme die bis
zum nächsten Tag ausreicht. Regelgröße ist der berechnete PV-Überschuss
(PV-Leistung minus Hausverbrauch). Die Batterie hat dabei Vorrang: Sie lädt
zuerst bis zur konfigurierten Mindestgrenze und konkurriert nicht mit dem Heizstab.
Der Heizstab bekommt nur was nach dem Speichern übrig bleibt — und das zuverlässig.

#### Modus 3 — ÜBERLADEN (Überladen / Restüberschuss sichern)
*Aktiv wenn: Puffertemperatur über Zieltemperatur (z.B. über 50 °C)*

Die Tagestemperatur ist erreicht. Ab jetzt wird nur noch echter Netzüberschuss
genutzt — Strom der nach dem Laden der Batterie und nach allen anderen
Verbrauchern übrig bleibt und sonst ins Netz ginge. Regelgröße ist der
reale Export-Messwert am Smartmeter. Andere PV-gesteuerte Geräte wie
Wallbox oder Geschirrspüler haben dadurch automatisch Vorrang. Der Heizstab
ist in diesem Modus bewusst die letzte Instanz und heizt den Puffer nur
soweit auf wie es der echte Überschuss erlaubt.

Praktischer Nebeneffekt: Ein übervoll aufgeheizter Puffer sichert einen
bewölkten Folgetag ab — oder man gönnt sich einfach eine günstige heiße
Badewanne. Eine Integration von PV-Ertragsprognosen zur vorausschauenden
Steuerung ist für eine zukünftige Version geplant.

```
Ablauf jede Minute:

  Puffer < Boost-Schwelle  →  BOOST:       Batteriegrenze ignorieren
  Puffer < Zieltemperatur  →  TAGESBEDARF: Regelgröße: PV - Last
  Puffer ≥ Zieltemperatur  →  ÜBERLADEN:      Regelgröße: Export-Sensor

  In allen Modi gilt die Prioritätskette:
  P0  Batterie unter Limit ODER Temp über Maximum  →  SOFORT AUS
  P1  Überschuss ≥ H1 + H2                         →  BEIDE EIN
  P2  Überschuss ≥ größerer Heizstab               →  GRÖSSEREN EIN
  P3  Überschuss ≥ kleinerer Heizstab              →  KLEINEREN EIN
  P4  Zu wenig Überschuss                          →  Countdown → ABSCHALTEN
  P5  Situation erholt                             →  Countdown abbrechen
  P6  Zirkulationspumpe: EIN mit Heizstab, AUS nach Nachlaufzeit
```

---

### Logbook

Jeder Schaltvorgang wird im Home Assistant Logbook protokolliert und enthält
den aktiven Modus, den aktuellen Überschuss bzw. Export, den Batterieladestand
sowie die Puffer- und Zieltemperatur. Das Logbook ist erreichbar unter:

*Einstellungen → Protokoll* — dort nach "PV-Heizsteuerung" filtern.

Alternativ in der Automations-Trace-Ansicht: *Einstellungen → Automationen →
Automation auswählen → Traces* zeigt jeden Durchlauf mit allen Variablenwerten
und dem Ergebnis jeder Bedingung.

---

### Mehrere PV-Automationen kombinieren

Im ÜBERLADEN-Modus sind die Heizstäbe automatisch die letzte Instanz. Jedes andere
Gerät das einschaltet erhöht den Hausverbrauch, reduziert den Export — und der
Blueprint reagiert beim nächsten Minutentakt automatisch.

Empfehlung für mehrere PV-Automationen:

```yaml
# Andere PV-Automationen (höhere Priorität) — auf Sekunde :30:
trigger:
  - platform: time_pattern
    seconds: "30"

# Dieser Blueprint (niedrigste Priorität) — auf Sekunde :00:
trigger:
  - platform: time_pattern
    minutes: "*"
```

So hat der Export-Sensor 30 Sekunden Zeit sich nach dem Einschalten anderer
Geräte einzupendeln bevor die Heizsteuerung entscheidet.

---

### Changelog

| Version | Datum | Änderungen |
|---|---|---|
| v5 | April 2026 | Drei-Modus Regelung (BOOST + TAGESBEDARF + ÜBERLADEN), Zirkulationspumpe mit Nachlauf |
| v4 | März 2026 | Export-basierte Regelung, dynamische Heizstab-Priorisierung *(experimentell)* |
| v3 | Februar 2026 | Optionaler 2. Heizstab, konfig. Leistung, BOOST-Modus, Temperatur-Abschaltung |
| v2 | Februar 2026 | Kein delay-Block mehr, input_boolean Helfer für Countdown |
| v1 | Januar 2026 | Initiale Version |

---

### Beitragen

Pull Requests und Issues sind willkommen. Besonders hilfreich:
- Erfahrungsberichte und Entitätsnamen anderer Wechselrichter
- Verbesserungsvorschläge zur Regellogik

---

*Entwickelt von [Knutzen19](https://github.com/Knutzen19) mit Unterstützung von [Claude](https://claude.ai) (Anthropic).*
