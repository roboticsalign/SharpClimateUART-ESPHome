# Buderus Logacool AC186i - ESPHome Konfiguration

## Verwendung

Kopieren Sie den Inhalt von `buderus-buero.yaml` in die ESPHome Web-Oberfläche in Home Assistant.

## Aktuelle Konfiguration

**UART-Parameter:**
- Baudrate: 9600
- Parity: EVEN
- Stop Bits: 2

**Code-Version:**
- Commit: c44a179
- Timeout: 30 Sekunden (statt 10s)
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

Diese Konfiguration basiert auf Tests, die ergaben:
- 9600 Baud funktioniert besser als 4800
- EVEN Parity ist korrekt
- 2 Stop Bits sind notwendig (1 Stop Bit schlägt bei 2/8 fehl)
- Längere Timeouts notwendig, da AC langsam antwortet

Mit dieser Konfiguration wurde 4/8 erreicht. Die verlängerten Timeouts sollten ausreichen, um 8/8 zu erreichen.
