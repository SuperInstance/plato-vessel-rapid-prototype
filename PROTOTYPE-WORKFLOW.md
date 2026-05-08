# Prototype Workflow — Concept to Revision N

## The Iteration Loop

```
┌─────────────────────────────┐
│  1. DESCRIBE                │
│  "ESP32 temp logger to SD"  │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  2. AGENT ANALYZES          │
│  Generates system diagram,  │
│  power budget, constraints  │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  3. BOM GENERATION          │
│  Parts list with supplier   │
│  links and alternatives     │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  4. WIRING DIAGRAM          │
│  ASCII schematic + pin map  │
│  + validation checks        │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  5. BUILD & TEST            │
│  (you do this part)         │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  6. DOCUMENT RESULTS        │
│  Agent logs what worked,    │
│  what broke, measurements   │
└─────────────┬───────────────┘
              ▼
┌─────────────────────────────┐
│  7. ITERATE                 │
│  "Replace SD with SPI flash │
│  — same pinout"             │
│  → back to step 2           │
└─────────────────────────────┘
```

## Step 1: Describe

Give the agent a natural-language description of your project. Be as specific or as vague as you want:

```bash
# Vague — agent fills in defaults and asks clarifying questions
> "I want a temperature logger"

# Specific — agent skips to BOM generation
> "ESP32-based temperature logger with DS18B20, logging to SD card over SPI,
   powered by 18650 battery, sending data to MQTT broker at 10.0.0.1:1883"
```

The agent responds with a summary interpretation:
```
📋 PROJECT: Temperature Logger v1

System diagram:
  ESP32 ──┬── DS18B20 (GPIO4, one-wire)
          ├── SD Card (SPI, CS=GPIO5)
          ├── MQTT via WiFi (broker: 10.0.0.1:1883)
          └── Power: 18650 → 3.3V regulator (AMS1117-3.3)

Power budget:
  ESP32 @ 80MHz:      75mA
  DS18B20 active:     1mA
  SD card write:     100mA (peak)
  WiFi TX:           180mA (peak)
  Total idle:         76mA
  Total peak:        356mA
  18650 (2500mAh):   ~7h continuous logging

Estimated build time: 2-3 hours
Estimated cost:      $18-25 (without enclosure)
```

## Step 2: Agent Analyzes

The agent validates your description for:
- **Compatibility:** DS18B20 needs 3.3V-5V — ESP32 GPIO is 3.3V ✅
- **Pin conflicts:** GPIO4 is fine for one-wire, but if you also have an I²C display on GPIO4 → conflict ❌
- **Power feasibility:** 18650 → 3.3V regulator: AMS1117 needs 1V dropout, 18650 at 3.7V nominal → 2.7V minimum before brownout ⚠️
- **Protocol matching:** MQTT over WiFi needs ESP32 WiFi stack — confirmed ✅

## Step 3: BOM Generation

The agent generates a bill of materials with suppliers:

```
🛒 BILL OF MATERIALS — Temperature Logger v1

| Component         | Part              | Qty | Supplier   | Price  | Link |
|-------------------|-------------------|-----|------------|--------|------|
| Microcontroller   | ESP32 Dev Board   | 1   | DigiKey    | $5.50  | ...  |
| Temp sensor       | DS18B20 TO-92     | 1   | Mouser     | $3.25  | ...  |
| SD card module    | MicroSD SPI       | 1   | Amazon     | $2.50  | ...  |
| Voltage regulator | AMS1117-3.3       | 1   | DigiKey    | $0.85  | ...  |
| Battery           | 18650 2500mAh     | 1   | Amazon     | $6.00  | ...  |
| Resistor          | 4.7kΩ (pull-up)   | 1   | DigiKey    | $0.10  | ...  |
| Resistor          | 10kΩ (divider)    | 2   | DigiKey    | $0.20  | ...  |
| Capacitor         | 10µF ceramic      | 2   | Mouser     | $0.30  | ...  |
| Protoboard        | 5×7cm perfboard   | 1   | Amazon     | $1.50  | ...  |
| Header pins       | 2.54mm male       | 10  | DigiKey    | $0.50  | ...  |
|                   |                   |     | TOTAL      | $20.70 |      |

📦 All parts available with Prime shipping: $24.19
🎯 Cheapest alternatives (different suppliers): $16.45
```

## Step 4: Wiring Diagram

The agent generates an ASCII wiring diagram:

