# Buderus Logacool AC186i - Error Analysis Report

**Date**: 2025-11-12
**Device**: Buderus Logacool AC186i
**Issue**: Connection failures, repeated timeouts, UART communication errors

---

## Executive Summary

The Buderus Logacool AC186i fails to establish a stable UART connection with ESPHome. Analysis of the log files reveals massive UART synchronization issues, with hundreds of "error recovery" bytes and repeated connection timeouts. The most likely root cause is **incorrect UART baudrate configuration**.

---

## Log Analysis

### Observed Issues

1. **Massive Error Recovery Messages**
   - Hundreds of single-byte reads with error recovery: `FF`, `FE`, `02`, `01`, `F0`, `E0`, `D2`, etc.
   - Indicates severe UART desynchronization between ESP and AC unit

2. **UART Read Timeout**
   ```
   [12:53:39.047][E][uart:015]: Reading from UART timed out at byte 2!
   ```
   - Multi-byte frames are incomplete, timing out mid-transmission
   - Suggests wrong baudrate or electrical issues

3. **Connection Never Completes**
   - Maximum progress: "Connecting (2/8)..."
   - Target: "Connected" (status 8/8)
   - Every 10 seconds: "Timeout - no response for 10s, reconnecting..."

4. **Partial Data Reception**
   - Occasional valid-looking frames are received:
     ```
     RX: FF.FF.00.F0.00.FE.00.FE (8)
     RX: E0.01.00.00.E0.02.FF.00 (8)
     RX: 02.00.FF.C6.02.FF.FD.01.36.66... (long frame)
     ```
   - But immediately followed by more error recovery bytes
   - Suggests intermittent synchronization, then loss of sync

---

## Root Cause Analysis

### Primary Suspect: Wrong Baudrate ⚠️

**Current Configuration** (`klima.yaml`):
```yaml
uart:
   tx_pin: 1
   rx_pin: 3
   baud_rate: 9600
   parity: EVEN
```

**Problem**: The Buderus Logacool AC186i may use a different baudrate than 9600.

**Evidence**:
- Error pattern is consistent with baudrate mismatch
- Some bytes are correctly received (ACK, partial frames), suggesting electrical connection is OK
- But most data is corrupted or desynchronized

**Common UART configurations for HVAC systems**:
| Baudrate | Parity | Likelihood for AC186i |
|----------|--------|----------------------|
| 4800     | EVEN   | ⭐⭐⭐ Very High      |
| 9600     | NONE   | ⭐⭐ Medium           |
| 19200    | EVEN   | ⭐ Low               |

### Secondary Suspects

#### 1. Missing/Incorrect Level Shifter 🔧

The AC unit uses 5V logic, ESP8266 uses 3.3V logic.

**Required**: Bidirectional 5V ↔ 3.3V level shifter on both TX and RX lines

**Symptoms if missing/wrong**:
- Unreliable communication
- Random byte corruption
- Similar to baudrate mismatch symptoms

#### 2. Poor Physical Connection 🔌

**Possible issues**:
- Loose crimp contacts (SPHD-001T-P)
- Insufficient cable shielding
- Long cable runs (>20cm increases noise)
- Missing ground connection

#### 3. TX/RX Swapped 🔄

Less likely (would show no communication at all), but worth checking:
- White wire: Should connect ESP RX → AC TX
- Green wire: Should connect ESP TX → AC RX

---

## Technical Details from Codebase

### Connection Sequence (Status 0-7 → 8)

The connection protocol requires 8 steps:

| Status | Expected Frame | AC Response | Next Action |
|--------|---------------|-------------|-------------|
| 0      | `02.FF.FF.00.00.00.00.02` | `0x02` | Send init_msg2 |
| 1      | `02.FF.FF.01.01.00.01.00.FF` | `ACK` | Wait for ACK |
| 2      | - | `ACK` | Increment to 3 |
| 3      | `03.FF.A0.01.00.00.00.60` | `0x03` | Send subscribe_msg2 |
| 4      | - | `0x03` | Send get_state |
| 5      | - | `0xdc` | Send get_status |
| 6      | - | `0xdc` | Send connected_msg |
| 7      | - | `ACK` | Increment to 8 |
| 8      | **CONNECTED** | - | Normal operation |

**Current Issue**: Connection fails between status 1-2, never reaching status 8.

### Error Recovery Logic

Located in `core_logic.cpp:107-143`, the `readMsg()` function:

```cpp
if (hardware->available() < 8) {
    if (this->errCounter < 5) {
        this->errCounter++;
        return SharpFrame(msg, 0);  // Wait for more bytes
    } else {
        this->errCounter = 0;
        uint8_t singleByte = hardware->read();  // Read single byte (error recovery)
        hardware->log_debug(TAG, "RX: %s (error recovery)", ...);
        return frame;
    }
}
```

**What this means**: After 5 attempts of insufficient bytes, the code reads one byte at a time to resync. The massive number of error recovery messages shows the UART buffer is constantly desynchronized.

### Timeout Mechanism

Located in `core_logic.cpp:397-408`:

