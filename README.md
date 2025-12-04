Fronius Solar Curtailment for Amber Electric


Automatically limit Fronius solar generation during Amber Electric negative pricing to eliminate export charges



🎯 What Problem Does This Solve?


During negative Amber Electric pricing periods (typically 7am-4pm on sunny days), exporting solar to the grid costs you money instead of earning credits.


Example (verified on my system):


Fronius Primo 8.2 generates full 8.2kW
House uses 1.2kW during day
Battery (Alpha ESS) charges at ~4kW
Excess 3kW exported = You pay ~$0.09/kWh to export
Daily cost: $2-3 during negative pricing periods


With this automation:


Fronius limits to calculated target based on house load + battery charge + buffer
Prevents unnecessary grid export
Savings: Near-zero export during negative pricing



🌟 Features


✅ Automatic curtailment during negative Amber pricing
✅ Dynamic calculation based on actual house load + battery charging needs
✅ Works standalone - no external energy management system required
✅ Battery optional - set charge target to 0 if no battery
✅ Safety features - manual override and automatic reset
✅ Monitoring dashboard included



🔧 Verified Compatibility


My Tested System (gjh1967)




Component
Model
Status




Inverter
Fronius Primo 8.2-1
✅ Working


Battery
Alpha ESS SMILE-S5 (20kWh)
✅ Working


Retailer
Amber Electric (NSW)
✅ Working


Modbus Register
40232
✅ Confirmed


House Load
~1.2kW average
✅ Tested




Should Work With


Other Fronius Primo models (3.0, 5.0, 6.0, 8.2)
Fronius Symo models (check your Modbus register address)
Fronius Gen24 (check your Modbus register address)
Any battery system (or no battery)
Other dynamic pricing retailers (adapt price sensor)


Note: Modbus register 40232 works for most Fronius inverters but verify in your Fronius Modbus documentation.



📋 Prerequisites


Required


Fronius solar inverter with Modbus TCP support
Amber Electric account + Home Assistant integration installed
Home Assistant 2024.1 or newer
Network access to Fronius inverter


Required Sensors


Grid feed-in price (from Amber integration - sensor.amber_feed_in_price)
House load power in Watts
Grid export power in Watts
Battery SOC in % (if you have a battery)


Important


❌ Disable Amber SmartShift in the Amber app
❌ Disable Automated Solar Curtailment in the Amber app
This automation replaces those features



🚀 Quick Installation


Step 1: Install Amber Integration


Settings → Devices & Services → Add Integration
Search "Amber Electric"
Enter API token from https://app.amber.com.au
Verify sensor.amber_feed_in_price exists


Step 2: Enable Fronius Modbus


