# Fronius Solar Curtailment Blueprint - Installation Guide

## Overview
This blueprint system automatically throttles your Fronius inverter during negative grid prices to prevent exporting power at a loss while maintaining battery charging and house load coverage.

## What You'll Need

### Required Sensors
- **Grid Sell Price Sensor** - Shows current sell price (negative during curtailment periods)
- **Battery Level Sensor** - Battery state of charge percentage
- **House Load Power Sensor** - Current house power consumption in Watts
- **Export Power Sensor** - Current grid export power in Watts

### Required Configuration
- Fronius inverter with Modbus TCP enabled
- Home Assistant with Modbus integration

---

## Step 1: Create Input Boolean Helpers

Go to **Settings → Devices & Services → Helpers** and create these two helpers:

### Helper 1: Solar Curtailment Active
- **Type:** Toggle
- **Name:** Solar Curtailment Active
- **Icon:** mdi:solar-power-variant
- **Entity ID:** `input_boolean.solar_curtailment_active`

### Helper 2: Fronius Manual Override
- **Type:** Toggle
- **Name:** Fronius Manual Override
- **Icon:** mdi:toggle-switch-off-outline
- **Entity ID:** `input_boolean.fronius_manual_override`

---

## Step 2: Configure Modbus Integration

Add this to your `configuration.yaml` or create a file `integrations/fronius_modbus.yaml`:

```yaml
# Fronius Modbus Control Configuration
modbus:
  - name: mb_fronius_control
    type: tcp
    host: YOUR_INVERTER_IP  # e.g., 10.1.1.181
    port: 502
    
    # Timing configuration to prevent communication errors
    timeout: 5
    delay: 100                      # Wait 100ms between packets
    message_wait_milliseconds: 500  # Wait 500ms between commands
    
    # No sensors - write-only hub
    sensors: []
```

**Replace `YOUR_INVERTER_IP`** with your Fronius inverter's IP address.

**Restart Home Assistant** after adding this configuration.

---

## Step 3: Import the Blueprints

1. **Copy the blueprint file** to your Home Assistant blueprints folder:
   - Location: `config/blueprints/automation/fronius/`
   - Create the folders if they don't exist

2. **Or import via URL** (if hosted on GitHub):
   - Go to **Settings → Automations & Scenes → Blueprints**
   - Click **Import Blueprint**
   - Paste the blueprint URL

3. **You'll import FOUR blueprints:**
   - Main Dynamic Price Control
   - 4PM Restoration
   - Manual Override
   - Morning Reset

---

## Step 4: Create Automations from Blueprints

### Automation 1: Main Dynamic Price Control

Go to **Settings → Automations & Scenes → Create Automation → Start with a Blueprint**

Select: **Fronius Solar Curtailment - Dynamic Price Control**

**Configure these fields:**

#### Modbus Configuration
- **Modbus Hub Name:** `mb_fronius_control` (or your hub name)
- **Modbus Unit ID:** `1` (default for Fronius)

#### Inverter Specifications
- **Inverter Maximum Power:** Enter your inverter's max output in Watts
  - Example: 8200W for an 8.2kW inverter
  - Common values: 5000W, 8200W, 10000W, 15000W
- **Battery Charge Rate:** Desired charging power during curtailment
  - Example: 4500W for a 4.5kW charge rate
  - Adjust based on your battery capacity
- **Safety Buffer:** 500W (recommended)
- **Minimum Generation:** 500W (recommended)

#### Sensor Entities
- **Grid Sell Price Sensor:** Select your price sensor
- **Battery Level Sensor:** Select your battery SOC sensor
- **House Load Power Sensor:** Select your load sensor
- **Export Power Sensor:** Select your export sensor

#### Helper Entities
- **Curtailment Active Switch:** `input_boolean.solar_curtailment_active`
- **Manual Override Switch:** `input_boolean.fronius_manual_override`

#### Time Settings
- **Curtailment End Time:** 16:00:00 (4PM)
- **Daily Reset Time:** 06:00:00 (6AM)
- **Update Interval:** 5 minutes

**Save the automation**

---

### Automation 2: 4PM Restoration

Create from blueprint: **Fronius Solar Curtailment - 4PM Restoration**

