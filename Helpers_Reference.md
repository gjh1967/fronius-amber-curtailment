# Required Helpers - Quick Reference

## Create These in Home Assistant

Go to: **Settings → Devices & Services → Helpers → Create Helper → Toggle**

---

## Helper 1: Solar Curtailment Active

```yaml
Name: Solar Curtailment Active
Icon: mdi:solar-power-variant
Entity ID: input_boolean.solar_curtailment_active
```

**Purpose:** Tracks whether solar curtailment is currently active

**States:**
- ON = Curtailment is active (inverter is limited)
- OFF = Normal operation (inverter at full power)

---

## Helper 2: Fronius Manual Override

```yaml
Name: Fronius Manual Override
Icon: mdi:toggle-switch-off-outline
Entity ID: input_boolean.fronius_manual_override
```

**Purpose:** Emergency override to force full power immediately

**Usage:**
- Turn ON to immediately restore full power
- System will disable curtailment and reset inverter
- Leave OFF for normal automated operation

---

## YAML Configuration (Alternative Method)

If you prefer to create helpers via YAML, add this to your `configuration.yaml`:

```yaml
input_boolean:
  solar_curtailment_active:
    name: Solar Curtailment Active
    icon: mdi:solar-power-variant
  
  fronius_manual_override:
    name: Fronius Manual Override
    icon: mdi:toggle-switch-off-outline
```

Then restart Home Assistant.

---

## Dashboard Card Example

Add this to your dashboard to monitor both helpers:

```yaml
type: entities
title: Solar Curtailment Control
entities:
  - entity: input_boolean.solar_curtailment_active
    name: Curtailment Status
    icon: mdi:solar-power-variant
  - entity: input_boolean.fronius_manual_override
    name: Emergency Override
    icon: mdi:alert-octagon
    tap_action:
      action: toggle
show_header_toggle: false
```

---

## Quick Icons Reference

**Curtailment Active:**
- 🔆 mdi:solar-power-variant
- ⚡ mdi:lightning-bolt
- 📊 mdi:chart-line

**Manual Override:**
- 🔀 mdi:toggle-switch-off-outline
- 🛑 mdi:stop-circle
- ⚠️ mdi:alert-octagon
- 🔓 mdi:lock-open

---

## Automation Requirements

Both helpers **MUST** exist before creating the automations from blueprints.

The blueprint automations will:
- ✅ Read the helper states
- ✅ Turn helpers ON/OFF automatically
- ✅ Use helpers to coordinate between automations

Without these helpers, the automations will fail with entity not found errors.

