# Vendor Database — How the Agent Finds Parts

## Overview

The Rapid Prototype agent maintains a structured vendor database. When you describe a component need, the agent queries this database with specs, returns matching parts with pricing and links, and can optimize for cost, availability, or lead time.

## Core Schema

Each vendor item is stored as a PLATO tile:

```json
{
  "domain": "vendor",
  "question": "L298N",
  "answer": {
    "name": "L298N Motor Driver",
    "category": "motor_controller",
    "subcategory": "h_bridge",
    "suppliers": [
      {
        "name": "DigiKey",
        "sku": "497-1445-ND",
        "price_usd": 7.95,
        "qty_available": 482,
        "lead_days": 2,
        "url": "https://www.digikey.com/..."
      },
      {
        "name": "Amazon",
        "sku": "B01M0F8Q9K",
        "price_usd": 9.99,
        "qty_available": 15000,
        "lead_days": 1,
        "url": "https://www.amazon.com/dp/B01M0F8Q9K"
      }
    ],
    "specs": {
      "voltage_min": 5,
      "voltage_max": 35,
      "current_per_channel": 2,
      "logic_voltage": 5,
      "package": "module",
      "interface": "step_dir",
      "features": ["heatsink_required_above_1A", "flyback_diodes_builtin"]
    },
    "alternatives": ["TB6612", "L293D", "IRF520_MOSFET"],
    "notes": "Popular for beginner robotics. Heavy — not for drones."
  }
}
```

## Query Examples

### By Spec

```
"Find a 12V 3A motor controller"
→ Agent searches: category=motor_controller, voltage_min<=12, voltage_max>=12, current_per_channel>=3
→ Returns: [L298N ($7.95, 2A/ch — needs dual), BTS7960 ($12.50, 43A), VNH2SP30 ($14.00, 30A)]
```

### By Category + Constraint

```
"Smallest GPS module under $15"
→ Agent searches: category=gps, price_usd<15, sort_by=physical_size
→ Returns: [NEO-6M ($8.00, 25×25mm), SAM-M8Q ($13.50, 18×18mm)]
```

### Alternate Search

```
"Cheaper alternative to DHT22"
→ Agent checks: category=temp_humidity_sensor, alternative_of=DHT22
→ Returns: [DHT11 ($2.50, less accurate), SHT30 ($4.00, more accurate, same price)]
```

## Data Sources

The vendor database is populated from:

| Source | Update Frequency | Coverage |
|--------|-----------------|----------|
| **DigiKey API** | Daily (agent poll) | 2M+ parts, reliable pricing |
| **Mouser API** | Daily | 1M+ parts, good for passives |
| **AliExpress scraping** | Weekly | Cheap parts, variable quality |
| **Amazon Product API** | Weekly | Consumer-accessible parts |
| **Manual entries** | As added by users | Custom/specialty parts |

The agent falls back between sources. If DigiKey is down, it uses Mouser.

## Adding Custom Parts

Users can add parts to the database:

```bash
python3 scripts/vendor_add.py --name "My Custom Sensor" \
  --category "temperature" \
  --specs '{"voltage_min": 3.3, "interface": "i2c"}' \
  --supplier "direct" --price 12.00
```

The part is added as a PLATO tile and becomes searchable immediately.

## Database Expansion

The agent automatically expands the database by:

1. **Watching prototype rooms** — if 3+ users mention the same unknown part, the agent researches and adds it
2. **Cross-referencing BOMs** — parts listed together in BOMs are tagged as "frequently paired"
3. **Price alerts** — agent rechecks prices weekly and updates tiles. If a part drops 20%+, the agent alerts affected prototype owners.
4. **End-of-life detection** — DigiKey/Mouser mark parts as NRND (Not Recommended for New Designs). Agent flags any prototype using that part and suggests alternatives.

## Vendor Database Rooms

```
room: "vendor-database"
├── motor_controller/    — all motor controller parts
├── sensor/              — temp, humidity, pressure, distance
├── display/             — OLED, LCD, e-ink, TFT
├── power/               — regulators, converters, battery protection
├── wireless/            — WiFi, BLE, LoRa, cellular
├── passive/             — resistors, capacitors, inductors
├── active/              — transistors, op-amps, logic gates
├── electromechanical/   — relays, solenoids, motors, fans
└── fasteners/           — screws, standoffs, connectors, wire
```

Each category room has a "top_picks" tile — the agent's most-recommended parts based on cost + reliability + availability.
