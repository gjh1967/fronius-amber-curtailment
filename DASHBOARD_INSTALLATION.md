# Dashboard Installation Guide

## Overview
This dashboard provides real-time monitoring and control of your Fronius solar curtailment system.

---

## Method 1: Quick Install (Recommended for Beginners)

### Step 1: Create a New Dashboard
1. Go to **Settings → Dashboards**
2. Click **Add Dashboard**
3. Choose **New dashboard from scratch**
4. Name it: "Solar Curtailment"
5. Icon: `mdi:solar-power-variant`

### Step 2: Add Manual Controls Card
1. Click **Add Card**
2. Select **Entities**
3. Configure:
   - **Title:** Manual Controls
   - **Entities:**
     - `input_boolean.fronius_manual_override`
       - Name: 🚨 MANUAL OVERRIDE
       - Show secondary info: Last changed
     - Add divider
     - `input_boolean.solar_curtailment_active`
       - Name: Curtailment Status
   - **Toggle:** Show state color = ON

### Step 3: Add Override Instructions Card
1. Click **Add Card**
2. Select **Markdown**
3. Configure:
   - **Title:** ⚠️ Override Instructions
   - **Content:**
```
**Turn ON Manual Override to:**
• Immediately restore inverter to full power
• Block all automatic curtailment
• Emergency failsafe if automation misbehaves

**Remember to turn OFF when issue is resolved!**
```

### Step 4: Add Current Status Card
1. Click **Add Card**
2. Select **Entities**
3. Configure:
   - **Title:** Current Status
   - **Entities:** (Replace with YOUR sensor names)
     - Your PV power sensor (e.g., `sensor.inverter_pv_power`)
     - Your load power sensor (e.g., `sensor.inverter_load_power`)
     - Your battery level sensor (e.g., `sensor.inverter_battery_level`)
     - Your grid power sensor (e.g., `sensor.smart_meter_real_power`)
     - Your grid price sensor (e.g., `sensor.grid_sell_price`)

### Step 5: Add Live Calculation Card
1. Click **Add Card**
2. Select **Markdown**
3. Configure:
   - **Title:** Live Calculation
   - **Content:** (Copy from template below and customize)

```yaml
**Time:** {{ now().strftime('%H:%M:%S') }}

**House Load:** {{ states('sensor.YOUR_LOAD_SENSOR') | int(0) }}W
**Battery Charge Target:** YOUR_BATTERY_CHARGE_RATE W
**Safety Buffer:** YOUR_SAFETY_BUFFER W
**Fronius Target:** {{ (states('sensor.YOUR_LOAD_SENSOR') | int(0) + YOUR_BATTERY_CHARGE_RATE + YOUR_SAFETY_BUFFER) }}W
**Max Inverter Power:** YOUR_INVERTER_MAX W

**Curtailment Allowed:** {% if now().hour < 16 and is_state('input_boolean.fronius_manual_override', 'off') %}✅ YES{% else %}❌ NO{% endif %}
**Override Active:** {% if is_state('input_boolean.fronius_manual_override', 'on') %}🚨 YES{% else %}✅ Normal{% endif %}
**Grid Price:** {{ states('sensor.YOUR_GRID_PRICE_SENSOR') }}
**Curtailment Status:** {% if is_state('input_boolean.solar_curtailment_active', 'on') %}🔴 ACTIVE{% else %}🟢 NORMAL{% endif %}
```

### Step 6: Add Automations Status Card
1. Click **Add Card**
2. Select **Entities**
3. Configure:
   - **Title:** Automations Status
   - **Entities:** (Your automation entity_ids)
     - Main curtailment automation
     - 4pm restoration automation
     - Morning reset automation
     - Manual override automation
   - **Toggle:** Show state color = ON

### Step 7: Add History Graph Card
1. Click **Add Card**
2. Select **History Graph**
3. Configure:
   - **Title:** Generation Limiting Effect - 24 Hours
   - **Hours to show:** 24
   - **Entities:** (Replace with YOUR sensors)
     - Your PV power sensor
     - Your grid power sensor
     - Your grid price sensor
     - `input_boolean.solar_curtailment_active`

---

## Method 2: YAML Installation (Advanced Users)

### Step 1: Edit Dashboard YAML
1. Go to your dashboard
2. Click the three dots (⋮) in top right
3. Select **Edit Dashboard**
4. Click three dots again → **Raw configuration editor**

### Step 2: Copy Template
1. Open `fronius_curtailment_dashboard.yaml`
2. Find the section that matches your needs:
   - **Template Version:** Has placeholders like `YOUR_LOAD_POWER_SENSOR`
   - **Example Version:** Has actual sensor names (use as reference)
   - **Mobile Version:** Simpler, compact layout
   - **Advanced Version:** Conditional cards that show/hide

### Step 3: Customize Template
Replace these placeholders with YOUR actual entity names:

