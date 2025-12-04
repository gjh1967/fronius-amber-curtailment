Fronius Solar Curtailment - Installation Guide


Complete step-by-step installation instructions for the Fronius Solar Curtailment automation with Amber Electric.



📋 Prerequisites Check


Before starting, ensure you have:


Hardware


✅ Fronius solar inverter (Primo, Symo, or Gen24)
✅ Network access to Fronius inverter
✅ (Optional) Battery system
✅ Home Assistant server running


Software


✅ Home Assistant 2024.1 or newer
✅ Amber Electric account (or other dynamic pricing retailer)
✅ Access to Fronius web interface
✅ Basic Home Assistant knowledge


Required Sensors


You'll need sensors for:


Grid sell price (from Amber integration)
House load power (watts)
Solar generation power (watts)
Grid export power (watts)
Battery state of charge (if you have a battery)



Step 1: Install Amber Electric Integration


If you already have Amber integration installed, skip to Step 2.


1.1 Get Your Amber API Token


Go to https://app.amber.com.au
Log in to your account
Navigate to Account → Developer → API Token
Copy your API token (you'll need this next)


1.2 Install Integration in Home Assistant


Open Home Assistant
Go to Settings → Devices & Services
Click + Add Integration (bottom right)
Search for "Amber Electric"
Click on it
Enter your API token
Select your site/location
Click Submit


1.3 Verify Installation


Go to Developer Tools → States
Search for: amber
Verify these sensors exist:

sensor.amber_general_price (this is the one you'll use)
sensor.amber_general_forecast




Note: sensor.amber_general_price should show current pricing. During negative pricing periods, it will show negative values (e.g., -0.03 or -3 depending on format).


Important: In the Amber app, you need to:


❌ Turn OFF SmartShift
❌ Turn OFF Automated Solar Curtailment


This automation replaces those features for Fronius inverters.



Step 2: Enable Fronius Modbus


2.1 Access Fronius Web Interface


Find your Fronius inverter IP address:

Check your router's connected devices
Common default: fronius-XXXXXXX.local
Or scan your network


Open browser and go to: http://YOUR_FRONIUS_IP


2.2 Enable Modbus TCP


For Fronius Primo/Symo:


Login (default username: customer, no password usually)
Go to Communication → Modbus
Set Modbus TCP to Enabled
Set Port to 502
Note the Unit ID (usually 1)
Click Save
Reboot inverter if prompted


For Fronius Gen24:


Login to inverter
Go to Communication → Modbus
Enable SunSpec Model
Enable Modbus TCP
Port: 502
Save changes


2.3 Write Down These Details


You'll need:


✅ Fronius IP address: _______________
✅ Modbus Port: 502 (default)
✅ Unit ID: 1 (default)
✅ Inverter max output (e.g., 8200W for Primo 8.2)



Step 3: Configure Home Assistant Modbus


3.1 Edit configuration.yaml


Open your configuration.yaml file
Add this section (or add to existing modbus: section):


modbus:
  - name: fronius_control
    type: tcp
    host: 192.168.1.XXX  # CHANGE: Your Fronius IP address
    port: 502
    delay: 1
    timeout: 5



Replace 192.168.1.XXX with your actual Fronius IP address.


3.2 Restart Home Assistant


Go to Developer Tools → YAML
Click Check Configuration
If valid, go to Settings → System
Click Restart → Restart Home Assistant


3.3 Verify Modbus Connection


After restart:


Go to Settings → Devices & Services
Look for Modbus integration
It should show as configured



Step 4: Create Helper Entities


4.1 Create Solar Curtailment Active Toggle


Go to Settings → Devices & Services → Helpers
Click + Create Helper (bottom right)
Select Toggle
Configure:

Name: Solar Curtailment Active
Icon: mdi:solar-power-variant


Click Create


4.2 Create Manual Override Toggle


Click + Create Helper again
Select Toggle
Configure:

Name: Fronius Manual Override
Icon: mdi:toggle-switch-off-outline


Click Create


4.3 Verify Helpers Created


Go to Developer Tools → States and search for:


input_boolean.solar_curtailment_active ✅
input_boolean.fronius_manual_override ✅



Step 5: Import and Configure Blueprint


5.1 Import Blueprint


Option A: Direct Import (Easiest)


Click this button:





Option B: Manual Import


Go to Settings → Automations & Scenes
Click Blueprints tab
Click Import Blueprint (bottom right)
Enter URL: https://github.com/gjh1967/fronius-amber-curtailment/blob/main/blueprint.yaml
Click Preview → Import


5.2 Create Automation from Blueprint


Go to Settings → Automations & Scenes
Click + Create Automation (bottom right)
Select Fronius Solar Curtailment for Amber Electric
Now configure each setting...


5.3 Configure Sensors


Grid Sell Price Sensor


Select: sensor.amber_general_price
(Or sensor.amber_feed_in_price if that's what you have)


House Load Power Sensor


Select your house consumption sensor
Common names: sensor.house_power, sensor.load_power
Must be in Watts


Battery State of Charge Sensor


Select your battery SOC sensor
Should show 0-100%
Examples: sensor.battery_soc, sensor.alpha_ess_battery_level
If no battery: Select any sensor (won't be used if charge target is 0)


Grid Export Power Sensor


Select your grid export sensor
Must be in Watts
Examples: sensor.grid_export, sensor.meter_export_power


5.4 Configure Fronius Settings


Modbus Hub Name


Enter: fronius_control
(Or whatever you named it in Step 3)


Modbus Unit ID


Enter: 1
(Unless you changed it in Step 2)


SunSpec Power Limit Register Address


Default: 40232
This works for 90% of Fronius inverters
Only change if you know your model uses a different address


Inverter Maximum AC Output


Enter your inverter's rating in Watts
Examples:

Primo 3.0: 3000
Primo 5.0: 5000
Primo 8.2: 8200
Symo 10.0: 10000
Symo 15.0: 15000
Symo 20.0: 20000




5.5 Configure Battery & Charging


Target Battery Charge Power


Default: 4500 watts
Set to 0 if you have no battery
Common values:

Alpha ESS: 4500-5000W
Tesla Powerwall 2: 5000W
No battery: 0W




Safety Buffer


Default: 500 watts
Increase to 700-1000W if you still see export during curtailment
Decrease to 300W if limiting too much


Minimum Solar Generation


Default: 500 watts
Safety feature - inverter won't go below this


5.6 Configure Timing


Curtailment End Time


Default: 16:00:00 (4pm)
Adjust based on when negative pricing typically ends in your area


Manual Override Reset Time


Default: 06:00:00 (6am)
Automation resets override each morning


5.7 Save Automation


Click Save
Give it a name (e.g., "Fronius Solar Curtailment")
Ensure it's enabled (toggle should be on)



Step 6: Install Companion Automations


6.1 Create 4pm Auto-Restore Automation


Go to Settings → Automations & Scenes
Click + Create Automation
Choose Create new automation
Configure:


Trigger:


Type: Time
At: 16:00:00


Condition:


Type: State
Entity: input_boolean.solar_curtailment_active
State: on


Action:


Type: Call Service
Service: input_boolean.turn_off
Target: input_boolean.solar_curtailment_active


Name it: "Fronius - Auto Restore at 4pm"
Save


6.2 Create Manual Override Handler


Create new automation
Configure:


Trigger:


Type: State
Entity: input_boolean.fronius_manual_override
To: on


Action 1:


Service: input_boolean.turn_off
Target: input_boolean.solar_curtailment_active


Action 2:


Service: modbus.write_register
Hub: fronius_control
Unit: 1
Address: 40232
Value: [8200, 0, 0, 0, 1]
(Change 8200 to your inverter max output)


Name it: "Fronius - Manual Override Handler"
Save


6.3 Create 6am Override Reset


Create new automation
Configure:


Trigger:


Type: Time
At: 06:00:00


Condition:


Type: State
Entity: input_boolean.fronius_manual_override
State: on


Action:


Service: input_boolean.turn_off
Target: input_boolean.fronius_manual_override


Name it: "Fronius - Reset Override at 6am"
Save



Step 7: Install Dashboard (Optional)


7.1 Create New Dashboard View


Go to your main dashboard
Click Edit Dashboard (pencil icon)
Click + Add View
Set:

Title: Solar Curtailment
Icon: mdi:solar-power-variant


Click Save


7.2 Add Dashboard YAML


While in edit mode, click the three dots on the new view
Select Edit in YAML
Copy contents from dashboard.yaml file
IMPORTANT: Replace all entity IDs marked with # CHANGE THIS
Update hardcoded values:

Change 4500 to your battery charge target
Change 500 to your safety buffer
Change 8200 to your inverter max output


Click Save



Step 8: Testing


8.1 Test Manual Override (Safe Test)


Go to your dashboard
Turn ON the "Fronius Manual Override" toggle
Verify:

✅ "Solar Curtailment Active" turns OFF
✅ Solar generation returns to normal (if sunny)


Turn OFF manual override


If this works, Modbus communication is functioning! ✅


8.2 Test Automation Trace


Go to Settings → Automations & Scenes
Find your "Fronius Solar Curtailment" automation
Click on it
Click Traces (top right)
Look at recent runs
Check:

Are conditions passing?
What choice is being executed?
Are there any errors?




8.3 Wait for Negative Pricing


Best test is real conditions:


Check Amber app for next negative pricing period
Monitor automation during that time
Verify:

✅ solar_curtailment_active turns ON
✅ Fronius generation drops to calculated limit
✅ Grid export drops to near 0W
✅ Battery continues charging
✅ House load covered


When price returns positive:

✅ Curtailment turns OFF
✅ Full solar generation restored





Troubleshooting


Automation Not Running


Check:


Manual override is OFF
Current time is before 4pm
Grid price sensor shows negative value
Automation is enabled


Fix:


View automation traces
Check entity states in Developer Tools


Fronius Not Limiting


Check:


Modbus connection (Settings → Integrations)
Register address is correct (40232 for most)
Unit ID is correct (usually 1)
Inverter has Modbus TCP enabled


Test:


Try manual override - does it restore full power?
If yes: automation logic issue
If no: Modbus configuration issue


Still Exporting During Negative Pricing


Solutions:


Increase safety buffer to 700-1000W
Check house load sensor is correct
Verify battery charge target matches actual charge rate
Monitor for a few days and adjust


Sensors Unavailable


Check:


Amber integration installed and authenticated
Sensor names in Developer Tools → States
Sensors updating regularly (check Last Updated)



✅ Installation Complete!


Your Fronius solar curtailment automation is now installed and ready.


Next Steps


Monitor for first few days - watch how it performs
Adjust settings if needed (safety buffer, battery charge target)
Share your results - help the community by reporting your experience
Star the repo if it's working well!


Getting Help


Check Troubleshooting Guide
Review Configuration Guide
Ask in GitHub Issues
Post in Home Assistant Community



Installation guide complete! You're ready to save money on negative pricing days! ☀️💰
