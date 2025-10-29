# EMU – Expandable Multi-material Unit - Assembly and Wiring Guides


#### Table of Contents
- [BOM](#bom)
- [Assembly Videos](#assembly-videos)
- [Wiring Instructions and Diagrams](#wiring-diagrams)


## BOM  
[Draft BOM here](https://docs.google.com/spreadsheets/d/1jYJXBgpc_iLDfC17fC2LTYKrSEy5ocPbGEQ_EEOGCvI)

## Assembly Videos

**Part 1: Filamentalist and Stepper Assembly Guide**<br/>
[![EMU Filamentalist and Stepper Assembly Guide](https://img.youtube.com/vi/FNcXC9HXUmI/0.jpg)](https://www.youtube.com/watch?v=FNcXC9HXUmI)
<br/>
**Errata:**
1. BMG Gear insertion is not visible in the video. Do this before adding the stepper main body and screwing it on.
2. Switch sensor screws are M2x10 self taping or m2x8 bhcs/shcs
3. Filament path - BMG gear alignment step is not shown in the video. Insert some filament through the unit manually, loosen the BMG geear set screw and move it back and forth. Screw down the BMG gear.
4. For the BMG tensioning arm, the bushing is no longer an option. 
5. Add a washer to the stepper tensioning arm on both sides of the spring.
6. Stepper body screws are all M3x20
<br/>

**Part 2: Drybox Assembly Guide**<br/>
[![EMU Dry Box Assembly Guide](https://img.youtube.com/vi/JZzMyOBCdSM/0.jpg)](https://www.youtube.com/watch?v=JZzMyOBCdSM)
<br/>

**Errata:**
1. A threaded stand off is inserted in the front lip of the lid and secured from either side with M3x40 FHCS bolts.
2. Box and wiring hatch changes - the wiring hatch now is screwed in from the outside of the unit. Push through the hole from the inside out and then screw in place.
3. Wiring hatch foam is now glued on the box itself. The foam chord channel is on the outside of the box. 
4. The BME sensor has been relocated to the LED cover.
<br/>

**Part 3: Base Assembly Guide** <br/>
[![EMU Base Assembly Guide](https://img.youtube.com/vi/2SmAMuXas38/0.jpg)](https://www.youtube.com/watch?v=2SmAMuXas38)
<br/>

**Part 4: Electronics Assembly Guide** <br/>
[![EMU Elecrronics Assembly Guide](https://img.youtube.com/vi/Iang2JYkTh8/0.jpg)](https://www.youtube.com/watch?v=Iang2JYkTh8)
<br/>

**Errata:**
1. Base unit top clamping screws (M3x8 FHCS) now screw into heatset inserts inserted on the other side of the frame.
2. External combiner sensor is no longer specified in the default setup. That wiring part can be omitted.
3. Encoder is no longer specified in the default setup That wiring part can be omitted.
<br/>


## Wiring Diagrams
Depending on the number of lanes assembled, the wiring will vary slightly. Below are some initial wiring key notes to get your started.<br/>

**Multi-lane setup wiring notes (2+ lanes)**
1. Please note that the sync feedback sensor is connected to the first lane board. The encoder and combiner endstop is connected to the second lane EBB board.
2. The CAN bus umbilical from the printer feeds power and the CAN H / L wires to the WAGO connectors in the first lane. 
3. A 24V and Ground "bus" is established throughout the base unit using the two rearward WAGO connectors (WAGO 3 and WAGO 4).
4. The EBB boards connect to the power bus for power via their XT ports.
5. The EBB boards are daisy chained to establish the CANbus network. The first lane canbus High / Low / Power wires are connected to the first lane WAGO connectors.
7. The second lane canbus High / Low wires are connected to the CAN out dupont connectors on the first board itself. Creates the daisy chain
9. The last EBB board needs the 120Ohm resistor installed.
10. The second lane, third lane etc 2nd WAGO connector is used to connect the Neopixel Out wire from the eject button to the Neopixel In wire on the dry box.
11. The second lane, third lane etc 1st WAGO connector is used to connect the Neopixel Out wire from the dry box to the next lane Neopixel In wire on the eject button.

**Adding a second base unit** <br/>
You can daisy chain as many base units as you desire. Simply wire the XT30+2 CAN out port next to the last lane to the power bus and CAN H / L dupont connectors on the last EBB board. The next base unit can connect using a standard CAN Bus cable to the first unit.


**Single-lane setup wiring notes (2+ lanes)** <br/>
1. A single lane setup does not support an encoder/combiner. Instead, the Post Extruder sensor is configured in the Happy Hare configuration as the gate sensor.
2. There is no daisy chaining required. The single EBB board is connected to the first lane WAGO connectors.
3. There is no daisy chaining of subsequent lane Neopixels. 

**EBB 42 wiring diagram**<br/>
<p align="center">
  <img src="/docs/wiring_diagrams/EMU_wiring_ebb42.png" width="95%">
</p>

**EBB 36 wiring diagram**<br/>
Please note, the EBB36 uses identical pin definitions to the EBB42.
<p align="center">
  <img src="/docs/wiring_diagrams/EMU_wiring_ebb36.png" width="95%">
</p>

**Printer PSU**<br/>
> [!WARNING]
> You need to evaluate whether your PSU has enough power headroom to operate the number of lanes of your choosing. This entirely depends on your printer's overall setup and how many lanes of the EMU you are planning to build.

Key factors to consider:
1. PSU wattage
2. How many A-B/X-Y and Z steppers 
3. At what current the main printer steppers are operating at 
4. Your heater cartridge wattage 
5. What other power consuming accessories are connected to the printer (cpap, heater bed if 24V etc)
6. How many EMU lanes you are planning to build

Depending on the combined power consumption, you may need to consider whether a PSU upgrade is needed. 

As a rule of thumb, a 12 lane EMU build with 4 Z steppers and 2 AB steppers running moderately high currents and high print speeds on a Voron 2.4 may require a 350Watt power supply to ensure adequate power headroom.
