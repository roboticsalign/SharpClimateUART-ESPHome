# Buderus Logacool AC186i - ESPHome Konfiguration

## Verwendung

Kopieren Sie den Inhalt von `buderus-buero.yaml` in die ESPHome Web-Oberfläche in Home Assistant.

## Aktuelle Konfiguration

**UART-Parameter:**
- Baudrate: 9600
- Parity: EVEN
- Stop Bits: 1 (Standard - NICHT 2!)

**WICHTIG:** `stop_bits: 2` verursacht falsche Byte-Interpretation!

**Code-Version:**
- Standard Sharp HVAC Protocol (erfolgreich getestet im Original-Repo)
- Timeout: 30 Sekunden
- Init-Retry-Delay: 5 Sekunden

## Erfolgs-Kriterium

In den Logs muss erscheinen:
```
Timeout - no response for 30s, reconnecting...
```

Wenn Sie "10s" sehen, wird die alte gecachte Version geladen:
1. ESPHome → Gerät → ⋮ → Clean Build Files
2. Neu installieren

Falls das nicht hilft:
1. ESPHome → Gerät → ⋮ → Delete
2. Gerät komplett neu anlegen mit buderus-buero.yaml

## Verbindungsablauf

```
Connecting (1/8)...
Connecting (2/8)...
Connecting (3/8)...
Connecting (4/8)...
Connecting (5/8)...
Connecting (6/8)...
Connecting (7/8)...
Connecting (8/8)...
Connected
```

## Technische Details

Diese Konfiguration entspricht der erfolgreich getesteten Referenz (Original-Repo):
- 9600 Baud
- EVEN Parity
- 1 Stop Bit (Standard - kein stop_bits Parameter!)

**Problem mit 2 Stop Bits:** Bytes werden falsch gelesen (0x02 → 0x06 ACK), Verbindung kommt nicht zustande.
