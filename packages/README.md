# Buderus Logacool ESPHome Packages

Wiederverwendbare Konfigurationspakete für Buderus Logacool Klimageräte mit ESPHome.

## Verwendung

### Methode 1: Lokales Package (empfohlen für Entwicklung)

1. Kopieren Sie `packages/buderus_ac186i_base.yaml` in Ihr ESPHome-Verzeichnis
2. Erstellen Sie Ihre Gerätekonfiguration:

```yaml
substitutions:
  devicename: mein-klimageraet
  deviceid: hvac1
  ac_name: "Wohnzimmer AC"

  # UART Pin-Konfiguration
  uart_tx_pin: "1"
  uart_rx_pin: "3"

  # UART-Parameter (getestet mit AC186i)
  uart_baud: "4800"
  uart_parity: "NONE"
  uart_stop_bits: "1"

  log_level: "DEBUG"

esphome:
  name: ${devicename}

esp32:  # oder esp8266
  board: esp32-c3-devkitm-1

# Package importieren
packages:
  buderus_ac: !include packages/buderus_ac186i_base.yaml

api:
  # ... Ihre API-Konfiguration

wifi:
  # ... Ihre WiFi-Konfiguration
```

### Methode 2: GitHub Package (für Produktion)

Erstellen Sie in Ihrem Repository eine Datei `packages/buderus_ac186i.yaml` und nutzen Sie:

```yaml
packages:
  buderus_ac: github://sven819/SharpClimateUART-ESPHome/packages/buderus_ac186i_base.yaml@main
```

## Erforderliche Substitutions

| Variable | Beschreibung | Beispiel |
|----------|--------------|----------|
| `deviceid` | Eindeutige ID für die Climate-Entity | `hvac1`, `hvac2` |
| `ac_name` | Anzeigename für die Klimaanlage | `"Wohnzimmer AC"` |
| `log_level` | Log-Level | `DEBUG`, `INFO` |

## Optionale Substitutions (mit Standardwerten)

| Variable | Standard | Beschreibung |
|----------|----------|--------------|
| `uart_tx_pin` | `"1"` | GPIO für UART TX |
| `uart_rx_pin` | `"3"` | GPIO für UART RX |
| `uart_baud` | `"4800"` | UART Baudrate |
| `uart_parity` | `"NONE"` | UART Parity |
| `uart_stop_bits` | `"1"` | UART Stop Bits |

## Unterstützte Geräte

- Buderus Logacool AC166i (getestet)
- Buderus Logacool AC186i (in Entwicklung)
- Buderus Logacool AC196i
- Buderus Logacool AC176i.2
- Buderus Logacool AC186.2

## UART-Konfigurationen zum Testen

Wenn die Standardkonfiguration nicht funktioniert, probieren Sie:

### Config 1: 4800 Baud, NONE Parity (Standard)
```yaml
uart_baud: "4800"
uart_parity: "NONE"
uart_stop_bits: "1"
```

### Config 2: 4800 Baud, EVEN Parity
```yaml
uart_baud: "4800"
uart_parity: "EVEN"
uart_stop_bits: "1"
```

### Config 3: 9600 Baud, NONE Parity
```yaml
uart_baud: "9600"
uart_parity: "NONE"
uart_stop_bits: "1"
```

### Config 4: 9600 Baud, EVEN Parity
```yaml
uart_baud: "9600"
uart_parity: "EVEN"
uart_stop_bits: "1"
```

## Hardware-Verbindung

```
Buderus AC Unit (PAP-08V-S)    ESP32/ESP8266
──────────────────────────────────────────────
Black (GND)  ──────────────→ GND
White (TX)   ──────────────→ RX (GPIO3) *
Green (RX)   ──────────────→ TX (GPIO1) *
Red (5V)     ──────────────→ VCC (optional)
```

**\* WICHTIG**: Verwenden Sie einen 5V ↔ 3.3V Pegelwandler für TX/RX!

## Features

Das Package enthält automatisch:

- ✅ Climate-Entity mit allen Modi (Cool, Heat, Dry, Fan)
- ✅ Horizontale und vertikale Swing-Steuerung
- ✅ Plasmacluster (Ion) Switch
- ✅ Connection Status Sensor
- ✅ Reconnect Button
- ✅ Optimierte Timing-Parameter
- ✅ Automatische Fehlerbehandlung

## Beispiele

Siehe `/examples/` Verzeichnis für vollständige Konfigurationsbeispiele:

- `example_esp32c3.yaml` - ESP32-C3 Konfiguration
- `example_esp8266.yaml` - ESP8266 Konfiguration
- `example_multi_ac.yaml` - Mehrere Klimageräte

## Troubleshooting

### Logs zeigen "Timeout - no response"

1. Überprüfen Sie die UART-Parameter (Baudrate, Parity)
2. Testen Sie verschiedene Konfigurationen (siehe oben)
3. Überprüfen Sie den Pegelwandler
4. Prüfen Sie die Verkabelung (TX ↔ RX gekreuzt!)

### Logs zeigen viele "error recovery"

- Falsche UART-Parameter → Andere Baudrate/Parity testen
- Fehlender/defekter Pegelwandler → Hardware überprüfen
- Schlechte Kabelverbindung → Kabel kürzen, Kontakte prüfen

### Weitere Hilfe

Öffnen Sie ein Issue im GitHub-Repository mit:
- Vollständige Logs (erste 100 Zeilen nach Start)
- Ihre YAML-Konfiguration (ohne Secrets)
- Hardware-Setup (ESP-Modell, Pegelwandler)
- Buderus-Modell

## Lizenz

Siehe LICENSE im Repository-Root