**Configure:**
- Use the **same values** as the main automation for:
  - Modbus Hub Name
  - Modbus Unit ID
  - Inverter Maximum Power
  - Curtailment Active Switch
- **Restoration Time:** 16:00:00 (or your preferred time)

---

### Automation 3: Manual Override

Create from blueprint: **Fronius Solar Curtailment - Manual Override**

**Configure:**
- Use the **same values** as the main automation for:
  - Modbus Hub Name
  - Modbus Unit ID
  - Inverter Maximum Power
  - Curtailment Active Switch
  - Manual Override Switch

---

### Automation 4: Morning Reset

Create from blueprint: **Fronius Solar Curtailment - Morning Reset**

**Configure:**
- Use the **same values** as the main automation for:
  - Modbus Hub Name
  - Modbus Unit ID
  - Inverter Maximum Power
  - Curtailment Active Switch
- **Morning Reset Time:** 06:00:00 (or your preferred time)

---

## Step 5: Add to Dashboard (Optional)

Add these entities to your Lovelace dashboard for easy monitoring:

```yaml
type: entities
title: Solar Curtailment Control
entities:
  - entity: input_boolean.solar_curtailment_active
    name: Curtailment Status
  - entity: input_boolean.fronius_manual_override
    name: Manual Override
  - entity: sensor.grid_sell_price
    name: Grid Sell Price
  - entity: sensor.inverter_load_power
    name: House Load
  - entity: sensor.inverter_export_power
    name: Grid Export
```

---

## How It Works

### During Negative Grid Prices:
1. System calculates target generation = House Load + Battery Charge + Safety Buffer
2. Inverter is limited to this target (minimum 500W, maximum your inverter's rated capacity)
3. Updates every 5 minutes (or your configured interval)
4. Prevents exporting power when grid prices are negative

### During Normal/Positive Prices:
1. Inverter operates at full capacity
2. All curtailment limits are removed
3. Normal solar operation resumes

### Automatic Resets:
- **4PM:** Curtailment automatically disabled (end of typical negative pricing)
- **6AM:** Full system reset to prepare for the day
- **Manual Override:** Instant full power restoration via toggle switch

---

## Customization Examples

### Different Inverter Sizes

**5kW Fronius Inverter:**
- Inverter Maximum Power: `5000`
- Battery Charge Rate: `2500` (or based on your battery specs)

**10kW Fronius Inverter:**
- Inverter Maximum Power: `10000`
- Battery Charge Rate: `5000` (or based on your battery specs)

**15kW Fronius Inverter:**
- Inverter Maximum Power: `15000`
- Battery Charge Rate: `7000` (or based on your battery specs)

### Different Time Windows

**End curtailment at 3PM:**
- Curtailment End Time: `15:00:00`

**Start fresh at 5AM:**
- Daily Reset Time: `05:00:00`

### More Aggressive Curtailment

**Higher battery charge priority:**
- Battery Charge Rate: `6000` (charges faster during negative pricing)

**More conservative buffer:**
- Safety Buffer: `1000` (larger buffer to prevent any export)

### Less Aggressive Curtailment

**Lower battery charge:**
- Battery Charge Rate: `3000` (less aggressive charging)

**Tighter margins:**
- Safety Buffer: `200` (allows more generation)

---

## Troubleshooting

### Curtailment Not Activating
1. Check that grid price sensor shows negative values
2. Verify it's before 4PM
3. Ensure Manual Override is OFF
4. Check automation is enabled

### Inverter Not Responding
1. Verify Modbus TCP is enabled on inverter
2. Check IP address in modbus configuration
3. Test connectivity: `ping YOUR_INVERTER_IP`
4. Check Home Assistant logs for Modbus errors

### Too Much/Too Little Generation
1. Adjust Battery Charge Rate
2. Adjust Safety Buffer
3. Check House Load sensor accuracy
4. Verify Inverter Maximum Power is correct

### Manual Override Not Working
1. Check helper `input_boolean.fronius_manual_override` exists
2. Verify automation is enabled
3. Check Modbus hub name matches

---

## Support & Contribution

- **Issues:** Report on GitHub
- **Improvements:** Submit pull requests
- **Questions:** Home Assistant Community Forum

---

## Credits

Original automation designed for dynamic price-based solar curtailment with Fronius inverters.
Blueprint created to enable easy adoption by the Home Assistant community.

