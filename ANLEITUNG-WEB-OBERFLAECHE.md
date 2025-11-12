# Home Assistant ESPHome Web-Oberfläche - Test-Anleitung

## Schritt-für-Schritt

Da Sie die Home Assistant Web-Oberfläche verwenden, hier die genaue Vorgehensweise:

---

## TEST 1: 4800 Baud, EVEN Parity ⭐

### Schritt 1: YAML kopieren
1. Öffnen Sie die Datei `TEST-1-COPY-PASTE.yaml` in diesem Repository
2. Kopieren Sie **DEN KOMPLETTEN INHALT** (alles von oben bis unten)

### Schritt 2: In Home Assistant einfügen
1. Öffnen Sie Home Assistant → ESPHome Dashboard
2. Klicken Sie auf Ihr Gerät (`esphome-buderus-buero`)
3. Klicken Sie auf **EDIT**
4. **LÖSCHEN Sie den kompletten bisherigen Inhalt**
5. **FÜGEN Sie den kopierten Text ein**
6. Klicken Sie auf **SAVE**

### Schritt 3: Installieren und Logs beobachten
1. Klicken Sie auf **INSTALL**
2. Wählen Sie "Wirelessly" (OTA) wenn bereits verbunden, sonst USB
3. Warten Sie, bis Kompilierung und Installation abgeschlossen sind
4. Klicken Sie auf **LOGS** oder **VIEW LOGS**

### Schritt 4: Logs prüfen (60 Sekunden beobachten)

**✅ ERFOLG - Code geladen:**
```
Timeout - no response for 15s, reconnecting...
                            ^^^^
```
→ Zeigt "**15s**" (nicht 10s!) = Neue Version ist aktiv! ✅

**✅ ERFOLG - Verbindung funktioniert:**
```
[D][sharp_ac.climate:070]: Connecting (1/8)...
[D][sharp_ac.climate:070]: Connecting (2/8)...
[D][sharp_ac.climate:070]: Connecting (3/8)...
...
[D][sharp_ac.climate:070]: Connected
```
→ **GESCHAFFT!** Diese Konfiguration funktioniert! 🎉

**❌ FEHLER - Falsche UART-Parameter:**
```
[D][sharp_ac.climate:070]: RX: FF (error recovery)
[D][sharp_ac.climate:070]: RX: E0 (error recovery)
[D][sharp_ac.climate:070]: RX: C0 (error recovery)
```
→ Viele "error recovery" = Falsche Baudrate/Parity → **Weiter zu TEST 2**

**❌ FEHLER - Alte Version (Cache):**
```
Timeout - no response for 10s, reconnecting...
                            ^^^^
```
→ Zeigt "**10s**" (nicht 15s!) = Cache-Problem!

**Cache-Lösung für Home Assistant:**
1. ESPHome Dashboard → Drei-Punkte-Menü (⋮) → **Clean Build Files**
2. Gerät neu kompilieren und installieren

---

## TEST 2: 9600 Baud, EVEN Parity

**Nur wenn TEST 1 fehlgeschlagen ist!**

### Wiederholen Sie Schritte 1-4 mit `TEST-2-COPY-PASTE.yaml`

Diesmal mit **9600 Baud, EVEN Parity**.

Prüfen Sie die Logs genau wie bei TEST 1.

---

## TEST 3: 9600 Baud, NONE Parity

**Nur wenn TEST 1 UND TEST 2 fehlgeschlagen sind!**

### Wiederholen Sie Schritte 1-4 mit `TEST-3-COPY-PASTE.yaml`

Diesmal mit **9600 Baud, NONE Parity**.

Prüfen Sie die Logs genau wie bei TEST 1.

---

## Checkliste beim Testen

Für jede Konfiguration:

- [ ] Kompletten YAML-Inhalt kopiert
- [ ] Alten Inhalt in ESPHome gelöscht
- [ ] Neuen Inhalt eingefügt
- [ ] Gespeichert und installiert
- [ ] Logs 60 Sekunden beobachtet
- [ ] Timeout-Wert geprüft (15s oder 10s?)
- [ ] "Connected" oder "error recovery"?

---

