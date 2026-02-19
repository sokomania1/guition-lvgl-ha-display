# 🚀 Schnellstart-Anleitung

## 1. Repository klonen

```bash
cd /config/esphome
git clone https://github.com/sokomania1/guition-lvgl-ha-display.git guition
cd guition
```

## 2. Secrets erstellen

```bash
cp secrets.yaml.example secrets.yaml
nano secrets.yaml
```

**Trage ein:**
- WLAN-Name und Passwort
- API-Key (wird beim ersten Kompilieren generiert)
- OTA-Passwort

## 3. Entity-IDs anpassen

Öffne `pages/lights.yaml` und ändere:

```yaml
entity_id: light.wohnzimmer  # <-- Deine echte Entity-ID!
```

Finde deine Entity-IDs in Home Assistant:
- **Entwicklerwerkzeuge** → **Zustände** → nach "light." suchen

## 4. Kompilieren & Flashen

**Erste Installation (USB-Kabel):**

```bash
esphome run guition-display.yaml
```

**Updates (Over-The-Air):**

```bash
esphome run guition-display.yaml --device guition-display.local
```

## 5. In Home Assistant einbinden

Nach dem Flashen:

1. **Einstellungen** → **Geräte & Dienste**
2. ESPHome sollte automatisch "Guition Display" entdecken
3. Klicke auf **Konfigurieren**
4. API-Key aus `secrets.yaml` eintragen

## ✏️ Eigene Seiten hinzufügen

### Beispiel: Rollladen-Seite erstellen

1. **Neue Datei**: `pages/covers.yaml`

```yaml
# Rollladen-Sensor
binary_sensor:
  - platform: homeassistant
    id: cover_wohnzimmer_state
    entity_id: cover.wohnzimmer
    attribute: current_position

lvgl:
  pages:
    - id: page_covers
      widgets:
        - label:
            x: 10
            y: 10
            text: "Rollläden"
        
        - button:
            id: btn_cover_up
            x: 10
            y: 50
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
        
        - button:
            id: btn_cover_down
            x: 130
            y: 50
            width: 100
            height: 60
            widgets:
              - label:
                  text: "▼ AB"
            on_click:
              - homeassistant.action:
                  action: cover.close_cover
                  data:
                    entity_id: cover.wohnzimmer
```

2. **Einbinden** in `guition-display.yaml`:

```yaml
packages:
  # ...
  covers: !include pages/covers.yaml  # <-- Neue Zeile
```

3. **Startseite ändern** (optional):

```yaml
substitutions:
  home_page: "page_covers"  # <-- Neue Startseite
```

## 🛠️ Häufige Anpassungen

### Backlight-Helligkeit ändern

In `guition-display.yaml`:

```yaml
esphome:
  on_boot:
    then:
      - light.turn_on:
          id: backlight
          brightness: 100%  # <-- Ändere auf 0-100%
```

### Display-Rotation ändern

In `devices/JC3248W535.yaml`:

```yaml
display:
  - platform: mipi_spi
    rotation: 90  # <-- 0, 90, 180, 270
```

### Zwischen Seiten wechseln (Button)

```yaml
- button:
    text: "Zurück"
    on_click:
      - lvgl.page.show: page_lights  # <-- Ziel-Seite
```

## 🐛 Probleme?

### "Display bleibt schwarz"

1. Prüfe Backlight in Home Assistant:
   - Suche nach "Backlight" Entity
   - Schalte auf 100%

2. Prüfe Logs:
   ```bash
   esphome logs guition-display.yaml
   ```

### "Touch reagiert nicht"

- I2C-Scan aktivieren in `devices/JC3248W535.yaml`:
  ```yaml
  i2c:
    scan: true  # <-- Zeigt erkannte Geräte
  ```

### "Kompilier-Fehler"

- ESPHome updaten:
  ```bash
  pip install -U esphome
  ```

- Prüfe `secrets.yaml` auf Tippfehler

## 📚 Nächste Schritte

- 📖 Lies die [README.md](README.md) für Details
- 🎨 Erkunde [ESPHome LVGL Docs](https://esphome.io/components/lvgl/)
- 👥 Frage in der [Home Assistant Community](https://community.home-assistant.io/)

---

**Viel Erfolg! 🎉**