```
ESP32 TEMPERATURE LOGGER v1
============================

Power:
  18650 (+) ─── AMS1117(Vin) ─── AMS1117(Vout) ─── ESP32(3.3V)
     │               │                  │
     │             10µF                10µF
     │               │                  │
  18650 (-) ─── AMS1117(GND) ── GND ── ESP32(GND)

DS18B20:
  Pin 1 (GND) ─── GND
  Pin 2 (DQ)  ─── 4.7kΩ ─── ESP32(GPIO4)
               ─── 4.7kΩ ─── 3.3V
  Pin 3 (VDD) ─── 3.3V

SD Card (SPI):
  CS   ─── ESP32(GPIO5)
  MOSI ─── ESP32(GPIO23)
  MISO ─── ESP32(GPIO19)
  SCK  ─── ESP32(GPIO18)
  VCC  ─── 3.3V
  GND  ─── GND

⚠️  VALIDATION: No conflicts detected
   DS18B20 pull-up: 4.7kΩ (correct for short cables <10m)
   SD card CS: GPIO5 (default SPI SS, OK)
   Power: AMS1117 dropout=1V, battery min=3.3+1=4.3V before regulation fails
   → RECOMMEND: Use low-dropout regulator (MCP1700, dropout=0.4V)
```

## Step 5: Build & Test

You build the prototype. The agent doesn't do this part.

While you build, the agent suggests test points:
```
🔍 Build checklist:
  [ ] Power: measure AMS1117 output = 3.35V (±0.05V)
  [ ] DS18B20: read 0x28 ROM code on GPIO4
  [ ] SD card: init at SPI 1MHz, check CS response
  [ ] WiFi: scan for networks, connect to broker
  [ ] Full system: read temp → write to SD → publish MQTT
```

## Step 6: Document Results

When your prototype works (or fails), tell the agent:

```bash
> "The DS18B20 reading is stable but the SD card won't init at full speed"
```

The agent creates a diagnostic tile:
```
📊 DEBUG: SD Card Init Failure
──────────────────────────────
Symptom: SD card init fails at 10MHz SPI
Passes at 1MHz SPI

Likely causes:
1. Long wires (>10cm between ESP32 and SD module)
2. No series resistor on MOSI (ESP32 3.3V → SD 3.3V OK)
3. Missing 10kΩ pull-up on MISO

Suggested fix:
  Add 22Ω series resistor on MOSI, MISO, SCK lines
  Reduce SPI speed to 4MHz (fast enough for logging)
  Keep wires under 10cm

→ Next: "Add series resistors and retry"
```

## Step 7: Iterate

```bash
> "Replace SD card with SPI flash (W25Q64). Same pinout."

# Agent generates v2:
# - Removes SD card from BOM
# - Adds W25Q64 flash chip ($1.20)
# - Updates wiring diagram
# - Creates new PLATO room "temp-logger-v2"
# - Tags changes in v2/v1 diff tile
```

The previous revision is preserved in its own room. You can always go back.

## Example Session

```
> "Motion-activated security camera with ESP32-CAM"

📋 System: ESP32-CAM + PIR motion sensor + IR LED + MQTT notification
💰 BOM: $28.50 (ESP32-CAM $15, PIR $2, IR LED $1, power $8, misc $2.50)

> "Add battery backup"

📋 Update: Add TP4056 charger + 18650
💰 New BOM: $35.00
⚠️  Power note: ESP32-CAM + WiFi draws 300mA — 18650 lasts ~8h
   -> Need larger battery or solar charging for 24/7 operation

> "Add solar charging"

📋 Update: CN3791 MPPT charger + 5W solar panel
💰 New BOM: $48.50
✅ Runtime: Solar charges battery during day, battery runs camera at night
   Estimated: indefinite operation in sunny climates

> "Export PDF"

📄 PDF generated: security-cam-v3-spec.pdf (4 pages)
   - BOM with supplier links
   - Wiring diagram
   - Power budget
   - Iteration log (v1 → v2 → v3)
```

## Workflow Tips

- **Say what broke** — the agent can only diagnose what you tell it
- **Ask for alternatives** — "What's a cheaper temp sensor?" or "Any SD card module without voltage regulator?"
- **Save every revision** — each room is a checkpoint. You can diff any two rooms to see exactly what changed
- **Export before cutting PCB** — the PDF export includes all validation checks
- **Share your room** — send another builder your PLATO room URL. They see every decision, every component
