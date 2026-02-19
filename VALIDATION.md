# ✅ Validierungs-Checkliste: Vollständiger Import

## Import-Status

Dieser Commit importiert **alle Seiten und Entitäten** aus `esphome-guition-display` in das neue Repo mit funktionierender Hardware.

---

## ✅ Importierte Komponenten

### Seiten (100% vollständig)
- ✅ **pages/lights.yaml** - 9 Lichter mit Helligkeitssteuerung
  - Alle Lichter (Gruppenschalter + Slider)
  - Stehlampe Bibi
  - Garderobe
  - Wohnzimmer Schrank
  - Esszimmer
  - Küche Unterlicht
  - Flur Licht
  - Aktenschrank Markus
  - Bibis Ecke

- ✅ **pages/heating.yaml** - 3 Thermostate mit Slidern
  - Wohnzimmer Heizung (Temperatur + Batterie)
  - Büro Markus Heizung (Temperatur + Batterie)
  - Schlafzimmer Heizung (Temperatur + Batterie)

- ✅ **pages/weather.yaml** - Wetter Freudenstadt
  - Wettervorhersage (Temperatur, Zustand, Icons)
  - Luftfeuchtigkeit, Wind, Niederschlag
  - Eigener Außensensor (SNF Draußen)
  - Innentemperaturen (Wohnzimmer, Markus, Schlafzimmer)

### Layouts & Fonts
- ✅ **layouts/480x320.yaml**
  - Material Design Icons (48px + 24px)
  - Roboto Regular (12, 16, 20, 24, 32)
  - Roboto Bold (24, 32)
  - Top-Bar Header mit:
    - Uhrzeit (links)
    - Seitentitel (mittig)
    - WiFi-Signal (rechts)
  - Theme (Dunkelmodus)

### Gemeinsame Komponenten
- ✅ **common.yaml**
  - Zeit-Sensor (ha_time, Europe/Berlin)
  - Zeit-Formatierung (current_time_badge, current_date)
  - WiFi-Info (Signal, Stärke in %, SSID, IP, MAC)
  - Status-LED (GPIO2)
  - Restart-Switch
  - Global: homeassistant_ip

### Hardware (Funktionierende Konfiguration beibehalten!)
- ✅ **devices/JC3248W535.yaml**
  - **Display**: MIPI SPI (JC3248W535) - ID: `main_display`
  - **Touch**: AXS15231 - ID: `main_touchscreen`
  - **Backlight**: PWM GPIO1 - ID: `backlight`
  - ESP32-S3 mit PSRAM (Octal, 80MHz)
  - Quad SPI Pins korrekt

---

## 🔍 Code-Prüfung

### Konsistenz-Checks

✅ **Display-/Touch-IDs**
```yaml
# devices/JC3248W535.yaml definiert:
display:
  - id: main_display        # ✅ Wird verwendet
touchscreen:
  - id: main_touchscreen    # ✅ Wird verwendet

# layouts/480x320.yaml referenziert nicht explizit
# (LVGL nutzt automatisch das erste Display/Touchscreen)
```

✅ **Backlight-ID**
```yaml
# devices/JC3248W535.yaml:
light:
  - id: backlight           # ✅ Definiert

# guition-display.yaml:
on_boot:
  - light.turn_on:
      id: backlight         # ✅ Korrekt referenziert
```

✅ **Font-Referenzen**
```yaml
# layouts/480x320.yaml definiert:
font:
  - id: roboto_16           # ✅ Standard
  - id: roboto_20           # ✅ Genutzt in Seiten
  - id: roboto_24           # ✅ Genutzt in Seiten
  - id: roboto_bold_24      # ✅ Genutzt in Header
  - id: roboto_bold_32      # ✅ Genutzt in Seiten

# Alle Seiten nutzen diese Fonts korrekt!
```

✅ **Sensor-IDs für Header**
```yaml
# common.yaml definiert:
text_sensor:
  - id: current_time_badge  # ✅ Für Uhrzeit
  - id: current_date        # ✅ Für Datum
sensor:
  - id: wifi_strength       # ✅ Für WiFi-Balken

# layouts/480x320.yaml interval nutzt diese:
interval:
  - then:
      - lvgl.label.update:
          id: time_label
          text: !lambda return id(current_time_badge).state.c_str();
      - lvgl.label.update:
          id: wifi_label
          text: !lambda ...wifi_strength...
```