## Was tun bei Erfolg?

Wenn eine Konfiguration funktioniert (z.B. TEST 1):

1. ✅ **Notieren Sie die funktionierende Kombination**
   - Beispiel: "TEST 1 funktioniert: 4800 Baud, EVEN Parity"

2. ✅ **Lassen Sie diese YAML installiert**
   - Nicht mehr ändern!

3. ✅ **Testen Sie das Klimagerät**
   - Home Assistant → Klimaanlage einschalten
   - Temperatur ändern
   - Modi testen

4. ✅ **Melden Sie den Erfolg**
   - Teilen Sie mit, welche Konfiguration funktioniert hat
   - Damit können wir die Dokumentation aktualisieren

---

## Was tun wenn nichts funktioniert?

Falls **alle 3 Tests** fehlschlagen:

### Option A: Cache-Problem
Wenn die Logs immer noch "10s" zeigen (nicht "15s"):

1. ESPHome Dashboard → Gerät auswählen
2. Drei-Punkte-Menü (⋮) → **Clean Build Files**
3. Gerät neu kompilieren und alle 3 Tests wiederholen

### Option B: Hardware-Problem
Wenn die Logs "15s" zeigen, aber viele "error recovery":

1. **Prüfen Sie die Verkabelung:**
   - Ist das Klimagerät eingeschaltet?
   - Sind TX/RX richtig verbunden?
   - Ist ein Pegelwandler (5V ↔ 3.3V) vorhanden?

2. **Testen Sie die Verbindung:**
   - Messen Sie mit Multimeter die Spannungen
   - Prüfen Sie auf lose Kontakte

3. **Öffnen Sie ein GitHub Issue** mit:
   - Logs aller 3 Konfigurationen (jeweils erste 100 Zeilen)
   - Hardware-Setup (ESP-Modell, Pegelwandler ja/nein)
   - Buderus-Modell (AC186i)

---

## Unterschiede der 3 Tests

| Test | Baudrate | Parity | Wahrscheinlichkeit | Datei |
|------|----------|--------|-------------------|-------|
| 1 | 4800 | EVEN | ⭐⭐⭐ Hoch | `TEST-1-COPY-PASTE.yaml` |
| 2 | 9600 | EVEN | ⭐⭐ Mittel | `TEST-2-COPY-PASTE.yaml` |
| 3 | 9600 | NONE | ⭐ Niedrig | `TEST-3-COPY-PASTE.yaml` |

**Tipp:** TEST 1 hat die höchste Erfolgswahrscheinlichkeit für HVAC-Systeme!

---

## Technische Details

Alle 3 Test-YAMLs enthalten:

- ✅ Direkte GitHub-Referenz: `@fa69c94` (Commit mit allen Verbesserungen)
- ✅ `refresh: 60s` (stellt sicher, dass Code neu geladen wird)
- ✅ Timeout: 15 Sekunden (statt 10s)
- ✅ Init-Retry-Delay: 2 Sekunden
- ✅ Error-Counter: 10 Versuche (statt 5)

**Unterschied zwischen den Tests:** Nur die UART-Parameter (Baudrate, Parity)

---

## Hilfreiche Home Assistant Befehle

### Logs live ansehen
ESPHome Dashboard → Gerät → **LOGS**

### Build-Cache löschen
ESPHome Dashboard → Gerät → ⋮ (Drei Punkte) → **Clean Build Files**

### Gerät neu starten
ESPHome Dashboard → Gerät → ⋮ → **Restart Device**

### Offline-Modus (falls GitHub-Download nicht funktioniert)
Falls der externe Component-Download fehlschlägt, können Sie die Komponente lokal integrieren. Fragen Sie in diesem Fall nach weiteren Anweisungen.

---

## Zusammenfassung

1. **Kopieren** Sie `TEST-1-COPY-PASTE.yaml`
2. **Einfügen** in Home Assistant ESPHome Editor
3. **Installieren** und Logs beobachten
4. **Prüfen**: Zeigt es "15s" und "Connected"?
5. **Bei Erfolg**: Fertig! 🎉
6. **Bei Fehler**: Nächsten Test probieren

Viel Erfolg! 🚀
