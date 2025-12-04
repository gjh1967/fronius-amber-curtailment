Quick Start Guide - Fronius Solar Curtailment


5-minute setup for experienced Home Assistant users



✅ Prerequisites


Before starting, you must have:


[ ] Fronius inverter with network access
[ ] Amber Electric account
[ ] Home Assistant 2024.1+
[ ] Basic HA and YAML knowledge



🚀 Installation (6 Steps)


Step 1: Install Amber Integration


Settings → Devices & Services → Add Integration → "Amber Electric"
Enter API token from app.amber.com.au
Verify sensor.amber_feed_in_price exists (this shows negative values during negative pricing)



Important: In Amber app:


❌ Disable SmartShift
❌ Disable Automated Solar Curtailment



Step 2: Enable Fronius Modbus


Access Fronius web interface: http://YOUR_FRONIUS_IP
Go to: Communication → Modbus
Enable: Modbus TCP
Port: 502
Unit ID: 1 (default)
Save and note IP address




Step 3: Configure HA Modbus


Add to configuration.yaml:


modbus:
  - name: fronius_control
    type: tcp
    host: YOUR_FRONIUS_IP  # ← Change this
    port: 502
    delay: 1
    timeout: 5



Restart Home Assistant



Step 4: Create Helpers


Settings → Helpers → Create two Toggles:

1. Name: "Solar Curtailment Active"
   Icon: mdi:solar-power-variant
   
2. Name: "Fronius Manual Override"
   Icon: mdi:toggle-switch-off-outline



Verify they exist in Developer Tools → States.



Step 5: Import Blueprint


Click: 


Or manually:


Settings → Blueprints → Import Blueprint
URL: https://github.com/gjh1967/fronius-amber-curtailment/blob/main/blueprint.yaml




Step 6: Create Automation


Settings → Automations → Create Automation
Select: "Fronius Solar Curtailment for Amber Electric"



Configure these settings:




Setting
Enter
Notes




Grid Price Sensor
sensor.amber_feed_in_price
Shows negative during negative pricing


House Load Sensor
Your power sensor
In watts


Battery Level
Your battery SOC sensor
0-100% (or any if no battery)


Export Power
Your grid export sensor
In watts


Modbus Hub
fronius_control
From Step 3


Unit ID
1
Default


Register Address
40232
Verify for your model


Inverter Max Output
Your watts
8200 for Primo 8.2


Battery Charge Target
4500 or 0
0 if no battery


Safety Buffer
500
Increase if still exporting


Minimum Generation
500
Safety floor




Save automation



🧪 Quick Test


Test 1: Manual Override


1. Turn ON "Fronius Manual Override"
2. Verify solar returns to full power
3. Turn OFF override



✅ If this works, Modbus is configured correctly.


Test 2: Check Automation Trace


Settings → Automations → Your automation → Traces
Review latest run - any errors?



Test 3: Wait for Negative Pricing


Monitor during next negative pricing period:
- Does "Solar Curtailment Active" turn ON?
- Does Fronius generation drop?
- Does grid export drop to near 0W?




⚙️ Typical Settings (My System)


Verified working on:


Fronius Primo 8.2-1
Alpha ESS SMILE-S5 (20kWh)
Amber Electric NSW
House load: ~1.2kW average


My configuration:


Grid Price: sensor.amber_feed_in_price
House Load: sensor.inverter_load_power
Battery Level: sensor.inverter_battery_level
Export Power: sensor.inverter_export_power
Modbus Hub: fronius_control
Unit ID: 1
Register: 40232
Max Output: 8200W
Battery Target: 4500W
Safety Buffer: 500W
Min Generation: 500W



Result: Limits to ~6kW during negative pricing, near-zero export.



🐛 Quick Troubleshooting




Problem
Check




Automation not running
Manual override OFF? Before 4pm? Price negative?


Fronius not limiting
Modbus connected? Register correct? Unit ID correct?


Still exporting
Increase safety buffer to 700-1000W


Sensor unavailable
Amber integration installed? Sensor names correct?




View traces: Settings → Automations → Your automation → Traces



⚠️ Critical Notes


Register 40232 works for Primo 8.2 - verify for YOUR model
Disable Amber SmartShift or it will conflict
Test with manual override first
Monitor for first few days and adjust settings
Your model may differ - check Fronius Modbus documentation



📚 Full Documentation


For detailed setup: Installation Guide


For help: GitHub Issues



Quick start complete! Monitor during next negative pricing period. ☀️