---

## 🛠️ Bekannte Anpassungen nötig

### Home Assistant Entitäten

Alle Seiten nutzen **deine spezifischen Entity-IDs**. Du musst diese an dein HA anpassen:

**lights.yaml:**
```yaml
# Beispiel-Entitäten (ALLE ändern!):
entity_id: light.alle_lichter_2
entity_id: light.stehlampe_bibi
entity_id: light.garderobe_licht
# ... usw.
```

**heating.yaml:**
```yaml
# Beispiel-Entitäten:
entity_id: climate.wohnzimmer_heizung
entity_id: climate.buro_markus_heizung
entity_id: climate.schlafzimmer_heizung
entity_id: sensor.wohnzimmer_heizung_battery
# ... usw.
```

**weather.yaml:**
```yaml
# Beispiel-Entitäten:
entity_id: weather.freudenstadt
entity_id: sensor.freudenstadt_luftfeuchtigkeit
entity_id: sensor.snf_draussen_temperature
# ... usw.
```

### Empfehlung

1. **Suche & Ersetze** in allen `pages/*.yaml`:
   ```bash
   # Finde alle entity_id:
   grep -r "entity_id:" pages/
   
   # Ersetze mit deinen Entitäten
   sed -i 's/light.alte_id/light.neue_id/g' pages/lights.yaml
   ```

2. **Teste schrittweise:**
   - Erst nur `pages/lights.yaml` mit 1-2 Lichtern testen
   - Dann `heating.yaml` aktivieren
   - Zuletzt `weather.yaml` anpassen

---

## 🚀 Nächste Schritte

### 1. Git Pull
```bash
cd /config/esphome/guition
git pull
```

### 2. Entity-IDs anpassen
Öffne jede Seite und passe die `entity_id:` an:
```bash
nano pages/lights.yaml
nano pages/heating.yaml
nano pages/weather.yaml
```

### 3. Secrets prüfen
```bash
cat secrets.yaml
```
Stelle sicher, dass vorhanden sind:
- `wifi_ssid`
- `wifi_password`
- `api_encryption_key`
- `ota_password`

### 4. Kompilieren
```bash
esphome run guition-display.yaml
```

### 5. Bei Fehlern

**"Entity not found"**
- → Prüfe Entity-IDs in Home Assistant (Entwicklerwerkzeuge → Zustände)

**"Font not found"**
- → Beim ersten Kompilieren werden Fonts heruntergeladen (dauert länger)

**"Display bleibt schwarz"**
- → Backlight in HA auf 100% schalten
- → Logs prüfen: `esphome logs guition-display.yaml`

---

## 🎉 Erwartetes Ergebnis

Nach erfolgreichem Kompilieren und Flashen:

✅ Display zeigt **Lampen-Seite** mit:
- Header: Uhrzeit, Titel "Lampen", WiFi-Balken
- Großer "Alle Lichter" Button mit Helligkeits-Slider
- 8 individuelle Licht-Buttons in Grid-Layout
- Buttons wechseln Farbe (orange = an, grau = aus)

✅ **Navigation** (wenn implementiert):
- Wischen/Buttons zu "Heizung" und "Wetter" Seiten

✅ **Live-Updates**:
- Header-Uhrzeit aktualisiert jede Sekunde
- WiFi-Balken zeigt Signal-Stärke
- Buttons spiegeln Lichtzustand aus HA
- Thermostat-Werte aktualisieren sich

---

## 📝 Changelog

**2026-02-19 - Vollständiger Import**
- ✅ Alle 3 Seiten aus `esphome-guition-display` importiert
- ✅ Fonts (Roboto + Bold + Material Icons) hinzugefügt
- ✅ Header mit Uhrzeit/WiFi implementiert
- ✅ Zeit- und WiFi-Sensoren aus `common.yaml` übernommen
- ✅ Funktionierende Hardware (`devices/JC3248W535.yaml`) beibehalten
- ✅ Alle Entity-IDs aus Original übernommen (müssen angepasst werden)

---

**👉 Bei Problemen:** Kopiere die komplette Fehlermeldung und frage nach!
