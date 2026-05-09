# ⚡ PLATO Vessel Rapid Prototype


## Meta

**Domain:** ai-agents
**Depends on:** —
**Depended by:** —
**Implements:** Product developer iteration loop: describe a project → get BOM, wiring diagram, ...
**Related:** —


**Describe a project. Get a BOM, wiring diagram, and simulation. Every revision is a PLATO room.**

For product developers who iterate fast. You think in features and tradeoffs — the agent handles BOM generation, vendor lookup, wiring validation, and version control for hardware.

```
┌──────────────────────────────────────────────────┐
│         PLATO VESSEL RAPID PROTOTYPE             │
│                                                  │
│  Describe concept →                              │
│  Agent generates BOM →                           │
│  Agent validates spec →                          │
│  Agent generates wiring diagram →                │
│  Build prototype →                               │
│  Change design →                                 │
│  Agent re-evaluates → new BOM → new wiring →     │
│  Every revision = new PLATO room                 │
└──────────────────────────────────────────────────┘
```

## Why This Exists

Hardware iteration is slow. Changing a component means:
1. Check if the new part is compatible
2. Recalculate power budget
3. Rewire the breadboard
4. Update the BOM
5. Find a supplier
6. Update the documentation

With Rapid Prototype, you change the description and the agent does steps 1-6. You keep building.

## Key Features

### 🧾 BOM Generation
Describe your project → agent reads it → generates a bill of materials with:
- Part numbers and quantities
- Links to suppliers (DigiKey, Mouser, Amazon, AliExpress)
- Alternate parts for cost/supply optimization
- Estimated total cost

### 🔌 Wiring Validation
Before you power on, the agent checks:
- Are all pins connected correctly?
- Any voltage mismatches (5V sensor → 3.3V GPIO)?
- Current limits respected?
- Missing pull-up/down resistors?

### 🔄 Version Control for Hardware
Every prototype revision is a PLATO room:
- **Room v1:** "Motor spins when button pressed"
- **Room v2:** "Added MOSFET for speed control"
- **Room v3:** "Replaced BJT with MOSFET, added flyback diode"

Each room has tiles for: what changed, what broke, what worked, what the new BOM looks like.

### 📋 PDF Export
Generate printable documents from any prototype room:
- **Wiring diagram** (ASCII + component placement)
- **Assembly instructions** (step-by-step)
- **Bill of Materials** (with supplier links)
- **Design decision log** (why each component was chosen)

### 🔍 Vendor Lookup
Describe what you need → agent finds the right part:
```
"What motor controller do I need for a 12V 3A solenoid?"
→ "L298N: $8, handles 2A per channel, needs heatsink above 1A"
→ "TB6612: $5, handles 1.2A per channel, smaller package"
→ "MOSFET IRF520: $2, needs external flyback diode"
```

## Repository Contents

| File | Purpose |
|------|---------|
| `VENDOR-DATABASE.md` | How the vendor lookup works — schema, sources, fallback |
| `PROTOTYPE-WORKFLOW.md` | Step-by-step iteration process from concept to prototype |

## Quick Start

```bash
# Describe your project
python3 scripts/agent.py describe "ESP32 temperature logger with SD card"

# Get a BOM
python3 scripts/agent.py bom --project temp-logger-v1

# Generate wiring diagram
python3 scripts/agent.py wiring --project temp-logger-v1 --format ascii

# Export printable PDF
python3 scripts/agent.py export --project temp-logger-v1 --format pdf

# Create next revision
python3 scripts/agent.py iterate --from temp-logger-v1 --change "replace SD with SPI flash"
```

## License

AGPL-3.0. See LICENSE file.
