# Guition LVGL Home Assistant Display

🎯 **Zusammengeführtes ESPHome-Projekt für Guition ESP32-S3 3.5" Display mit LVGL**

Dieses Repository kombiniert die **funktionierende Hardware-Konfiguration** von [esp_lvg_eigen](https://github.com/sokomania1/esp_lvg_eigen) mit den **erweiterten LVGL-Layouts** von [esphome-guition-display](https://github.com/sokomania1/esphome-guition-display).

## ✅ Import-Status

**Stand: 2026-02-19**

🎉 **VOLLSTÄNDIG IMPORTIERT!**

- ✅ Alle 3 Seiten aus `esphome-guition-display` übertragen
- ✅ Komplettes Layout mit Fonts und Header
- ✅ Zeit- und WiFi-Sensoren implementiert
- ✅ Funktionierende Hardware beibehalten
- ⚠️ **Entity-IDs müssen angepasst werden!** (siehe unten)

📊 **Was ist drin:**
- **Lampen**: 9 Lichter mit Helligkeitssteuerung
- **Heizung**: 3 Thermostate mit Temperatur-Slidern
- **Wetter**: Freudenstadt-Vorhersage + eigene Sensoren

➡️ **Siehe [VALIDATION.md](VALIDATION.md) für Details!**

---

## 🔧 Hardware

- **Display**: Guition ESP32-S3 JC3248W535 (3.5", 480×320 Pixel)
- **Prozessor**: ESP32-S3 mit 16MB Flash, 8MB PSRAM (Octal)
- **Touchscreen**: AXS15231
- **Display-Treiber**: MIPI SPI (ST7701S)

## ⚡ Das Problem (gelöst)

Das `esphome-guition-display`-Projekt hatte **falsche Hardware-Definitionen**:

| Parameter | ❌ esphome-guition-display (falsch) | ✅ esp_lvg_eigen (korrekt) |
|-----------|--------------------------------------|-----------------------------|
| Display-Treiber | `ili9xxx` (ST7796) | `mipi_spi` (JC3248W535) |
| Touch-Treiber | `gt911` | `axs15231` |
| SPI-Pins | GPIO48/47/41 | Quad SPI: 47, 21, 48, 40, 39 |
| I2C-Pins | GPIO6/5 | GPIO4/8 |
| Backlight | GPIO45 | GPIO1 |

**Dieses Projekt nutzt die korrekte Hardware-Konfiguration!**

## 📁 Projektstruktur

```
guition-lvgl-ha-display/
├── guition-display.yaml       # Hauptkonfiguration
├── common.yaml                # Zeit- & WiFi-Sensoren
├── secrets.yaml.example       # Vorlage für Secrets
├── VALIDATION.md              # 🔍 Validierungs-Checkliste (NEU!)
├── devices/
│   └── JC3248W535.yaml       # ✅ Korrekte Hardware-Konfiguration
├── layouts/
│   └── 480x320.yaml          # LVGL-Layout + Fonts + Header
├── pages/
│   ├── lights.yaml           # 💡 9 Lichter mit Slider
│   ├── heating.yaml          # 🌡️ 3 Thermostate
│   └── weather.yaml          # ☁️ Wetter + Sensoren
└── README.md
```

## 🚀 Installation

### 1. Repository klonen

```bash
cd /config/esphome
git clone https://github.com/sokomania1/guition-lvgl-ha-display.git guition
cd guition
```

### 2. Secrets konfigurieren

```bash
cp secrets.yaml.example secrets.yaml
nano secrets.yaml
```

Trage deine Werte ein:

```yaml
wifi_ssid: "Dein_WLAN_Name"
wifi_password: "Dein_WLAN_Passwort"
api_encryption_key: "generierter_32_byte_key"
ota_password: "dein_ota_passwort"
```

### 3. ⚠️ Entity-IDs anpassen (WICHTIG!)

Die Seiten enthalten **Beispiel-Entitäten**. Du **MUSST** diese an deine Home-Assistant-Entitäten anpassen!

**Finde deine Entity-IDs:**
1. Home Assistant → **Entwicklerwerkzeuge** → **Zustände**
2. Suche nach `light.`, `climate.`, `weather.`, `sensor.`
3. Kopiere die IDs

**Passe die Dateien an:**

```bash
# Beispiel: Licht-Entitäten ersetzen
nano pages/lights.yaml
# Ändere: entity_id: light.alle_lichter_2
#     zu: entity_id: light.deine_lampe

nano pages/heating.yaml
nano pages/weather.yaml
```

📝 **Siehe [VALIDATION.md](VALIDATION.md)** für komplette Liste aller Entity-IDs!

### 4. Kompilieren und Flashen

**Erste Installation (USB):**

```bash
esphome run guition-display.yaml
```

**Updates (OTA):**

Nach der ersten Installation kannst du über WLAN updaten:

```bash
esphome run guition-display.yaml --device guition-display.local
```

## 🎨 Eigene Seiten hinzufügen

### Neue Seite erstellen

1. Erstelle `pages/meine-seite.yaml`:

```yaml
# Beispiel: Rollladen-Steuerung
binary_sensor:
  - platform: homeassistant
    id: cover_wohnzimmer_state
    entity_id: cover.wohnzimmer

lvgl:
  pages:
    - id: page_covers
      widgets:
        - button:
            id: btn_cover_up
            x: 50
            y: 100
            width: 100
            height: 60
            widgets:
              - label:
                  text: "▲ AUF"
            on_click:
              - homeassistant.action:
                  action: cover.open_cover
                  data:
                    entity_id: cover.wohnzimmer
```

2. In `guition-display.yaml` einbinden:

```yaml
packages:
  # ...
  meine_seite: !include pages/meine-seite.yaml
```

## 🏠 Home Assistant Integration

Nach dem Flashen wird das Display automatisch in Home Assistant erkannt:

1. **Einstellungen** → **Geräte & Dienste** → **ESPHome**
2. Gerät **"Guition Display"** konfigurieren
3. API-Verschlüsselungsschlüssel aus `secrets.yaml` eintragen

## 🔍 Unterschiede zu den Original-Projekten

### Von `esp_lvg_eigen` übernommen:
- ✅ Komplette Hardware-Konfiguration (`devices/JC3248W535.yaml`)
- ✅ Funktionierende Display- und Touch-Treiber
- ✅ Korrekte Pin-Belegung

### Von `esphome-guition-display` übernommen:
- ✅ Erweiterte LVGL-Seiten (Licht, Heizung, Wetter)
- ✅ Modulare Struktur mit `pages/`-Ordner
- ✅ Header mit Uhrzeit/WiFi-Signal
- ✅ Roboto Fonts (Regular + Bold)
- ✅ Material Design Icons

### Neu in diesem Projekt:
- ✅ Korrigierte Hardware-Definitionen
- ✅ Dokumentation der Unterschiede ([VALIDATION.md](VALIDATION.md))
- ✅ Kompatibilität mit neuesten ESPHome-Versionen
- ✅ Vollständige Import-Validierung

## 📝 Anpassungen

### Startseite ändern

In `guition-display.yaml`:

```yaml
substitutions:
  home_page: "page_lights"  # Ändere auf page_heating, page_weather etc.
```

### Backlight-Helligkeit

In `guition-display.yaml`:

```yaml
esphome:
  on_boot:
    then:
      - light.turn_on:
          id: backlight
          brightness: 80%  # Wert zwischen 0-100%
```

## 🐛 Troubleshooting

### Display bleibt schwarz

- Prüfe, ob Backlight eingeschaltet ist:
  ```yaml
  light.turn_on: backlight
  ```
- In Home Assistant: Suche "Backlight" und schalte auf 100%

### Touch funktioniert nicht

- Prüfe I2C-Verbindung:
  ```yaml
  i2c:
    scan: true  # Zeigt erkannte Geräte im Log
  ```
- Logs prüfen: `esphome logs guition-display.yaml`

### "Entity not found"

- ⚠️ **Du musst die Entity-IDs anpassen!**
- Siehe Abschnitt "Entity-IDs anpassen" oben
- Vollständige Liste in [VALIDATION.md](VALIDATION.md)

### "Font download failed"

- Beim **ersten** Kompilieren werden Fonts aus Google Fonts heruntergeladen
- Das dauert 2-5 Minuten länger
- Bei erneutem Fehler: Internet-Verbindung prüfen

### "Display-Treiber nicht gefunden"

- Stelle sicher, dass du die **korrekte** `devices/JC3248W535.yaml` aus diesem Projekt nutzt
- Nicht die Version aus `esphome-guition-display` verwenden!

## 📚 Weitere Ressourcen

- **[VALIDATION.md](VALIDATION.md)** - Komplette Import-Checkliste
- [ESPHome LVGL Dokumentation](https://esphome.io/components/lvgl/)
- [Home Assistant Community: Guition Displays](https://community.home-assistant.io/t/guition-4-480x480-esp32-s3-4848s040-smart-display-with-lvgl/729271)
- [LVGL Widget-Referenz](https://docs.lvgl.io/master/widgets/index.html)

## 📜 Lizenz

Dieses Projekt kombiniert Code aus:
- [esp_lvg_eigen](https://github.com/sokomania1/esp_lvg_eigen)
- [esphome-guition-display](https://github.com/sokomania1/esphome-guition-display)

Bitte beachte die jeweiligen Lizenzen der Original-Projekte.

---

**Erstellt**: 2026-02-19  
**Letztes Update**: 2026-02-19 21:30 CET  
**ESPHome-Version**: 2024.11.0+  
**Status**: ✅ Vollständiger Import abgeschlossen