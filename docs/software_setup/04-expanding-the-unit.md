# EMU Happy Hare Hardware and Software Setup Guide

This is a provisional sofware setup guide for the EMU using Happy Hare v3. This guide is meant to be read in conjunction with the Happy Hare setup guide as found here: https://github.com/moggieuk/Happy-Hare/wiki

## Table of Contents

- [Expanding the unit with more lanes](#expanding-the-unit-with-more-lanes)

## Expanding the unit with more lanes
Beyond the electrical connections, the below steps need to be undertaken when adding additional lanes to the EMU unit.

**mmu.cfg**

Add an additional MCU block for each new lane:
```
[mcu mmuN]
canbus_uuid: UUID 
canbus_interface: can0
```

Add an additional boards pin block:
```
[board_pins mmuN]
mcu: mmuN
aliases:

    MMU_GEAR_UART_N=PA15,
    MMU_GEAR_STEP_N=PD0,
    MMU_GEAR_DIR_N=PD1,
    MMU_GEAR_ENABLE_N=PD2,
    MMU_GEAR_DIAG_N=,

    MMU_PRE_GATE_N=PB7,
    MMU_POST_GEAR_N=PB5,

    EJECT_BUTTON_N=PB6,
```

**mmu_hardware.cfg:**

Update the below to reflect your current lane count:
```
[mmu_machine]
num_gates: 8
```
Add additional stepper definitions, where N is the lane number:
```
...
[tmc2209 stepper_mmu_gear_N]
uart_pin: mmuN:MMU_GEAR_UART_N

[stepper_mmu_gear_N]
step_pin: mmuN:MMU_GEAR_STEP_N
dir_pin: mmuN:MMU_GEAR_DIR_N
enable_pin: !mmuN:MMU_GEAR_ENABLE_N
...
```

Define the additional pre-gate and post gear sensors (pre-stepper, post-stepper)
```
[mmu_sensors]
...
pre_gate_switch_pin_N: ^mmuN:MMU_PRE_GATE_N
post_gear_switch_pin_N: ^mmuN:MMU_POST_GEAR_N
...
```

Increase your neopixel chain count to match the total number of lanes multipled by 2
```
[neopixel mmu_leds]
chain_count: NN			# Number lanes x2
```

Update the mmu_led's entry and exit LED numbers
```
[mmu_leds unit0]
exit_leds:   neopixel:mmu_leds (1,3,5,7,9,11,13,15, NN)
entry_leds: neopixel:mmu_leds (2,4,6,8,10,12,14,16, NN)
```

**mmu_eject_buttons_hw.cfg**

Define the additional eject buttons in the mmu_eject_buttons_hw.cfg file:
```
[gcode_button mmu_eject_button_N]
pin: mmuN:EJECT_BUTTON_N
press_gcode: _MMU_EJECT_BUTTON GATE=N
```

**mmu_macro_vars.cfg:**

Add an additional tool change gcode macro at the end of the file for each new lane:
```
[gcode_macro TN]
gcode: MMU_CHANGE_TOOL TOOL=N
```

**emu_macros.cfg**

Add additional temperature sensor, fan and BME sensor definitions and update the custom fan control macro to use them

Finally dont forget to execute the 2 calibrations for the new lanes - MMU_CALIBRATE_BOWDEN and  MMU_CALIBRATE_GEAR as descibed in the calibration section.

```
[temperature_sensor Lane_N]
sensor_type: BME280
i2c_address: 118
i2c_mcu: mmuN
i2c_software_scl_pin: mmuN:PB3
i2c_software_sda_pin: mmuN:PB4

[temperature_sensor Lane_N_onboard]
sensor_type: temperature_mcu
sensor_mcu: mmuN
min_temp: 0
max_temp: 130

[fan_generic emu_fan_N]
pin: mmuN:PA0
max_power: 1
kick_start_time: 0.5

[gcode_macro MMU_FAN_CFG]
...
variable_sensors: ".....,Lane_N_onboard"
variable_fans:    "......,emu_fan_N"
...
```