Access Fronius web interface (http://YOUR_FRONIUS_IP)
Communication → Modbus → Enable Modbus TCP
Port: 502, Unit ID: 1 (typical)
Save and note your IP address


Step 3: Configure Home Assistant Modbus


Add to configuration.yaml:


modbus:
  - name: fronius_control
    type: tcp
    host: YOUR_FRONIUS_IP  # Change this
    port: 502
    delay: 1
    timeout: 5



Restart Home Assistant.


Step 4: Create Helper Toggles


Settings → Helpers → Create two Toggles:


"Solar Curtailment Active" (icon: mdi:solar-power-variant)
"Fronius Manual Override" (icon: mdi:toggle-switch-off-outline)


Step 5: Import Blueprint





Or manually: Settings → Blueprints → Import Blueprint → Enter URL above


Step 6: Configure Automation


Settings → Automations → Create Automation
Select "Fronius Solar Curtailment for Amber Electric"
Configure your sensors and inverter details
Save


Full detailed guide: Installation Guide



⚙️ Key Configuration Settings


Required Settings




Setting
What to Enter
Where to Find




Grid Feed in Price Sensor
sensor.amber_feed_in_price
Amber integration


House Load Sensor
Your power sensor
Developer Tools → States


Modbus Hub Name
fronius_control
What you used in Step 3


Inverter Max Output
Your inverter watts
Nameplate (e.g., 8200 for Primo 8.2)


Modbus Register
40232
Works for most - verify in your Fronius docs


Unit ID
1
Standard for single inverter




Battery Settings




Setting
Recommended
Notes




Battery Charge Target
4500W
Set to 0 if no battery


Safety Buffer
500W
Increase if still exporting


Minimum Generation
500W
Safety floor





📊 How It Works


Calculation:


When Amber Price < 0 Cents:
  Target = House Load + Battery Charge Target + Safety Buffer
  Limit Fronius to Target (capped at inverter max)

When Amber Price ≥ 0 Cents:
  Restore Fronius to full power



Example (my system):


House load: 1200W
Battery target: 4500W
Safety buffer: 500W
Fronius limited to: 6200W


Result: Battery charges, house powered, minimal export.



🧪 My Real-World Results


System:


Fronius Primo 8.2-1
Alpha ESS SMILE-S5 (20kWh)
Amber Electric (NSW)
Average house load: 1.2kW during day


Results:


Before automation: $2-3/day export charges during negative pricing
After automation: Near-zero export, $0-0.43/day
Automation behavior: Activates during negative pricing, limits as close to house load as possible, restores full power when price positive


Status: Working reliably as of December 2025



❓ FAQ


Do I need EnergyManager.com.au?


No. This automation is standalone. If you use EnergyManager.com.au, this can work alongside it, but it's not required. Further integration work needed for EnergyManager compatibility.


Disclaimer: Not affiliated with EnergyManager.com.au.


Can I use this without a battery?


Yes. Set "Battery Charge Target" to 0. Automation limits to house load + buffer only.


What if I don't have Amber Electric?


Adapt for other dynamic pricing retailers such as LocalVolts. by using their price sensor. Core logic remains the same.


Is this safe?


Yes, with proper testing:


✅ Manual override for emergencies
✅ Automatic reset at 6am daily
✅ Safety minimum generation (500W default)
✅ Automatic restore at 4pm


Always test with manual override first.


Will this work with my Fronius model?


Verified working: Fronius Primo 8.2-1 with register 40232


Likely to work: Other Primo models, most Symo models with same register


Must verify: Gen24 may use different register - check your Modbus documentation


Critical: Confirm your Modbus register address in Fronius documentation before use.



📦 What's Included


Core Files


blueprint.yaml - Main automation blueprint
helpers.yaml - Companion automations (4pm restore, override handler, etc.)
dashboard.yaml - Monitoring dashboard template


Documentation


DOCS/Installation_Guide.md - Complete step-by-step setup
DOCS/QUICKSTART.md - 5-minute quick reference



🤝 Contributing


Contributions welcome!


Ways to Help


🐛 Report bugs with your system specs
✅ Test on different Fronius models and report results
📝 Improve documentation
🌟 Share your configuration and results


Especially Needed


Testing on Fronius Symo models
Testing on Fronius Gen24 models
Verification of Modbus register addresses for different models
Results from other Australian states
Configurations without batteries



⚠️ Disclaimer


This automation directly controls your solar inverter via Modbus.


✅ Test thoroughly with manual override first
✅ Monitor closely for first few days
✅ Verify Modbus register address for YOUR inverter model
❌ Author not responsible for any issues or damages
❌ Use at your own risk
❌ Not affiliated with Fronius, Amber Electric, or any manufacturers
❌ Not compatible with Amber SmartShift (must disable in app)


This works on MY system (Primo 8.2). Your mileage may vary - verify compatibility with your equipment.



📜 License


MIT License - Free to use, modify, and distribute.



🙏 Acknowledgments


Fronius for SunSpec Modbus documentation
Amber Electric for dynamic pricing API
Home Assistant community for testing and feedback
Claude.ai for development assistance



📞 Support


Getting Help


Read the Installation Guide
Check GitHub Issues
Post in Home Assistant Community


Reporting Issues


Include:


Fronius inverter model
Battery system (if any)
Home Assistant version
Automation trace
What you expected vs what happened



🔗 Links


Amber Electric Integration: https://www.home-assistant.io/integrations/amberelectric/
Fronius Modbus Docs: https://www.fronius.com/en/solar-energy/installers-partners/service-support/software
Home Assistant Forum: https://community.home-assistant.io
Amber Electric: https://www.amber.com.au



Made with ☀️ by gjh1967 and the Home Assistant community & Claudeai


If this helps you:


⭐ Star this repo
🗣️ Share with other Fronius + Amber users
💬 Report your results



Version: 1.0.0

Last Updated: December 2025

Status: Working on Fronius Primo 8.2-1 + Alpha ESS + Amber NSW ✅