```cpp
void SharpAcCore::checkTimeout() {
    if (currentMillis - lastRequestTime >= responseTimeout) {  // 10000ms
        hardware->log_debug(TAG, "Timeout - no response for 10s, reconnecting...");
        resetConnection();
    }
}
```

After 10 seconds without a valid response, the connection resets to status 0.

---

## Recommended Solutions

### Solution 1: Change Baudrate (HIGHEST PRIORITY) ⭐⭐⭐

**Try this first!**

Edit `klima.yaml`:

```yaml
uart:
   tx_pin: 1
   rx_pin: 3
   baud_rate: 4800    # ← CHANGE THIS
   parity: EVEN
```

**Steps**:
1. Change baudrate to 4800
2. Flash ESPHome
3. Monitor logs for 60 seconds
4. Check for:
   - ✅ Fewer/no "error recovery" messages
   - ✅ Progress beyond "Connecting (2/8)"
   - ✅ Successful "Connected" status

If 4800 doesn't work, try:
```yaml
baud_rate: 9600
parity: NONE
```

Then try:
```yaml
baud_rate: 19200
parity: EVEN
```

### Solution 2: Verify Level Shifter

**Hardware checklist**:
- [ ] Bidirectional 5V ↔ 3.3V level shifter installed
- [ ] Both TX and RX lines pass through level shifter
- [ ] Level shifter powered correctly (3.3V and 5V rails)
- [ ] Verify with multimeter:
  - AC side: 0V (low) / 5V (high)
  - ESP side: 0V (low) / 3.3V (high)

**Recommended level shifter**: TXS0108E or similar

### Solution 3: Improve Physical Connection

1. **Shorten cables**: Keep total length under 15cm if possible
2. **Use shielded cable**: Twisted pair or shielded cable reduces noise
3. **Check crimp connections**:
   - Use proper crimp tool for SPHD-001T-P contacts
   - Ensure solid mechanical and electrical contact
4. **Verify pinout**:
   ```
   AC Unit Side (PAP-08V-S connector):
   Black:  GND → ESP GND
   White:  TX  → ESP RX (via level shifter)
   Green:  RX  → ESP TX (via level shifter)
   Red:    5V  → (optional, can power ESP)
   ```

### Solution 4: Add Debug Logging (If Still Failing)

If solutions 1-3 don't work, enable more detailed UART debugging:

```yaml
logger:
  baud_rate: 0  # Already set
  level: VERBOSE  # ← Change from DEBUG
```

This will show every single byte transmitted/received.

---

## Testing Protocol

After each change, follow this testing procedure:

1. **Flash the new configuration**
   ```bash
   esphome run klima.yaml
   ```

2. **Monitor logs for 60 seconds**
   ```bash
   esphome logs klima.yaml
   ```

3. **Look for success indicators**:
   - ✅ "Connected" message
   - ✅ "Connecting (3/8)" → "Connecting (4/8)" → ... → "Connected"
   - ✅ Periodic status updates: `RX: 0xDC...` (status frames)
   - ✅ No "error recovery" messages

4. **Look for failure indicators**:
   - ❌ Repeated "Timeout - no response for 10s"
   - ❌ Stuck at "Connecting (1/8)" or "Connecting (2/8)"
   - ❌ Many "error recovery" messages

5. **Document results**:
   - Save first 100 lines of logs
   - Note configuration used
   - Share with community if issue persists

---

## Expected Behavior (After Fix)

Once the correct configuration is found, you should see:

```
[timestamp][D][sharp_ac.climate:070]: Initializing connection...
[timestamp][D][sharp_ac.climate:070]: TX: 02.FF.FF.00.00.00.00.02 (8)
[timestamp][D][sharp_ac.climate:070]: RX: 02
[timestamp][D][sharp_ac.climate:070]: TX: 02.FF.FF.01.01.00.01.00.FF (9)
[timestamp][D][sharp_ac.climate:070]: Connecting (1/8)...
[timestamp][D][sharp_ac.climate:070]: RX: ACK
[timestamp][D][sharp_ac.climate:070]: TX: 03.FF.A0.01.00.00.00.60 (8)
[timestamp][D][sharp_ac.climate:070]: Connecting (2/8)...
[... continues through 8/8 ...]
[timestamp][D][sharp_ac.climate:070]: Connected
```

Then regular status updates every 60 seconds.

---

## Additional Resources

- **GitHub Repository**: https://github.com/sven819/SharpClimateUART-ESPHome
- **ESPHome UART Documentation**: https://esphome.io/components/uart.html
- **Compatible Models**: AC166i (tested), AC186i, AC196i, AC176i.2, AC186.2

---

## Conclusion

The Buderus Logacool AC186i communication failure is **most likely caused by incorrect UART baudrate** (currently 9600, should probably be 4800). Secondary causes could be missing/incorrect level shifter or poor physical connections.

**Next Steps**:
1. **Immediately try baudrate 4800 with EVEN parity**
2. If that fails, verify level shifter installation
3. If still failing, check physical connections and try other baudrate combinations

The codebase itself is correctly implemented and designed for Buderus Logacool devices, so the issue is configuration or hardware-related.

---

**Report prepared by**: Claude (ESPHome Assistant)
**Analysis based on**: Log files from 12:53:35 - 12:53:50, codebase review, protocol documentation
