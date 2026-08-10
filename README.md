# ESP32-RC-Sound

ESP32-basiertes RC-Soundmodul mit I2S-Audioausgabe, SD-Karten-Wiedergabe, WLAN-Weboberfläche, CRSF-Parametersystem und optionaler LiPo-Telemetrie über FrSky S.Port sowie GPS-Geschwindigkeitsmessung.

**Aktuelle Version:** v2.22 · **Autor:** PiperPilot (ab v0.70) / Ziege-One – Der RC-Modellbauer (v0.45)

---

## Inhalt

- [Features](#features)
- [Hardware-Versionen](#hardware-versionen)
- [Pin-Belegung](#pin-belegung)
- [Hardware-Voraussetzungen](#hardware-voraussetzungen)
- [Bibliotheken](#bibliotheken)
- [SD-Karten-Dateien](#sd-karten-dateien)
- [Installation & Flashen](#installation--flashen)
- [Konfiguration](#konfiguration)
- [Quellentypen](#quellentypen)
- [Multi-Device-Betrieb](#multi-device-betrieb)
- [Changelog](#changelog)
- [Bekannt behobene Fehler](#bekannt-behobene-fehler)

---

## Features

- Motorklangsimulation (Start, Loop, Abschalten) mit drehzahlabhängiger Abspielgeschwindigkeit
- 8 frei konfigurierbare Zusatzsounds (WAV-Dateien von SD-Karte) in drei Modi: **Normal**, **Loop**, **Tippbetrieb**
- Unterstützung für SBUS (FrSky, FlySky, ELRS SBUS, Hott) und CRSF/ELRS
- Einkanal-Multiplexing (MultiSwitch-Protokoll, WM0–WM3)
- Ebenenumschaltung (bis zu 7 Ebenen × 3 Gruppen)
- Alle fünf Hardware-Versionen (V1–V5) in einem Firmware-Image
- CRSF-Parametersystem (75 Parameter, vollständige Konfiguration über TBS Agent / ELRS-Lua)
- Moderne WLAN-Weboberfläche (Dark-Mode-UI, mobilfreundlich, PROGMEM-basiert – kein RAM-Overhead pro Request)
- LiPo-Telemetrie über FrSky S.Port für bis zu zwei Packs (Hardware V4/V5)
- GPS-Geschwindigkeitsmessung über CRSF-GPS-Telemetrie (Hardware V5)
- Granulare S.Port-/CRSF-Diagnosezähler im Debug-Tab zur Fehlersuche
- Multi-Device-tauglich: mehrere CRSF-Konfigurationsgeräte am selben Empfänger, eindeutig adressiert

---

## Hardware-Versionen

| Version | Zusätzliche Eingänge | Besonderheit |
|---|---|---|
| **V1** | GPIO 22, 0, 2, 4 | BUS + PWM-Eingang + GPIO-Pin + Einkanal |
| **V2** | GPIO 14, 27, 32, 33 | BUS + PWM-Eingang + GPIO-Pin + Einkanal |
| **V3** | – | Nur BUS-Kanal + Einkanal (SBUS/CRSF) |
| **V4** | – | Wie V3 + S.Port-LiPo-Telemetrie |
| **V5** | GPIO 27 (GPS RX), 14 (GPS TX) | Wie V4 + GPS-Geschwindigkeitsmessung über CRSF |

Die Platine ab V2 führt physisch dieselben Pins (14, 27, 32, 33): S.Port nutzt 32/33, die freien PWM-Pins 3/4 (GPIO 14/27) übernimmt in V5 das GPS.

---

## Pin-Belegung

### Alle Versionen

| GPIO | Funktion |
|---|---|
| 13 | WiFi-Aktivierung (LOW = AP-Modus) |
| 16 | CRSF RX / SBUS RX |
| 17 | CRSF TX |
| 5 | SD_CS |
| 18 | SD_CLK |
| 19 | SD_MISO |
| 23 | SD_MOSI |
| 21 | I2S_DOUT |
| 25 | I2S_LRC |
| 26 | I2S_BCLK |

### V1 – zusätzliche Eingänge

| GPIO | Funktion |
|---|---|
| 22, 0, 2, 4 | Input 3–6 (PWM/GPIO) |

### V2 – zusätzliche Eingänge

| GPIO | Funktion |
|---|---|
| 14, 27, 32, 33 | Input 3–6 (PWM/GPIO) |

### V4 – S.Port LiPo (zusätzlich)

| GPIO | Funktion |
|---|---|
| 32 | S.Port RX |
| 33 | S.Port TX |

> **Schaltung:** GPIO33 → Anode → [1N4148] → Kathode → S.Port-Signal ← GPIO32, zusätzlich **4,7 kΩ** zwischen S.Port-Signal und GND (Pull-Widerstand — zwingend für stabile Telemetrie). Bei zwei Sensoren am selben Bus genügt ein Widerstand für beide.

### V5 – GPS (zusätzlich zu V4)

| GPIO | Funktion |
|---|---|
| 27 | GPS TX → ESP32 RX (Empfang der NMEA-Daten) |
| 14 | ESP32 TX → GPS RX (Konfiguration, z. B. Updaterate) |

> NMEA-GPS-Modul (ATGM336H oder u-blox NEO-M8N), 9600 Baud, reine Software-Serial. S.Port (32/33) und CRSF (16/17) bleiben unberührt; die 5-Hz-Updaterate wird bei jedem Boot automatisch neu gesetzt.

---

## Hardware-Voraussetzungen

| Komponente | Beschreibung |
|---|---|
| ESP32 | Board-Version 3.3.8 |
| SD-Karte | SPI-Anschluss |
| I2S-DAC/Verstärker | z. B. MAX98357A |
| RC-Empfänger | SBUS oder CRSF (ExpressLRS ab 4.x für Multi-Device) |
| LiPo-BMS mit S.Port | nur für Hardware V4/V5 |
| GPS-Modul (NMEA, UART) | nur für Hardware V5 — ATGM336H oder u-blox NEO-M8N |

---

## Bibliotheken

| Bibliothek | Version / Quelle |
|---|---|
| Bolder Flight Systems SBUS | 8.1.4 |
| CRSF_ESP32 | https://github.com/Ziege-One/CRSF_ESP32 |
| EspSoftwareSerial | nur für Hardware V5 (GPS) |
| TinyGPSPlus | nur für Hardware V5 (GPS) |

---

## SD-Karten-Dateien

| Dateiname | Funktion |
|---|---|
| `loop.wav` | Motor-Laufgeräusch (Schleife) |
| `start.wav` | Motorstart-Geräusch |
| `shut.wav` | Motorabschalt-Geräusch |
| `sound1.wav` – `sound8.wav` | Zusatzsounds 1–8 |

---

## Installation & Flashen

1. Repository klonen bzw. das gewünschte Versions-ZIP entpacken.
2. Benötigte Bibliotheken installieren (siehe oben).
3. Sketch in der Arduino IDE öffnen (`ESP32-RC-Sound.ino`), Board auf ESP32 (3.3.8) stellen.
4. Kompilieren und flashen.
5. WAV-Dateien gemäß Tabelle oben auf die SD-Karte kopieren, Karte einsetzen.
6. Hardware-Version passend zur eigenen Platine in Weboberfläche oder Lua-Menü wählen (siehe [Konfiguration](#konfiguration)) und neu starten.

---

## Konfiguration

### Weboberfläche (alle Versionen)

1. GPIO 13 auf LOW ziehen.
2. ESP32 startet als WLAN-Access-Point.
3. Mit dem konfigurierten Netzwerk verbinden.
4. Weboberfläche im Browser öffnen (Dark-Mode-UI, mobilfreundlich).
5. Einstellungen anpassen und speichern.

### TBS Agent / EdgeTX Lua (CRSF-Modus)

- 75 CRSF-Parameter in Ordnerstruktur: Motor, Sound 1–8, Einstellungen.
- Hardware-Version umschaltbar (Neustart erforderlich).
- Sound-Test direkt aus dem Agent heraus möglich.

### Sound-Wiedergabemodi

| Modus | Solange Quelle „an" | Beim Ausschalten |
|---|---|---|
| **Normal** | spielt einmal durch | kein Einfluss — spielt bis zum Ende |
| **Loop** | wiederholt sich ab Anfang | laufende Runde wird noch zu Ende gespielt, dann Stopp |
| **Tippbetrieb** | spielt einmal | sofortiger Abbruch (falls noch am Laufen) |

---

## Quellentypen

| Code | Quelle | V1/V2 | V3/V4/V5 |
|---|---|---|---|
| 0–15 | BUS-Kanal Low (1–16) | ✓ | ✓ |
| 20–35 | BUS-Kanal High (1–16) | ✓ | ✓ |
| 40–45 | PWM-Eingang Low (1–6) | ✓ | – |
| 50–55 | PWM-Eingang High (1–6) | ✓ | – |
| 60–65 | GPIO-Pin direkt (1–6) | ✓ | – |
| 70–77 | Einkanal-Bit (1–8) | ✓ | ✓ |
| 80–103 | Ebenen-Umschaltung | ✓ | ✓ |
| 200 | Dauerbetrieb | ✓ | ✓ |
| 999 | Deaktiviert | ✓ | ✓ |

---

## Multi-Device-Betrieb

Mehrere CRSF-Konfigurationsgeräte (z. B. dieses Soundmodul plus ein Wilhelm-Meier-Multiswitch) können am selben Empfänger betrieben werden:

- Jedem Modul eine eigene Modul-/WM-Adresse geben — die CRSF-Geräteadresse wird als `0xC0 + Modul-Adresse` abgeleitet (WM0 → 0xC0, WM2 → 0xC2, …).
- Der Empfänger muss ExpressLRS ab Version 4.x fahren, damit Adressen außerhalb 0xC8 weitergeleitet werden.
- Geräteantworten auf den Broadcast-Ping werden per Wilhelm-Meier-Zeitschlitz-Schema (`Slot = (Adresse − 0xC0) × 2`) zeitversetzt gesendet, damit sich mehrere Module nicht überlagern.

---

## Changelog

| Version | Highlights |
|---|---|
| v0.45 → **v0.46** | NVS statt EEPROM, CRSF-Timeout-Failsafe · *v0.46: Playlist-Stop()-Fix (siehe unten)* |
| v0.70 | V1/V2/V3 vereint, CRSF-Parametersystem (75 Parameter) |
| v0.71 → **v0.72** | schlanke Multi-Device-Variante · *v0.72: Playlist-Stop()-Fix* |
| v0.84 | Hardware V4, S.Port-LiPo-Telemetrie, Einzelzellen-SoC |
| v0.86 → **v0.87** | S.Port-Variante mit Absturz-Schutz · *v0.87: Playlist-Stop()-Fix* |
| v1.22 | Weboberfläche als PROGMEM (Flash), ~60 KB RAM-Ersparnis pro Request, Dark-Mode-UI |
| v1.23 | S.Port-Empfangslogik korrigiert, FrSky/FLVSS-Zellendekodierung, CRSF-Voltages statt Batterie-Frame |
| v1.24 → **v1.25** | Eindeutige CRSF-Geräteadresse je WM-Adresse, Wilhelm-Meier-Zeitschlitz-Schema, Parameter-IDs lückenlos (72) · *v1.25: Playlist-Stop()-Fix* |
| v2.0 | Hardware V5 (GPS-Geschwindigkeitsmessung über CRSF-GPS-Telemetrie) |
| v2.2 | Granulare S.Port-/CRSF-Diagnosezähler im Debug-Tab, GPIO27-Konfliktschutz bei V5 + SBUS |
| v2.21 | Bugfix: Ausschalten eines fertig gespielten Tipp-Sounds konnte die komplette Playliste (u. a. Motorsound) kappen |
| **v2.22** | Bugfix: Web-UI zeigte nach dem Speichern von Sound-/Motor-Modus beim Tab-Wechsel wieder den alten Wert (reiner Anzeige-Bug, Speicherung war stets korrekt) |

---

## Bekannt behobene Fehler

### Playlist-Abbruch beim Ausschalten eines Tipp-Sounds (behoben in v0.46 / v0.72 / v0.87 / v1.25 / v2.21)

**Symptom:** Der Motorsound (und andere gerade laufende Sounds) konnte abrupt abbrechen, wenn ein Sound im **Tipp-Modus** ausgeschaltet wurde, nachdem er selbst schon fertig durchgespielt hatte.

**Ursache:** `handleSound()` rief beim Loslassen eines Tipp-Sounds unconditional `I2SAudio.Stop()` auf — auch wenn der Sound sich zuvor schon selbst aus der internen Playliste entfernt hatte. `RemoveFromPlayList()` in `XT_I2S_Audio.cpp` interpretierte das fälschlich als „einziger Eintrag in der Liste" und setzte `FirstPlayListItem`/`LastPlayListItem` bedingungslos auf `nullptr` — das kappte die gesamte aktive Playliste, unabhängig vom tatsächlichen Inhalt.

**Fix:** `Stop()` prüft jetzt vorab mit `AlreadyPlaying()`, ob der Sound überhaupt noch verlinkt ist. Der Fehler steckte identisch seit der ersten Version im Code und wurde rückwirkend in allen gepflegten Zweigen behoben.

### Web-UI zeigte nach dem Speichern den alten Sound-/Motor-Modus (behoben in v2.22)

**Symptom:** Nach dem Speichern eines Sound- oder Motor-Modus über die Weboberfläche zeigte die Auswahlbox beim nächsten Tab-Wechsel wieder den alten Wert — die Speicherung selbst war jedoch stets korrekt (nach Browser-Neuladen war der richtige Wert sichtbar).

**Ursache:** Das lokale `cfg`-Objekt im Browser-JavaScript wurde nach `saveSound()`/`saveMotor()` nicht aktualisiert und beim nächsten Tab-Wechsel aus veralteten Daten neu gerendert.

**Fix:** Beide Funktionen synchronisieren `cfg` jetzt direkt nach erfolgreichem Speichern mit den neuen Werten.# ESP32-Sound-Martin
