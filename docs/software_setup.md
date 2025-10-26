# EMU Happy Hare Hardware and Software Setup Guide

This is a provisional sofware setup guide for the EMU using Happy Hare v3. This guide is meant to be read in conjunction with the Happy Hare setup guide as found here: https://github.com/moggieuk/Happy-Hare/wiki

## Table of Contents

- [Setting up CAN Bus](#setting-up-can-bus)
- [Flashing the EBB boards](#flashing-the-ebb-boards)
- [Updating the boards with the latest klipper version](#updating-the-boards-with-the-latest-klipper-version)
- [Setting up Happy Hare](#setting-up-happy-hare)
- [Configuring the EMU hardware in Happy Hare](#configuring-the-emu-hardware-in-happy-hare)
- [Configuring Happy Hare](#configuring-happy-hare)
- [Calibrating the unit](#calibrating-the-unit)
- [Next steps](#next-steps)


## Setting up CAN Bus
If your printer does not have canbus set up, follow the excellent guide from Esoterical here: https://canbus.esoterical.online.

Do not forget to terminate the last EBB board or CANBus device in your bus (depending on your bus topology).

## Flashing the EBB boards
Prior to setting up the EMU, the EBB boards need to be flashed with Katapult and Klipper. Flashing instructions can be found here https://canbus.esoterical.online/toolhead_flashing.html. The boards are flashed in exactly the same way as if they were used as a toolhead board. Below are some high level instructions; however do refer back to the guide by Esoterical for a more comprehensive guide.<br/>

**Step 1:** Install katapult:<br/>
```
test -e ~/katapult && (cd ~/katapult && git pull) || (cd ~ && git clone https://github.com/Arksine/katapult) ; cd ~
```
**Step 2:** Connect your first EBB board over USB **outside the EMU**. Make sure the **power by USB jumper is set**.<br/>
**Step 3:** Set the board in DFU mode by pressing and holding the reset and boot buttons. Release the reset button first, then release the boot button<br/>
**Step 4:** The board should show as in DFU mode. Run ```lsusb``` and check that something like the below is shown in the command line:<br/>
```
Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```
**Step 5:** initiate menuconfig for Katapult.<br/>
```
cd ~/katapult
make menuconfig
```
![image](https://github.com/user-attachments/assets/9f642d03-d781-4025-bc28-959574c30c8a)

```
make
```
**Step 6:** Flash katapult to the EBB board<br/>
```
sudo dfu-util -R -a 0 -s 0x08000000:mass-erase:force:leave -D ~/katapult/out/katapult.bin -d 0483:df11
```
**Step 7:** initiate menuconfig for klipper and make the firmware<br/>
```
sudo service klipper stop
cd ~/klipper
make clean
make menuconfig
```
![image](https://github.com/user-attachments/assets/f5ab3561-25aa-42c2-b838-8629a2cb38cf)
```
make
```
**Step 8:** Prepare to flash klipper<br/>
Now that katapult is loaded, unplug the USB cable, remove the USB power jumper (if installed) and install the EBB on the unit. Power it via the canbus pigtail.

Please note, **you can set up katapult on all the boards and install them back to the unit all at once**. However, identifying which board is which from the UID is more tricky if you have them all installed in the unit. 

You have two options to identify them.
1. Unplug the positive (red) wire from the wagos for all the boards except the one you are intending to flash klipper, Run the command from Step 9, note the UUID, flash klipper and note down the UUID you just used with the lane number in your documentation  _--or--_
2. Run the command from Step 9, pick one UUID, flash Klipper, note which board's led stopped flashing and note down the UUID you just used with the lane number in your documentation.

**Step 9:** Find the board UUID<br/>
Run the command below and note the produced UUID
```
python3 ~/katapult/scripts/flashtool.py -i can0 -q
```
**Step 10:** Flash klipper<br/>
```
python3 ~/katapult/scripts/flashtool.py -i can0 -f ~/klipper/out/klipper.bin -u youruuid
```
**Flashing the remaining boards:** <br/>

Repeat step 2 (connect via USB),3 (set in DFU mode),4 (lsusb to confirm DFU),6 (flash katapult),8 (install the board),9 (pick the UUID),10 (flash klipper) for each subsequent board. Please note that as the boards are the same you do not need to re-run the menu config and make process.

Remember to note down the UUID - Lane values for your reference. This combination will be used later in the guide to set up Happy Hare.

## Updating the boards with the latest klipper version
```
sudo service klipper stop
cd ~/klipper
make clean
make menuconfig
make
python3 ~/katapult/scripts/flashtool.py -i can1 -u youruuid -r
python3 ~/katapult/scripts/flashtool.py -i can1 -u youruuid -f ~/klipper/out/klipper.bin
```
Repeat the last two steps for each EBB board. Note the uuid used with the lane number in your documentation. This will be used later.

## Setting up Happy Hare
Install Happy Hare. More detailed instructions can be found here: https://github.com/moggieuk/Happy-Hare/wiki/Installation
```
cd ~
git clone https://github.com/moggieuk/Happy-Hare.git
cd ~/Happy-Hare
./install.sh -i
```
**Select MMU:** Pick option 16 - Other/Custom. <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/7a0875fe-fc19-4172-8f1c-deb0e7f4bb1c" alt="Installer screen: Pick option 16 - Other/Custom" width="80%">
</p>

**Select MMU type:** Pick option 7 - Type-B (multiple filament drive steppers) with individual post-gear sensors, No Encoder <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/d1ddd2f6-689f-45ee-a54c-eb8dbc09bf76" alt="Installer screen: Pick option 7 - Type-B with individual post-gear sensors, no encoder" width="80%">
</p>

**Select lane number:** Type the number of lanes you have set up. In my example below it is 8. <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/cc2bfdec-5ab5-4715-9762-552189d4da7f" alt="Enter number of lanes (example: 8)" width="80%">
</p>

**Select the board used:** Pick option 15 - Not in list / Unknown <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/23d291e3-ea6b-45d7-888f-879e93633102" alt="Select board: option 15 Not in list / Unknown" width="80%">
</p>

**LED Support:** Enable LED support <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/b77bf49f-08e1-41dd-a3e1-17870d80b37b" alt="Enable LED support" width="80%">
</p>

**Endless Spool Support:** Enable Endless Spool support <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/b967c996-0aa4-4c02-9de8-6f7dc08fde91" alt="Enable Endless Spool support" width="80%">
</p>

**Include all the MMU config files:** Include all mmu config files in the printer.cfg <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/e1668ab6-4c4b-4392-bc74-ed01228eb570" alt="Include MMU config files in printer.cfg" width="80%">
</p>

**(Optional) Mini 12864 screen support:** If you have a Mini 12864 screen, enabe the Happy Hare screen menu <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/83d4cb41-0651-4f1d-aebf-6e1c41ed148a" alt="Enable Happy Hare menu for Mini 12864 (optional)" width="80%">
</p>

**Default pause/resume macros:** Include the default pause/resume macros from Happy Hare. Once the installation is complete, make sure you disable/remove any custom pause/resume macros that you may have in your existing Klipper installation! <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/4141828a-493c-4bae-92ce-3db9b4941633" alt="Include default pause/resume macros from Happy Hare" width="80%">
</p>

**EREC support:** Decline the installation of the EREC filament cutter <br/><br/>
<p align="center">
  <img src="https://github.com/user-attachments/assets/7f0d3412-4ee3-42e5-bbed-569f64db7d8a" alt="Decline EREC filament cutter installation" width="80%">
</p>

**Moonraker.conf update:** Update your moonraker.conf as below:
```
[file_manager]
enable_object_processing: True

[mmu_server]
enable_file_preprocessor: True
enable_toolchange_next_pos: True
update_spoolman_location: True
```
![image](https://github.com/user-attachments/assets/ac85bba6-3346-47cf-b396-6b29552069f5)
<br/><br/>
Baseline setup is now complete!
<br/><br/>
## Configuring the EMU hardware in Happy Hare

### Update mmu/base/mmu.cfg
The Happy hare installer generates a generic mmu.cfg file that needs re-writing for the EMU. 

**Step 1: Clear the mmu.cfg file.** <br/><br/>
Start by completely deleting the content of that file and hitting save.

**Step 2: Update mmu.cfg with the EMU mcu boards definitions** <br/><br/>
For each lane you will need a distinct mcu mmu block defining the board name and the canbus UUID as noted earlier. In the example below I am using an 8 lane configuration.
```
[mcu mmu0]
canbus_uuid: cc8f39d714ce 
canbus_interface: can0

[mcu mmu1]
canbus_uuid: c2357f5873d7 
canbus_interface: can0

[mcu mmu2]
canbus_uuid: 31fb8c95fbfd 
canbus_interface: can0

[mcu mmu3]
canbus_uuid: 8579c5130649 
canbus_interface: can0

[mcu mmu4]
canbus_uuid: 49abda4839e6
canbus_interface: can0

[mcu mmu5]
canbus_uuid: 7d42f7c43687
canbus_interface: can0

[mcu mmu6]
canbus_uuid: 3efbe46e5d5f
canbus_interface: can0

[mcu mmu7]
canbus_uuid: f050daaa5448 
canbus_interface: can0
```

**Step 3: Update mmu.cfg with the EMU mcu board pin aliases** <br/><br/>
After the board definitions, insert the board pin aliases. **Important note:** The first board defines the aliases for the tension and compression sensor too, so it is different to the rest. In the example below I am using an 8 lane configuration.
```
[board_pins mmu0]
mcu: mmu0 
aliases:
    MMU_GEAR_UART=PA15,
    MMU_GEAR_STEP=PD0,
    MMU_GEAR_DIR=PD1,
    MMU_GEAR_ENABLE=PD2,
    MMU_GEAR_DIAG=,

    MMU_NEOPIXEL=PD3,
    MMU_NEOPIXEL_LOGO=,

    MMU_PRE_GATE_0=PB7,
    MMU_POST_GEAR_0=PB5,

    MMU_TENSION=PB8, # First lane has the sync feedback sensor attached to it!
    MMU_COMPRESSION=PB9, # First lane has the sync feedback sensor attached to it!

    EJECT_BUTTON_0=PB6,

[board_pins mmu1]
mcu: mmu1 
aliases:

    MMU_GEAR_UART_1=PA15,
    MMU_GEAR_STEP_1=PD0,
    MMU_GEAR_DIR_1=PD1,
    MMU_GEAR_ENABLE_1=PD2,
    MMU_GEAR_DIAG_1=,

    MMU_PRE_GATE_1=PB7,
    MMU_POST_GEAR_1=PB5,

    EJECT_BUTTON_1=PB6,

[board_pins mmu2]
mcu: mmu2 
aliases:

    MMU_GEAR_UART_2=PA15,
    MMU_GEAR_STEP_2=PD0,
    MMU_GEAR_DIR_2=PD1,
    MMU_GEAR_ENABLE_2=PD2,
    MMU_GEAR_DIAG_2=,

    MMU_PRE_GATE_2=PB7,
    MMU_POST_GEAR_2=PB5,

    EJECT_BUTTON_2=PB6,

[board_pins mmu3]
mcu: mmu3
aliases:

    MMU_GEAR_UART_3=PA15,
    MMU_GEAR_STEP_3=PD0,
    MMU_GEAR_DIR_3=PD1,
    MMU_GEAR_ENABLE_3=PD2,
    MMU_GEAR_DIAG_3=,

    MMU_PRE_GATE_3=PB7,
    MMU_POST_GEAR_3=PB5,

    EJECT_BUTTON_3=PB6,

[board_pins mmu4]
mcu: mmu4 
aliases:

    MMU_GEAR_UART_4=PA15,
    MMU_GEAR_STEP_4=PD0,
    MMU_GEAR_DIR_4=PD1,
    MMU_GEAR_ENABLE_4=PD2,
    MMU_GEAR_DIAG_4=,

    MMU_PRE_GATE_4=PB7,
    MMU_POST_GEAR_4=PB5,

    EJECT_BUTTON_4=PB6,

[board_pins mmu5]
mcu: mmu5
aliases:

    MMU_GEAR_UART_5=PA15,
    MMU_GEAR_STEP_5=PD0,
    MMU_GEAR_DIR_5=PD1,
    MMU_GEAR_ENABLE_5=PD2,
    MMU_GEAR_DIAG_5=,

    MMU_PRE_GATE_5=PB7,
    MMU_POST_GEAR_5=PB5,

    EJECT_BUTTON_5=PB6,

[board_pins mmu6]
mcu: mmu6
aliases:

    MMU_GEAR_UART_6=PA15,
    MMU_GEAR_STEP_6=PD0,
    MMU_GEAR_DIR_6=PD1,
    MMU_GEAR_ENABLE_6=PD2,
    MMU_GEAR_DIAG_6=,

    MMU_PRE_GATE_6=PB7,
    MMU_POST_GEAR_6=PB5,

    EJECT_BUTTON_6=PB6,

[board_pins mmu7]
mcu: mmu7
aliases:

    MMU_GEAR_UART_7=PA15,
    MMU_GEAR_STEP_7=PD0,
    MMU_GEAR_DIR_7=PD1,
    MMU_GEAR_ENABLE_7=PD2,
    MMU_GEAR_DIAG_7=,

    MMU_PRE_GATE_7=PB7,
    MMU_POST_GEAR_7=PB5,

    EJECT_BUTTON_7=PB6,
```
This file is now complete! 


### Update mmu/base/mmu_hardware.cfg

**Step 1: Clear the mmu_hardware.cfg file.** <br/><br/>
Start by completely deleting the content of that file and hitting save.

**Step 2: Paste the below configuration in the mmu_hardware.cfg file**<br/><br/>
The below starter setup is for an 8 lane unit. To set up a lower lane count, paste the complete content below and change the below:
1. num_gates: 8 -> to equal to the number of lanes you have
2. Delete the unecessary tmc and stepper blocks. For example if you have a 5 lane unit, delete Filament Drive Gear_5, Filament Drive Gear_6, Filament Drive Gear_7 blocks from the below.
3. Delete the uncesessary pre_gate_switch_pin sections and post_gear_switch_pin sections. For example for a 5 lane setup, remove pre_gate_switch_pin_5, pre_gate_switch_pin_6, pre_gate_switch_pin_7, post_gear_switch_pin_5, post_gear_switch_pin_6, post_gear_switch_pin_7
4. Update the LED chain_count: 16 is for 8 lanes. This needs to be equal to number of lanes x 2. So for a 5 lane setup this would be set to 10.
5. Update the LED effect exit leds: exit_leds: neopixel:mmu_leds (1,3,5,7,9,11,13,15) is for 8 lanes. For 5, this should be 1,3,5,7,9.
6. Update the LED effect entry leds: entry_leds: neopixel:mmu_leds (2,4,6,8,10,12,14,16) is for 8 lanes. For 5, this should be 2,4,6,8,10

If you have more than 8 lanes, insert accordingly additional blocks, following the patterns as illustrated in the full configuration file below.

**Note:** If during testing you see that the EMU stepper spins backwards, invert the dir_pin by adding a ! infront of it (!dir_pin).

Finally, update your toolhead sensors definition to match your setup. Here I have the extruder entry (top) sensor in my toolhead can board named EBBCan connected to pin PB6 and the toolhead sensor (post extruder) at pin PB5.
```
extruder_switch_pin: ^EBBCan: PB6
toolhead_switch_pin: ^EBBCan: PB5
```

File contents - mmu_hardware.cfg:

```
[mmu_machine]
num_gates: 8			
mmu_vendor: Other			
mmu_version: 1.0			

selector_type: VirtualSelector
variable_bowden_lengths: 1		
variable_rotation_distances: 1		
require_bowden_move: 1		
filament_always_gripped: 1	
has_bypass: 1 


# FILAMENT DRIVE GEAR STEPPER  -----------------------------------------------------------------------------------------
[tmc2209 stepper_mmu_gear]
uart_pin: mmu0:MMU_GEAR_UART
run_current: 0.8		
hold_current: 0.2	
interpolate: False
sense_resistor: 0.110	
stealthchop_threshold: 1		

[stepper_mmu_gear]
step_pin: mmu0:MMU_GEAR_STEP
dir_pin: mmu0:MMU_GEAR_DIR
enable_pin: !mmu0:MMU_GEAR_ENABLE
rotation_distance: 22.7574		
gear_ratio: 1:1			
microsteps: 32				
full_steps_per_rotation: 200		

# Filament Drive Gear_1 --------------------------
[tmc2209 stepper_mmu_gear_1]
uart_pin: mmu1:MMU_GEAR_UART_1

[stepper_mmu_gear_1]
step_pin: mmu1:MMU_GEAR_STEP_1
dir_pin: mmu1:MMU_GEAR_DIR_1
enable_pin: !mmu1:MMU_GEAR_ENABLE_1

# Filament Drive Gear_2 --------------------------
[tmc2209 stepper_mmu_gear_2]
uart_pin: mmu2:MMU_GEAR_UART_2

[stepper_mmu_gear_2]
step_pin: mmu2:MMU_GEAR_STEP_2
dir_pin: mmu2:MMU_GEAR_DIR_2
enable_pin: !mmu2:MMU_GEAR_ENABLE_2

# Filament Drive Gear_3 --------------------------
[tmc2209 stepper_mmu_gear_3]
uart_pin: mmu3:MMU_GEAR_UART_3

[stepper_mmu_gear_3]
step_pin: mmu3:MMU_GEAR_STEP_3
dir_pin: mmu3:MMU_GEAR_DIR_3
enable_pin: !mmu3:MMU_GEAR_ENABLE_3

# Filament Drive Gear_4 --------------------------
[tmc2209 stepper_mmu_gear_4]
uart_pin: mmu4:MMU_GEAR_UART_4

[stepper_mmu_gear_4]
step_pin: mmu4:MMU_GEAR_STEP_4
dir_pin: mmu4:MMU_GEAR_DIR_4
enable_pin: !mmu4:MMU_GEAR_ENABLE_4

# Filament Drive Gear_5 --------------------------
[tmc2209 stepper_mmu_gear_5]
uart_pin: mmu5:MMU_GEAR_UART_5

[stepper_mmu_gear_5]
step_pin: mmu5:MMU_GEAR_STEP_5
dir_pin: mmu5:MMU_GEAR_DIR_5
enable_pin: !mmu5:MMU_GEAR_ENABLE_5

# Filament Drive Gear_6 --------------------------
[tmc2209 stepper_mmu_gear_6]
uart_pin: mmu6:MMU_GEAR_UART_6

[stepper_mmu_gear_6]
step_pin: mmu6:MMU_GEAR_STEP_6
dir_pin: mmu6:MMU_GEAR_DIR_6
enable_pin: !mmu6:MMU_GEAR_ENABLE_6

# Filament Drive Gear_7 --------------------------
[tmc2209 stepper_mmu_gear_7]
uart_pin: mmu7:MMU_GEAR_UART_7

[stepper_mmu_gear_7]
step_pin: mmu7:MMU_GEAR_STEP_7
dir_pin: mmu7:MMU_GEAR_DIR_7
enable_pin: !mmu7:MMU_GEAR_ENABLE_7

[mmu_sensors]
pre_gate_switch_pin_0: ^mmu0:MMU_PRE_GATE_0
pre_gate_switch_pin_1: ^mmu1:MMU_PRE_GATE_1
pre_gate_switch_pin_2: ^mmu2:MMU_PRE_GATE_2
pre_gate_switch_pin_3: ^mmu3:MMU_PRE_GATE_3
pre_gate_switch_pin_4: ^mmu4:MMU_PRE_GATE_4
pre_gate_switch_pin_5: ^mmu5:MMU_PRE_GATE_5
pre_gate_switch_pin_6: ^mmu6:MMU_PRE_GATE_6
pre_gate_switch_pin_7: ^mmu7:MMU_PRE_GATE_7

post_gear_switch_pin_0: ^mmu0:MMU_POST_GEAR_0
post_gear_switch_pin_1: ^mmu1:MMU_POST_GEAR_1
post_gear_switch_pin_2: ^mmu2:MMU_POST_GEAR_2
post_gear_switch_pin_3: ^mmu3:MMU_POST_GEAR_3
post_gear_switch_pin_4: ^mmu4:MMU_POST_GEAR_4
post_gear_switch_pin_5: ^mmu5:MMU_POST_GEAR_5
post_gear_switch_pin_6: ^mmu6:MMU_POST_GEAR_6
post_gear_switch_pin_7: ^mmu7:MMU_POST_GEAR_7

extruder_switch_pin: ^EBBCan: PB6
toolhead_switch_pin: ^EBBCan: PB5

sync_feedback_tension_pin: ^mmu0:MMU_TENSION         # Compression is when you pull the bowden tubes (entry/exit) away from each other. Tension when you push the tubes together.
sync_feedback_compression_pin: ^mmu0:MMU_COMPRESSION

# MMU NEOPIXEL LED SUPPORT ------------------------------------------------------------------------------------

[neopixel mmu_leds]
pin: mmu0:MMU_NEOPIXEL
chain_count: 16			# Number lanes x2
color_order: GRBW		# Set based on your particular neopixel specification

# MMU LED EFFECT SEGMENTS ----------------------------------------------------------------------------------------------

[mmu_leds unit0]
exit_leds:   neopixel:mmu_leds (1,3,5,7,9,11,13,15) # First, third, fifth LED and so forth.
entry_leds: neopixel:mmu_leds (2,4,6,8,10,12,14,16) # Second, fourth, sixth LED and so forth.
logo_leds:    
frame_rate: 24

enabled: True                           # True = LEDs are enabled at startup (MMU_LED can control), False = Disabled
animation: True                         # True = Use led-animation-effects, False = Static LEDs
exit_effect: gate_status                #    off|gate_status|filament_color|slicer_color|r,g,b|_effect_
entry_effect: filament_color            #    off|gate_status|filament_color|slicer_color|r,g,b|_effect_
status_effect: off                      # on|off|gate_status|filament_color|slicer_color|r,g,b|_effect_
logo_effect: (0, 0, 0)                        #    off                                        |r,g,b|_effect_
white_light: (1, 1, 1)                  # RGB color for static white light
black_light: (1, 1, 1)                  # RGB color used to represent "black" (filament)
empty_light: (0.0, 0.0, 0.0)                  # RGB color used to represent empty gate

effect_loading:            mmu_blue_clockwise_slow, (0, 0, 0.4)
effect_loading_extruder:   mmu_blue_clockwise_fast, (0, 0, 1)
effect_unloading:          mmu_blue_anticlock_slow, (0, 0, 0.4)
effect_unloading_extruder: mmu_blue_anticlock_fast, (0, 0, 1)
effect_heating:            mmu_breathing_red,       (0.3, 0, 0)
effect_selecting:          mmu_white_fast,          (0.2, 0.2, 0.2)
effect_checking:           mmu_white_fast,          (0.8, 0.8, 0.8)
effect_initialized:        mmu_rainbow,             (0.5, 0.2, 0)
effect_error:              mmu_strobe,              (1, 0, 0)
effect_complete:           mmu_sparkle,             (0.3, 0.3, 0.3)
effect_gate_selected:      mmu_static_blue,         (0, 0, 1)
effect_gate_available:     mmu_static_white_dim,        (0.3, 0.3, 0.3)
effect_gate_available_sel: mmu_ready_white,         (0.75, 0.75, 0.75)
effect_gate_unknown:       mmu_static_orange,       (0.5, 0.2, 0)
effect_gate_unknown_sel:   mmu_ready_orange ,       (0.75, 0.3, 0)
effect_gate_empty:         mmu_static_black,        (0, 0, 0)
effect_gate_empty_sel:     mmu_ready_red,          (0.2, 0, 0)
```

### Update mmu/addons/mmu_eject_buttons_hw.cfg
**Step 1: Clear the mmu_eject_buttons_hw.cfg file.** <br/><br/>
Start by completely deleting the content of that file and hitting save.

**Step 2: Paste the below configuration in the mmu_eject_buttons_hw.cfg file**<br/><br/>
The below starter setup is for an 8 lane unit. To set up a lower lane count, paste the complete content below and delete the corresponding [gcode_button mmu_eject_button_N] sections. If you have more than 8 lanes, add more blocks following the patterns below.
```
[gcode_button mmu_eject_button_0]
pin: mmu0:EJECT_BUTTON_0
press_gcode: _MMU_EJECT_BUTTON GATE=0

[gcode_button mmu_eject_button_1]
pin: mmu1:EJECT_BUTTON_1
press_gcode: _MMU_EJECT_BUTTON GATE=1

[gcode_button mmu_eject_button_2]
pin: mmu2:EJECT_BUTTON_2
press_gcode: _MMU_EJECT_BUTTON GATE=2

[gcode_button mmu_eject_button_3]
pin: mmu3:EJECT_BUTTON_3
press_gcode: _MMU_EJECT_BUTTON GATE=3

[gcode_button mmu_eject_button_4]
pin: mmu4:EJECT_BUTTON_4
press_gcode: _MMU_EJECT_BUTTON GATE=4

[gcode_button mmu_eject_button_5]
pin: mmu5:EJECT_BUTTON_5
press_gcode: _MMU_EJECT_BUTTON GATE=5

[gcode_button mmu_eject_button_6]
pin: mmu6:EJECT_BUTTON_6
press_gcode: _MMU_EJECT_BUTTON GATE=6

[gcode_button mmu_eject_button_7]
pin: mmu7:EJECT_BUTTON_7
press_gcode: _MMU_EJECT_BUTTON GATE=7
```

### Update printer.cfg
We will be updating the printer.cfg to add support for the BME humidity sensors as below. Similarly to the other files, remove or add the temperature sensor blocks to match your lane count.

```
[temperature_sensor Lane_0]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu0
i2c_software_scl_pin: mmu0:PB3
i2c_software_sda_pin: mmu0:PB4

[temperature_sensor Lane_1]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu1
i2c_software_scl_pin: mmu1:PB3
i2c_software_sda_pin: mmu1:PB4

[temperature_sensor Lane_2]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu2
i2c_software_scl_pin: mmu2:PB3
i2c_software_sda_pin: mmu2:PB4

[temperature_sensor Lane_3]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu3
i2c_software_scl_pin: mmu3:PB3
i2c_software_sda_pin: mmu3:PB4

[temperature_sensor Lane_4]
sensor_type: BME280
bme280_report_time: 20
i2c_address: 118
i2c_mcu: mmu4
i2c_software_scl_pin: mmu4:PB3
i2c_software_sda_pin: mmu4:PB4

[temperature_sensor Lane_5]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu5
i2c_software_scl_pin: mmu5:PB3
i2c_software_sda_pin: mmu5:PB4

[temperature_sensor Lane_6]
sensor_type: BME280
bme280_report_time: 20
i2c_address: 118
i2c_mcu: mmu6
i2c_software_scl_pin: mmu6:PB3
i2c_software_sda_pin: mmu6:PB4

[temperature_sensor Lane_7]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmu7
i2c_software_scl_pin: mmu7:PB3
i2c_software_sda_pin: mmu7:PB4
```

### Save, restart and confirm lanes are visible
Once all of the above steps are completed hit save and restart klipper. The MMU hardware should now all be set up and all your lanes visible in the machine panel as below:

<p align="center">
  <img src="https://github.com/user-attachments/assets/ec2f4983-2228-4a1b-92f5-a9b0ec7f4e13" alt="Installer screen" width="80%">
</p>

## Configuring Happy Hare
Now that the hardware is set up and visible to klipper, the next step is to configure the MMU software to match the EMU configuration and capabilities. That configuration will be predominately done in the mmu/base/mmu_parameters.cfg file.

### Setting up the EMU software parameters in mmu/base/mmu_parameters.cfg
Unlike the hardware setup files, do not delete the content of this file. We will be making targeted changes to match the EMU capabilities as validated through our testing.

**MMU Hardware Limits:** The EMU can operate up to a maximum of 500mm/sec feeding velocity and 400mm/sec2 acceleration. Adjust the maximum hardware limits as below.</br>
```
gear_max_velocity: 500
gear_max_accel: 400
```
**Logging:** Reduce log verbosity to reduce SD Card wear and system load</br></br>
```
log_level: 1
log_file_level: 1		
log_statistics: 0 		
log_visual: 1			
log_startup_status: 1		
log_m117_messages: 1
```			
**Speeds:** Our objective is reliability with operation. While the EMU can operate at 500mm/sec feeding, any snag in the filament path risks the lane stepper skipping steps. As such, it is recommended that conservative values are used in the speed set up, and tune up from there.</br>
```
gear_from_spool_speed: 200
gear_from_spool_accel: 150
gear_from_buffer_speed: 200
gear_from_buffer_accel: 150
gear_unload_speed: 200
gear_unload_accel: 150

gear_short_move_speed: 100
gear_short_move_accel: 150
gear_short_move_threshold: 100
gear_homing_speed: 100

extruder_load_speed: 16
extruder_unload_speed: 20
extruder_sync_load_speed: 16
extruder_sync_unload_speed: 20
extruder_homing_speed: 16

macro_toolhead_max_accel: 0
macro_toolhead_min_cruise_ratio: 0.5
```
**Gate load:** The EMU uses the post stepper (gear) switch sensor as the homing point for the gates. In addition a conservative homing max and preload homing max value has been set to allow the user time to load the filament and ensure it is gripped by the EMU lane stepper. Use the below values as is and do not modify unless trying to troubleshoot a specific issue.</br>
```
gate_homing_endstop: mmu_gear	
gate_homing_max: 600
gate_preload_homing_max: 600
gate_preload_parking_distance: 2
gate_unload_buffer: 50
gate_parking_distance: 2
gate_endstop_to_encoder: 13
gate_autoload: 1
gate_final_eject_distance: 100
```
**Bowden load:** As the EMU can accomodate long bowden tubes feeding to the printer, the following adjustments need to be made to allow Happy Hare to operate in that environment.</br>
```
bowden_homing_max: 3000	# if the EMU lane to toolhead distance is larger than 3 meters, increase this value.

bowden_apply_correction: 0	
bowden_allowable_load_delta: 10.0

bowden_pre_unload_test: 0	
bowden_pre_unload_error_tolerance: 60
```
**Extruder homing:** From experience the below values are tuned to improve overall feeding reliability vs the Happy Hare stock values.</br>
```
extruder_homing_max: 400			# Larger than the default HH value. This allows for homing to complete even if the filament skips steps during the load/
extruder_homing_endstop: extruder	
extruder_homing_buffer: 50
extruder_collision_homing_current: 100

extruder_force_homing: 0
```
**Toolhead Load:** This section needs to be populated following toolhead dimension calibration. Starter values from the community are available here: https://link.3dcoded.xyz/tipconfigs/ . Adust the below settings as shown to improve feeding reliability and allow for more "flex" in a slightly miss-tuned setup</br>
```
toolhead_homing_max: 250			# Increased from default of 40, to allow more tolerance in the extruder not grabbing the filament immediately.
toolhead_unload_safety_margin: 50	# Increased from the default of 10, to ensure filament is for sure clear of the extruder during unload.
toolhead_post_load_tighten: 0		# this is ignored as EMU is a Type B MMU - set to 0
toolhead_post_load_tension_adjust: 1	# Adjust tension post load to attemt to centre the sync feedback sensor. 
toolhead_entry_tension_test: 0		# Not required - toolhead sensor ensures feeding has happened.
```
**Tip Forming:** A toolhead cutter is highly recommended. Venture into tip forming at your own risk!</br>
```
force_form_tip_standalone: 1		  # Always standalone tip forming (TURN SLICER OFF!)
form_tip_macro: _MMU_CUT_TIP	
extruder_form_tip_current: 130		# Bump up extruder current during tip forming. Helps guarantee no skipped steps on the extruder if cutting is not 100% clean
slicer_tip_park_pos: 0			      # This specifies the position of filament in extruder after slicer completes tip forming. (TURN SLICER TIP FORMING OFF!)
```
**Purging:** If you have the blobifier set up, enable it here. For a baseline, starter setup, the below is recommended.</br>
```
force_purge_standalone: 0 		# Use slicer purging (purge block)
purge_macro: _MMU_PURGE			# Replace this with BLOBIFIER when set up (and set force purge stand alone to 1)
extruder_purge_current: 100		# leave this unchanged.
```
**Motor sync:** Setup sync feedback sensor (EMUSync) here.</br>
```
sync_gear_current: 80			# 80% EMU stepper current during printing
sync_feedback_enabled: 1		# Turn on sync feedback sensor
sync_feedback_buffer_range: 12		# EMU Sync switch to switch distance
sync_feedback_buffer_maxrange: 25	# EMU Sync max travel
sync_multiplier_high: 1.10		# over-drive by 10% when in tension
sync_multiplier_low: 0.90		# under-drive by 10% when in compression
```
**Filament Management:** Disable clog detection and enable endless spool.</br>
```
enable_clog_detection: 0		
enable_endless_spool: 1			# enable endless spool
endless_spool_on_load: 0	
endless_spool_eject_gate: -1		# Eject filament in the current gate
```
Here you can also configure spoolman filament management (highly recommended - instructions here:https://github.com/moggieuk/Happy-Hare/wiki/Spoolman-Support). </br></br>

**Calibration:** While happy hare can auto calibrate both the bowden tube length and lane rotation distance, I personally recommend a manual calibration for run to run consistency</br>
```
autocal_bowden_length: 0	# Disable automated bowden length calibration
autotune_bowden_length: 0	# Disable automated bowden length tuning
skip_cal_rotation_distance: 0	# Require gate rotation distance calibration
autotune_rotation_distance: 0	# Disable automated gate calibration/tuning.
skip_cal_encoder: 0		# Encoder not fitted
autotune_encoder: 0		# Encoder not fitted
```

**Misc:** Minor options tuning and instructing Happy Hare that a filamentalist rewinder is used instead of a buffer </br>
```
timeout_pause: 86400		    # Idle time out 
disable_heater: 60		      # Delay in seconds after which the hotend heater is disabled in the MMU_PAUSE state
default_extruder_temp: 230	# Default temperature for performing swaps and forming tips when not in print (overridden by gate map)
extruder_temp_variance: 30	# When waiting for extruder temperature this is the +/- permissible variance in degrees (>= 1)

startup_home_if_unloaded: 0	    # 0 = do nothing
startup_reset_ttg_map: 0	      #  0 = do nothing
show_error_dialog: 0		        # 0 = show error in console
preload_attempts: 5		          # How many "grabbing" attempts are made to pick up the filament with preload feature
strict_filament_recovery: 0	    # If enabled with MMU with toolhead sensor, this will cause filament position recovery to
				                        # perform extra moves to look for filament trapped in the space after extruder but before sensor
filament_recovery_on_pause: 1	  # 1 = Run a quick check to determine current filament position on pause/error, 0 = disable
retry_tool_change_on_error: 1	  # Automatically retry a failed tool change.
bypass_autoload: 1		          # If extruder sensor fitted this controls the automatic loading of extruder for bypass operation
has_filament_buffer: 0          # Whether the MMU has a filament buffer. Set to 0 if using Filamentalist or DC eSpooler, etc
#
# Advanced options. Don't mess unless you fully understand. Read documentation.
#
encoder_move_validation: 1	# ADVANCED: 1 = Normally Encoder validates move distances are within given tolerance
                            #           0 = Validation is disabled (eliminates slight pause between moves but less safe)
print_start_detection: 1	  # ADVANCED: Enabled for Happy Hare to automatically detect start and end of print and call
				                    # ADVANCED: MMU_PRINT_START and MMU_PRINT_END automatically. Harmless to leave enabled but can disable
                            #           if you think it is causing problems and known START/END is covered in your macros
extruder: extruder		      # ADVANCED: Name of the toolhead extruder that MMU is using
gcode_load_sequence: 0		  # VERY ADVANCED: Gcode loading sequence 1=enabled, 0=internal logic (default)
gcode_unload_sequence: 0	  # VERY ADVANCED: Gcode unloading sequence, 1=enabled, 0=internal logic (default)
```

**Klipper:** Enable klipper TTC mitigations </br>
```
update_trsync: 1		      # 1 = Increase TRSYNC_TIMEOUT, 0 = Leave the klipper default
canbus_comms_retries: 3		# Number of retries. Recommend the default of 3.
update_bit_max_time: 1		# 1 = Increase BIT_MAX_TIME, 0 = Leave the klipper default
update_aht10_commands: 0	# 1 = Config AHT10 for BTT ViViD heater sensor, 0 = Leave the klipper default
```

### Setting up the EMU software parameters in mmu/base/mmu_macro_vars.cfg
Unlike the hardware setup files, do not delete the content of this file. The below are some common settings that I have found usefull to change from default. However the detailed guide on Happy hare needs to be followed to set up your cut tip macro and to configure the system for your specific printer setup. For more read here: https://github.com/moggieuk/Happy-Hare/wiki/Configuring-mmu_macro_vars.cfg

```
variable_min_toolchange_z       : 15.0 # Be safe and dont scratch the bed
variable_park_travel_speed      : 450  # Travel a bit faster to avoid stringing
```

## Calibrating the unit
Now that the baseline software is set up, the unit needs to be calibrated. There are 2 key calibrations that need to be done before the unit allows you to print.
1. Lane rotation distance
2. Bowden length per lane

Detailed instructions can be found here: https://github.com/moggieuk/Happy-Hare/wiki/MMU-Calibration-TypeB . The below will outline the mandatory minimum steps to calibrate the unit.

### Lane rotation distance calibration
Before you start with this calibration, remove all bowden tubes from the rear of the dry boxes going to the combiner. Add a short length of PTFE tubing (50mm is enough) to the first lane dry box exit. This will help guide the filament and not grind on the ecas connector. Then proceed to load some filament and execute the below instructions to grip it and feed it.

**Step 1:** Select the gate, load filament and move 200mm (or more) till the filament is showing from the unit exit ECAS connector.
```
MMU_SELECT GATE=0
MMU_TEST_MOVE MOVE=200 # run this once or more till you get filament out from the dry box.
```
**Step 2:** Cut the end of the filament poking out from the short ptfe tube attached to the dry box flush with the ptfe tube. Run the below command:
```
MMU_TEST_MOVE MOVE=100
```
Cut the filament again flush with the ptfe tube attached to the dry box. Measure it. Now use the below command to calibrate the rotation distance of the box.
```
MMU_CALIBRATE_GEAR MEASURED=102.5 #102.5 is an example value and should correspond to the value measured.
```
**Step 3:** Repeat for all lanes
```
MMU_SELECT GATE=1 # then 2,3,4, etc etc.
MMU_TEST_MOVE MOVE=200 # run this once or more till you get filament out from the dry box.
..... cut the filament at the box ptfe tube exit .....
MMU_TEST_MOVE MOVE=100
MMU_CALIBRATE_GEAR MEASURED=102.5 #102.5 is an example value and should correspond to the value measured.
```
**Note:** If you observe significant variation between gates, double check that your BMG gears are centred with the filament path and that the tension is approximately as equal as possible between them.


Rotation distance calibration is now done! 

### Bowden tube calibration
Now remove the short PTFE tube from the rear of the dry box and connect the combiner and your bowden tube to the EMU Sync and from there to the toolhead. The final calibration is to measure the length of bowden tube from the lane filament parking location to the toolhead. These will be different for each lane, hence these need to be individually calibrated following the procedure below.

**IMPORTANT:** When calibrating bowden length for each lane, it is mandatory to keep the EMUSync manually held in the tension position, ie the two tubes brought towards each other. Why? The bowden calibration measures how much filament the lane feeds till the toolhead entry sensor is triggered. If the EMU Sync is left free, the springs can unpredictably compress/expand resulting in incorrect calibration. In addition, holding the EMU Sync in tension means the bowden length is slightly shorter. This helps avoid scenarios where the filament hits against the extruder if the lane rotation distance is slightly off.

```
MMU_SELECT GATE=0
MMU_CALIBRATE_BOWDEN
```
Repeat this for each lane, for example:
```
MMU_SELECT GATE=1 # then 2,3,4, etc etc.
MMU_CALIBRATE_BOWDEN
```
The unit calibration is now done and the unit is ready to be used!

## Next Steps
Finally you have to set up your slicer to recognise and operate with a multi-material unit. More instructions can be found here: https://github.com/moggieuk/Happy-Hare/wiki/Slicer-Setup

Optionally you can also set up:
1. Klipper screen integration: https://github.com/moggieuk/Happy-Hare/wiki/KlipperScreen
2. Mainsail / Fluid MMU panel: https://github.com/moggieuk/Happy-Hare/wiki/Mainsail-Fluidd-Integration
3. Spoolman: https://github.com/moggieuk/Happy-Hare/wiki/Spoolman-Support

## Seeking help

The Happy Hare discord is a great resource to help you going. Both in the general channel as well as the dedicated EMU channel. Link to the discord can be found here: https://discord.gg/aABQUjkZPk



