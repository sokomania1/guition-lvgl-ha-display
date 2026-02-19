# Guition LVGL Home Assistant Display

🎯 **Zusammengeführtes ESPHome-Projekt für Guition ESP32-S3 3.5" Display mit LVGL**

Dieses Repository kombiniert die **funktionierende Hardware-Konfiguration** von [esp_lvg_eigen](https://github.com/sokomania1/esp_lvg_eigen) mit den **erweiterten LVGL-Layouts** von [esphome-guition-display](https://github.com/sokomania1/esphome-guition-display).

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
├── common.yaml                # Gemeinsame ESPHome-Einstellungen
├── secrets.yaml.example       # Vorlage für Secrets
├── devices/
│   └── JC3248W535.yaml       # ✅ Korrekte Hardware-Konfiguration
├── layouts/
│   └── 480x320.yaml          # LVGL-Basis-Layout
├── pages/
│   ├── lights.yaml           # Licht-Steuerung
│   ├── heating.yaml          # Heizungs-Steuerung
│   └── weather.yaml          # Wetter-Anzeige
└── README.md
```

## 🚀 Installation

### 1. Repository klonen

```bash
cd /config/esphome
git clone https://github.com/sokomania1/guition-lvgl-ha-display.git
cd guition-lvgl-ha-display
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

### 3. Kompilieren und Flashen

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
- ✅ Detaillierte Home-Assistant-Integration

### Neu in diesem Projekt:
- ✅ Korrigierte Hardware-Definitionen
- ✅ Dokumentation der Unterschiede
- ✅ Kompatibilität mit neuesten ESPHome-Versionen

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

### Touch funktioniert nicht

- Prüfe I2C-Verbindung:
  ```yaml
  i2c:
    scan: true  # Zeigt erkannte Geräte im Log
  ```

### "Display-Treiber nicht gefunden"

- Stelle sicher, dass du die **korrekte** `devices/JC3248W535.yaml` aus diesem Projekt nutzt
- Nicht die Version aus `esphome-guition-display` verwenden!

## 📚 Weitere Ressourcen

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
**Letztes Update**: 2026-02-19  
**ESPHome-Version**: 2024.x+