**Sensors:**
- `sensor.YOUR_PV_POWER_SENSOR` → Your solar production sensor
- `sensor.YOUR_LOAD_POWER_SENSOR` → Your house load sensor
- `sensor.YOUR_BATTERY_LEVEL_SENSOR` → Your battery percentage sensor
- `sensor.YOUR_GRID_POWER_SENSOR` → Your grid import/export sensor
- `sensor.YOUR_GRID_PRICE_SENSOR` → Your grid sell price sensor

**Automations:**
- `automation.YOUR_MAIN_CURTAILMENT_AUTOMATION`
- `automation.YOUR_4PM_RESTORATION_AUTOMATION`
- `automation.YOUR_MORNING_RESET_AUTOMATION`
- `automation.YOUR_MANUAL_OVERRIDE_AUTOMATION`

**Constants:**
- `INVERTER_MAX_POWER` → Your inverter max (e.g., 8200)
- `BATTERY_CHARGE_RATE` → Your battery charge rate (e.g., 4500)
- `SAFETY_BUFFER` → Your safety buffer (e.g., 500)

### Step 4: Paste and Save
1. Paste your customized YAML into the raw editor
2. Click **Save**
3. Exit edit mode

---

## Finding Your Entity Names

### Method 1: Developer Tools
1. Go to **Developer Tools → States**
2. Search for your devices
3. Copy the `entity_id` values

### Method 2: Integration Page
1. Go to **Settings → Devices & Services**
2. Find your solar/energy integration
3. Click on the device
4. View all entities and their IDs

### Method 3: Existing Cards
1. If you already have cards showing this data
2. Edit the card
3. Look at the entity_id values

---

## Common Entity Patterns

**Fronius Inverters:**
- `sensor.fronius_inverter_ac_power`
- `sensor.fronius_battery_soc`
- `sensor.fronius_meter_power_real`

**SolarEdge:**
- `sensor.solaredge_current_power`
- `sensor.solaredge_battery_level`
- `sensor.solaredge_grid_power`

**Enphase:**
- `sensor.envoy_current_power_production`
- `sensor.envoy_current_power_consumption`

**Smart Meters:**
- `sensor.smart_meter_power`
- `sensor.shelly_em_power`

---

## Customization Examples

### Change Update Frequency
The markdown card updates every time Home Assistant refreshes. To force faster updates:
```yaml
- type: markdown
  content: >-
    {{ now().strftime('%H:%M:%S') }}
    ...your content...
  refresh: 10  # Update every 10 seconds
```

### Add Battery Percentage Bar
```yaml
- type: entity
  entity: sensor.YOUR_BATTERY_LEVEL_SENSOR
  name: Battery Level
  icon: mdi:battery
```

### Add Power Gauge
```yaml
- type: gauge
  entity: sensor.YOUR_PV_POWER_SENSOR
  name: Solar Production
  min: 0
  max: 8200  # Your inverter max
  severity:
    green: 0
    yellow: 4000
    red: 7000
```

### Add Energy Today
```yaml
- type: entity
  entity: sensor.YOUR_ENERGY_TODAY_SENSOR
  name: Energy Today
  icon: mdi:solar-power
```

---

## Mobile Optimization

For better mobile experience, use the **Compact Mobile-Friendly Version** from the YAML file:

**Key features:**
- Glance card for quick overview
- Large button for emergency override
- Minimal scrolling needed

---

## Troubleshooting

### Card Shows "Entity Unavailable"
- Check entity_id spelling
- Verify the sensor exists in Developer Tools → States
- Make sure integration is working

### Markdown Card Doesn't Update
- Ensure template syntax is correct
- Check for typos in `{{ states('...') }}`
- Verify entity names are correct

### History Graph is Empty
- Sensors need time to build history
- Check recorder configuration
- Verify entities are being recorded

### Automations Not Listed
- Wait for automations to appear after creating from blueprints
- Check **Settings → Automations & Scenes** for correct names
- Use Developer Tools → States to find exact entity_ids

---

## Recommended Dashboard Layout

**Order of cards for best user experience:**

1. **Manual Controls** - Most important, top position
2. **Override Instructions** - Keep safety info visible
3. **Current Status** - Real-time data
4. **Live Calculation** - Shows what automation is doing
5. **Automations Status** - Quick health check
6. **History Graph** - Performance over time
7. **Optional: Statistics/Energy cards** - Deep dive analysis

---

## Advanced: Multiple Views

Create tabs for different purposes:

**View 1: Control**
- Manual controls
- Current status
- Live calculation

**View 2: Monitor**
- History graphs
- Statistics
- Automations status

**View 3: Analysis**
- Long-term statistics
- Cost savings
- Export prevention metrics

---

## Next Steps

After dashboard is set up:
1. Test the manual override button
2. Watch the live calculation during the day
3. Monitor history graph during negative price periods
4. Verify automations trigger correctly
5. Adjust curtailment end time if needed

---

## Support

If you have issues:
1. Check entity names are correct
2. Verify sensors are working
3. Test with Developer Tools → Template
4. Share your YAML for community help

