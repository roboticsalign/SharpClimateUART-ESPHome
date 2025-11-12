# Buderus Logacool AC186i Setup - Einfache Anleitung

## Schnellstart

Diese Version enthält alle Timing-Verbesserungen und verwendet automatisch die neueste Version vom GitHub.

## 1. YAML-Datei verwenden

Benutzen Sie: **`buderus-buero.yaml`**

Diese Datei:
- ✅ Lädt automatisch die verbesserte Version von GitHub (roboticsalign/SharpClimateUART-ESPHome)
- ✅ Enthält alle Timing-Verbesserungen (15s timeout, 2s init delay, etc.)
- ✅ UART korrekt konfiguriert (4800 Baud, NONE Parity)
- ✅ Keine lokalen Abhängigkeiten
- ✅ Verweist auf den richtigen Fork mit allen Bugfixes

## 2. Secrets-Datei anlegen

Erstellen Sie `secrets.yaml` im selben Verzeichnis:

```yaml
# secrets.yaml
wifi_ssid: "IhrWiFiName"
wifi_password: "IhrWiFiPasswort"
api_key: "IhrAPISchlüssel"  # Generieren mit: esphome run buderus-buero.yaml
ota_password: "IhrOTAPasswort"
```

## 3. Flashen

```bash
esphome run buderus-buero.yaml
```

## 4. Logs beobachten

```bash
esphome logs buderus-buero.yaml
```

**Erfolgsanzeichen:**
- ✅ "Connecting (1/8)" → "Connecting (2/8)" → ... → "Connected"
- ✅ Wenige oder keine "error recovery" Meldungen
- ✅ Regelmäßige Status-Updates

**Bei Problemen:**
- ❌ Viele "error recovery" → Parity testen: `parity: EVEN`
- ❌ Timeouts → Baudrate testen: `baud_rate: 9600`

## Alternative UART-Konfigurationen

Falls `4800 Baud, NONE Parity` nicht funktioniert:

### Test 1: 4800 Baud mit EVEN Parity
```yaml
uart:
  tx_pin: 1
  rx_pin: 3
  baud_rate: 4800
  parity: EVEN
  stop_bits: 1
```

### Test 2: 9600 Baud mit NONE Parity
```yaml
uart:
  tx_pin: 1
  rx_pin: 3
  baud_rate: 9600
  parity: NONE
  stop_bits: 1
```

### Test 3: 9600 Baud mit EVEN Parity (Original)
```yaml
uart:
  tx_pin: 1
  rx_pin: 3
  baud_rate: 9600
  parity: EVEN
  stop_bits: 1
```

## Anpassungen für Ihr Setup

In `buderus-buero.yaml` ändern Sie:

```yaml
substitutions:
  nr: "2"                           # Ihre Nummer
  devicename: esphome-buderus-buero # Ihr Gerätename
  deviceid: hvac${nr}               # Entity ID

# WiFi Domain anpassen oder entfernen:
wifi:
  domain: ".iot.int.dauer.de"  # ← Ändern oder löschen
```

## Hardware-Verbindung

```
Buderus AC Unit          ESP32-C3
────────────────────────────────
Black (GND)  ─────────→  GND
White (TX)   ─────────→  GPIO3 (RX) *
Green (RX)   ─────────→  GPIO1 (TX) *
Red (5V)     ─────────→  VCC (optional)
```

**\* WICHTIG**: 5V ↔ 3.3V Pegelwandler verwenden!

## Features

Automatisch enthalten:
- ✅ Climate-Steuerung (Cool, Heat, Dry, Fan)
- ✅ Swing-Steuerung (Horizontal/Vertikal)
- ✅ Plasmacluster (Ion)
- ✅ Verbindungsstatus-Sensor
- ✅ Reconnect-Button
- ✅ Restart-Button
- ✅ Optimierte Timing-Parameter

## Troubleshooting

### Problem: Compilation Error "Component not found"
→ Warten Sie 30 Sekunden und versuchen Sie erneut. GitHub-Download kann dauern.

### Problem: "Timeout - no response for 15s"
→ Testen Sie andere UART-Parameter (siehe oben)
→ Prüfen Sie Verkabelung und Pegelwandler

### Problem: Viele "error recovery" Meldungen
→ Falsche UART-Parameter → Andere Baudrate/Parity testen
→ Hardware-Problem → Pegelwandler und Kabel prüfen

## Support

Öffnen Sie ein Issue mit:
- Vollständige Logs (erste 100 Zeilen)
- Ihre UART-Konfiguration
- Hardware-Setup (ESP-Modell, Pegelwandler)
- Buderus-Modell

## Technische Details

Diese Version enthält:
- Response-Timeout: 15 Sekunden (statt 10s)
- Init-Retry-Delay: 2 Sekunden zwischen Versuchen
- Error-Counter: 10 Versuche (statt 5)
- Alle Timing-Verbesserungen aus commit fa69c94 und eb126b2

Repository: `roboticsalign/SharpClimateUART-ESPHome` (Fork von sven819)
Branch: `claude/analyze-buderus-logacool-errors-011CV3yzwjNQyuorfWsCc6hW`

**HINWEIS**: Nach Pull Request zum Original-Repository (sven819), ändern Sie in `buderus-buero.yaml`:
```yaml
external_components:
  - source: github://sven819/SharpClimateUART-ESPHome@main
```

**AKTUELL**: Die YAML verweist auf den roboticsalign-Fork mit allen Timing-Verbesserungen!
