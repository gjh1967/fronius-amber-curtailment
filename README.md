# Fronius Solar Curtailment for Amber Electric

**Automatically limit solar generation during negative pricing periods to avoid export charges**

Compatible with Fronius Primo, Symo, Gen24, and other SunSpec-compliant inverters.

---

## 🎯 What Does This Do?

This automation monitors Amber Electric grid prices and automatically limits your Fronius solar inverter output when prices go negative. It calculates the optimal generation level to:

- ✅ Cover your house load
- ✅ Charge your battery (if you have one)
- ✅ Add a safety buffer
- ✅ Prevent exporting to grid during negative pricing

When prices return positive, it automatically restores full solar generation.

---

## 📋 Requirements

### Hardware
- Fronius solar inverter (Primo, Symo, Gen24, or any SunSpec-compliant model)
- Fronius inverter must have **Modbus TCP enabled** (see setup guide below)
- (Optional) Battery system for charging

### Software
- Home Assistant (2024.1 or newer recommended)
- [Amber Electric integration](https://www.home-assistant.io/integrations/amberelectric/) installed and configured
- Home Assistant **Modbus integration** configured

### Sensors Required
You need these sensors in Home Assistant:
- Grid sell price (from Amber integration)
- House load power (watts)
- Solar generation power (watts)
- Grid export power (watts)
- Battery state of charge (% - optional if no battery)

---

## 🔧 Installation

### Step 1: Enable Fronius Modbus

#### For Fronius Primo/Symo (Older Models):
1. Connect to your Fronius inverter web interface (usually `http://fronius-inverter-ip`)
2. Go to **Communication → Modbus**
3. Enable **Modbus TCP**
4. Set Port: `502` (default)
5. Note the Unit ID (usually `1`)
6. Save and reboot inverter

#### For Fronius Gen24:
1. Access inverter web interface
2. Go to **Communication → Modbus**
3. Enable **SunSpec Model**
4. Enable **Modbus TCP**
5. Port: `502`
6. Save changes

**Important:** Write down:
- Inverter IP address
- Modbus Unit ID (usually 1)
- Modbus Port (usually 502)

---

### Step 2: Configure Home Assistant Modbus

Add this to your `configuration.yaml`:

```yaml
modbus:
  - name: fronius_control
    type: tcp
    host: 192.168.1.XXX  # CHANGE: Your Fronius IP address
    port: 502
    delay: 1
    timeout: 5
```

**Restart Home Assistant** after adding this.

---

### Step 3: Create Helper Entities

Go to **Settings → Devices & Services → Helpers**

Create two **Toggle** helpers:
1. **Name:** `Solar Curtailment Active`
   - **Entity ID:** `input_boolean.solar_curtailment_active`
   - **Icon:** `mdi:solar-power-variant`

2. **Name:** `Fronius Manual Override`
   - **Entity ID:** `input_boolean.fronius_manual_override`
   - **Icon:** `mdi:toggle-switch-off-outline`

---

### Step 4: Install the Blueprint

#### Method A: Direct Import (Easiest)
1. Click this button:
   [![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/gjh1967/fronius-amber-curtailment/blob/main/blueprint.yaml)

2. Or manually:
   - Go to **Settings → Automations & Scenes → Blueprints**
   - Click **Import Blueprint**
   - Paste URL: `https://github.com/gjh1967/fronius-amber-curtailment/blob/main/blueprint.yaml`

#### Method B: Manual Installation
1. Download `fronius_amber_blueprint.yaml`
2. Copy to `/config/blueprints/automation/fronius/`
3. Restart Home Assistant
4. The blueprint will appear in **Settings → Automations & Scenes → Blueprints**

---

### Step 5: Create Automation from Blueprint

1. Go to **Settings → Automations & Scenes**
2. Click **Create Automation**
3. Select **Fronius Solar Curtailment for Amber Electric**
4. Fill in the configuration (see detailed guide below)
5. Click **Save**

---

## ⚙️ Configuration Guide

### Sensor Configuration

#### Grid Sell Price Sensor
- **What:** Your Amber Electric price sensor
- **Example:** `sensor.amber_general_price` or `sensor.amber_feed_in_price`
- **Must show:** Negative values during negative pricing (e.g., -0.03 for -3c/kWh)

#### House Load Power Sensor
- **What:** Current house power consumption
- **Unit:** Watts (W)
- **Example:** `sensor.house_consumption` or `sensor.inverter_load_power`

#### Battery Level Sensor (if applicable)
- **What:** Battery state of charge
- **Unit:** Percentage (0-100%)
- **Example:** `sensor.battery_soc` or `sensor.powerwall_charge`

#### Export Power Sensor
- **What:** Power being exported to grid
- **Unit:** Watts (W)
- **Example:** `sensor.grid_export` or `sensor.meter_export_power`

---

### Fronius Inverter Settings

#### Modbus Hub Name
- **What:** Name you gave the Modbus connection in Step 2
- **Default:** `fronius_control`
- **Find it:** Check your `configuration.yaml` under `modbus:` → `name:`

#### Modbus Unit ID
- **Default:** `1`
- **Range:** 1-247
- **Where to find:**
  1. Fronius web interface → Communication → Modbus
  2. Look for "Unit ID" or "Slave ID"
  3. Usually `1` unless you have multiple devices

#### SunSpec Power Limit Register Address
**This is CRITICAL - wrong address won't work!**

| Inverter Model | Recommended Address | How to Verify |
|----------------|---------------------|---------------|
| Fronius Primo 3.0-8.2 | 40232 | Check Fronius Modbus docs |
| Fronius Symo 3.0-20.0 | 40232 | Same as Primo |
| Fronius Gen24 Plus | 40232 | Usually same |
| Fronius Symo Hybrid | 40232 | Check documentation |

**How to find your address:**
1. Download Fronius Modbus specification from [Fronius Solar Downloads](https://www.fronius.com/en/solar-energy/installers-partners/service-support/software)
2. Look for "SunSpec Model 123" or "Immediate Controls"
3. Find "WMaxLim" or "Maximum Power Limit" register
4. Use the register address (e.g., 40232)

**If unsure:** Try 40232 first - it works for 90% of Fronius inverters.

#### Inverter Maximum AC Output

**IMPORTANT:** This must match your inverter's nameplate rating!

| Model | Max Output |
|-------|-----------|
| Fronius Primo 3.0-1 | 3000 W |
| Fronius Primo 5.0-1 | 5000 W |
| Fronius Primo 6.0-1 | 6000 W |
| Fronius Primo 8.2-1 | 8200 W |
| Fronius Symo 10.0-3 | 10000 W |
| Fronius Symo 15.0-3 | 15000 W |
| Fronius Symo 20.0-3 | 20000 W |
| Fronius Gen24 Plus 6.0 | 6000 W |
| Fronius Gen24 Plus 10.0 | 10000 W |

**Find yours:**
- Check inverter nameplate label
- Fronius Solar.web app → Device info
- Inverter web interface → Status

---

### Battery & Charging Settings

#### Target Battery Charge Power
- **What:** How fast you want to charge battery during negative pricing
- **Default:** 4500W
- **Set to 0** if you don't have a battery
- **Common values:**
  - Alpha ESS: 4500-5000W
  - Tesla Powerwall 2: 5000W
  - BYD Battery-Box HVS: 2500W
  - BYD Battery-Box Premium HVM: 5000W
  - Pylontech US2000/US3000: 2500W

#### Safety Buffer
- **What:** Extra power buffer to prevent grid import during fluctuations
- **Default:** 500W
- **Recommended:**
  - Stable house load: 300-500W
  - Variable house load (lots of appliances): 700-1000W
  - Very stable (minimal load changes): 200W

#### Minimum Solar Generation
- **What:** Absolute minimum generation (safety feature)
- **Default:** 500W
- **Purpose:** Prevents complete inverter shutdown
- **Recommended:** 300-1000W depending on your system

---

### Timing Settings

#### Curtailment End Time
- **What:** Time to automatically restore full solar
- **Default:** 16:00:00 (4pm)
- **Why:** Amber negative pricing typically ends 3-4pm
- **Adjust:** Check your Amber app for when negative pricing usually ends in your region

#### Manual Override Reset Time
- **What:** When to auto-reset the emergency override
- **Default:** 06:00:00 (6am)
- **Why:** Ensures automation is ready for next day
- **Adjust:** Set to before your typical negative pricing starts

---

## 📊 Dashboard Setup

### Install Dashboard

1. Download `fronius_amber_dashboard.yaml`
2. **Settings → Dashboards → Add Dashboard**
3. Create new dashboard: "Solar Curtailment"
4. Click three dots → **Edit Dashboard**
5. Click three dots again → **Raw configuration editor**
6. Paste the dashboard YAML
7. **Customize all sensor entity IDs** (marked with `# CHANGE THIS`)
8. Save

### Customize Dashboard

Search for all `# CHANGE THIS` comments and replace with your entities:

```yaml
# Example replacements:
sensor.amber_general_price → sensor.your_amber_price
sensor.inverter_pv_power → sensor.your_solar_power
sensor.inverter_load_power → sensor.your_house_load
```

Also update hardcoded values in the "Live Calculation Debug" card:
- Change `4500` to your battery charge target
- Change `500` to your safety buffer
- Change `8200` to your inverter max output
- Change `16` to your curtailment end hour

---

## 🧪 Testing

### Test 1: Manual Override (Safe Test)

1. Turn ON **Fronius Manual Override** switch
2. Verify:
   - ✅ Curtailment Active turns OFF
   - ✅ Solar generation returns to normal
3. Turn OFF manual override

### Test 2: Simulate Negative Pricing

**Option A: Wait for real negative pricing**
- Check Amber app for next negative pricing period
- Monitor automation behavior

**Option B: Create test sensor (Advanced)**
1. Create template sensor with negative value
2. Point automation to test sensor
3. Watch it activate curtailment
4. Switch back to real sensor when done

### Test 3: Monitor First Real Run

When negative pricing occurs:
1. Watch **Live Calculation Debug** card
2. Check automation trace: Settings → Automations → (Your automation) → Traces
3. Verify Fronius generation drops to calculated target
4. Check grid export drops to near 0W
5. Confirm automation restores full power when price returns positive

---

## 🐛 Troubleshooting

### Automation Not Running

**Problem:** Automation doesn't activate during negative pricing

**Check:**
1. Manual Override is OFF
2. Current time is before curtailment end time (default 4pm)
3. Grid price sensor shows negative value (check Developer Tools → States)
4. Automation is enabled (Settings → Automations)

**Fix:**
- Check automation conditions in trace
- Verify sensor entities exist and update

---

### Fronius Not Limiting

**Problem:** Automation runs but inverter doesn't limit

**Check:**
1. Modbus connection works:
   - Go to Settings → Integrations → Modbus
   - Should show "Connected"
2. Register address is correct
3. Unit ID is correct
4. Inverter has Modbus TCP enabled

**Test Modbus:**
```yaml
# Add temporary script to test
script:
  test_fronius_limit:
    sequence:
      - service: modbus.write_register
        data:
          hub: fronius_control  # Your hub name
          unit: 1  # Your unit ID
          address: 40232  # Your register address
          value: [5000, 0, 0, 0, 0]  # Limit to ~4.1kW
```

Run script and check if inverter limits. If yes, problem is in automation logic. If no, problem is Modbus config.

---

### Wrong Calculation

**Problem:** Inverter limits too much or too little

**Check:**
1. House load sensor reading correctly
2. Battery charge target matches your battery
3. Inverter max output set correctly
4. Safety buffer appropriate for your system

**Debug:**
- Use "Live Calculation Debug" dashboard card
- Compare calculated target vs actual limit
- Check automation trace variables

---

### Sensor Issues

**Problem:** Sensors showing unavailable/unknown

**Fix:**
1. Verify Amber integration installed and authenticated
2. Check sensor names in Developer Tools → States
3. Ensure sensors update regularly (check Last Updated timestamp)
4. Restart integrations if needed

---

## 📈 Optimization Tips

### Minimize Export During Negative Pricing

**If still exporting:**
1. Increase safety buffer (try 700-1000W)
2. Reduce battery charge target slightly
3. Add manual load control (turn on swim spa, etc.)

### Maximize Solar Utilization

**If too much curtailment:**
1. Reduce safety buffer (try 300W)
2. Ensure battery charge target matches actual charge rate
3. Check minimum generation isn't too high

### Battery Charging Strategy

**Slow charging strategy** (prevent early full charge):
1. Set battery charge target to 2000-3000W
2. Battery reaches 100% closer to 4pm
3. Reduces hours of forced export

**Fast charging strategy:**
1. Set battery charge target to max (5000W+)
2. Fill battery quickly
3. Use manual loads (appliances) for excess

---

## 🔄 Updates & Maintenance

### Updating the Blueprint

1. Re-import blueprint URL
2. Home Assistant will detect changes
3. Existing automations inherit updates automatically
4. Review changelog for breaking changes

### Seasonal Adjustments

**Summer (longer days):**
- May need to extend curtailment end time to 5-6pm
- Amber negative pricing can last longer

**Winter (shorter days):**
- Curtailment end time 3-4pm usually sufficient
- Less total solar generation anyway

---

## 🤝 Contributing

Found a bug? Have an improvement?

**GitHub:** [gjh1967/fronius-amber-curtailment](https://github.com/gjh1967)

**Home Assistant Community:** [Forum Thread](https://community.home-assistant.io)

---

## 📄 License

MIT License - Free to use and modify

---

## ⚠️ Disclaimer

This automation directly controls your solar inverter. While tested extensively:

- ✅ Always test with manual override first
- ✅ Monitor first few days closely
- ✅ Have a way to manually restore full power
- ❌ Author not responsible for any issues or damages
- ❌ Use at your own risk
- ❌ Not affiliated with Fronius or Amber Electric

---

## 📞 Support- Best not ask for Ambers help.

**Before asking for help, please:**
1. Read this entire guide
2. Check troubleshooting section
3. Review automation traces (Settings → Automations → Traces)
4. Provide logs and configuration when seeking help

**Get help:**
- Home Assistant Community Forum
- GitHub Issues
- r/homeassistant on Reddit

---

## 🙏 Credits

Created by the Home Assistant community

Special thanks to:
- Fronius for SunSpec Modbus documentation
- Amber Electric for dynamic pricing
- Home Assistant developers

**If this helped you save money on negative pricing, consider:**
- ⭐ Star the GitHub repo
- 💬 Share your experience in the forum
- ☕ Buy me a coffee (optional!)

---

**Version:** 1.0.0  
**Last Updated:** December 2024  
**Compatibility:** Home Assistant 2024.1+

