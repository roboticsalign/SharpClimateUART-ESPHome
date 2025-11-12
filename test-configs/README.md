# UART Test-Konfigurationen für Buderus AC186i

## Problem erkannt

Ihre Logs zeigen **"Timeout - no response for 10s"**, aber die verbesserte Version sollte **"Timeout - no response for 15s"** anzeigen.

**Das bedeutet**: ESPHome lädt eine **gecachte alte Version** des Codes!

## Lösung

Alle Test-Configs verwenden jetzt:
```yaml
external_components:
  - source: github://roboticsalign/SharpClimateUART-ESPHome@fa69c94
```

**@fa69c94** ist ein spezifischer Commit-Hash (statt Branch-Name). Das zwingt ESPHome, die richtige Version zu laden.

---

## Systematisches Testen

Da die Hardware früher funktioniert hat, ist das Problem die **UART-Konfiguration**. Testen Sie diese Kombinationen **in dieser Reihenfolge**:

### **CONFIG 1: 4800 Baud, EVEN Parity** ⭐⭐⭐ (Empfohlen!)
```bash
esphome run test-configs/config-1-4800-even.yaml
```
**Warum zuerst?** Häufigste Konfiguration für HVAC-Systeme

**Erfolgsanzeichen**:
- "Timeout - no response for **15s**" (nicht 10s!)
- Weniger "error recovery" Meldungen
- "Connecting (1/8)" → "Connecting (2/8)" → ... → "Connected"

**Failure-Anzeichen**:
- Immer noch "10s" Timeout → Cache-Problem, nochmal compilieren!
- Viele "error recovery" → Falsche Parameter, nächste Config testen

---

### **CONFIG 2: 9600 Baud, EVEN Parity** ⭐⭐
```bash
esphome run test-configs/config-2-9600-even.yaml
```
**Warum?** Ihre ursprüngliche Konfiguration

---

### **CONFIG 3: 9600 Baud, NONE Parity** ⭐
```bash
esphome run test-configs/config-3-9600-none.yaml
```
**Warum?** Manche Geräte verwenden NONE statt EVEN

---

## Wichtige Hinweise

### 1. Cache löschen (falls immer noch "10s" Timeout)

Wenn die Logs immer noch "10s" zeigen:

```bash
# ESPHome Cache manuell löschen
rm -rf .esphome/build/
rm -rf .esphome/storage/

# Dann neu compilieren
esphome run test-configs/config-1-4800-even.yaml
```

### 2. Was in den Logs beachten

**✅ Erfolg = Code geladen**:
```
[timestamp][D][sharp_ac.climate:070]: Timeout - no response for 15s, reconnecting...
```
→ Zeigt "15s" statt "10s" = Neue Version ist aktiv!

**✅ Erfolg = Verbindung funktioniert**:
```
[timestamp][D][sharp_ac.climate:070]: Connecting (1/8)...
[timestamp][D][sharp_ac.climate:070]: Connecting (2/8)...
[timestamp][D][sharp_ac.climate:070]: Connecting (3/8)...
...
[timestamp][D][sharp_ac.climate:070]: Connected
```

**❌ Failure = Falsche UART-Parameter**:
```
[timestamp][D][sharp_ac.climate:070]: RX: FF (error recovery)
[timestamp][D][sharp_ac.climate:070]: RX: E0 (error recovery)
[timestamp][D][sharp_ac.climate:070]: RX: C0 (error recovery)
```
→ Viele einzelne Bytes = Baudrate oder Parity falsch

**❌ Failure = Alte Version geladen**:
```
[timestamp][D][sharp_ac.climate:070]: Timeout - no response for 10s, reconnecting...
```
→ Zeigt "10s" statt "15s" = Cache löschen und neu compilieren!

---

## Test-Protokoll

Füllen Sie beim Testen aus:

| Config | Baudrate | Parity | Timeout angezeigt? | Verbindung? | Notizen |
|--------|----------|--------|-------------------|-------------|---------|
| 1 | 4800 | EVEN | [ ] 10s [ ] 15s | [ ] Ja [ ] Nein | |
| 2 | 9600 | EVEN | [ ] 10s [ ] 15s | [ ] Ja [ ] Nein | |
| 3 | 9600 | NONE | [ ] 10s [ ] 15s | [ ] Ja [ ] Nein | |

---

## Wenn nichts funktioniert

Falls **alle** Konfigurationen fehlschlagen:

1. **Überprüfen Sie die Hardware**:
   - Ist das Klimagerät eingeschaltet?
   - Ist der Pegelwandler (5V ↔ 3.3V) korrekt angeschlossen?
   - Sind TX/RX gekreuzt? (ESP TX → AC RX, ESP RX → AC TX)

2. **Versuchen Sie andere Baudraten**:
   Editieren Sie eine der Configs und testen Sie:
   - 2400 Baud
   - 19200 Baud

3. **Öffnen Sie ein GitHub Issue** mit:
   - Vollständige Logs aller 3 Konfigurationen
   - Hardware-Setup (ESP-Modell, Pegelwandler-Modell)
   - Buderus-Modell

---

## Nach erfolgreicher Konfiguration

Wenn eine Config funktioniert:

1. Kopieren Sie die funktionierende YAML nach `buderus-buero.yaml`
2. Dokumentieren Sie die erfolgreiche Kombination
3. Flashen Sie mit der finalen Config

**Beispiel**: Wenn CONFIG 1 funktioniert:
```bash
cp test-configs/config-1-4800-even.yaml buderus-buero.yaml
esphome run buderus-buero.yaml
```

Viel Erfolg! 🚀
