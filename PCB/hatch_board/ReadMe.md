## EMU Wiring Hatch Board

The **EMU Wiring Hatch Board** is a dedicated PCB designed for EMU to make wiring and assembly easier and more convenient. It replaces the printed hatch ([Box_wiring_hatch](https://github.com/DW-Tas/EMU/blob/main/STL/Box/%5Ba%5D_Box_wiring_hatch.stl)), eliminates the need to solder the Filamentalist assembly wires to the JST connectors, and removes the need to use silicone to seal gaps around the JST connector.

The EMU Wiring Hatch Board was designed and developed by <a href="https://github.com/kashine6">kashine6</a>.

## Key Features

1. **Eliminates the need to solder wires directly to JST connectors**
   - Simplifies assembly
   - Reduces soldering complexity
   - Improves maintainability, as the Filamentalist assembly can be easily unplugged from inside the base unit

2. **Simplified pre-stepper sensor connections**
   - The two pre-stepper switches no longer need to be soldered in parallel. Instead, solder the sensor and ground wires to each switch individually, then plug them into the board separately.

3. **Easier box sealing**
   - Removes the need to use silicone to seal the JST connector on the hatch cover.

4. **Wire labeling**
   - Wire labels on the PCB silkscreen reduce the chance of wiring errors during assembly.

<img src="./images/1.jpg" width=800>

## BOM variations vs. baseline EMU BOM
Due to the PCB being thinner than the printed box wiring hatch, **the 2x M3x8mm FHCS screws are now replaced by 2x M3x6 BHCS/SHCS screws**.

If buying pre-assembled, the corresponding straight through hole PCB header JST connectors in the BOM are no longer needed (1x JST XH 8 pin plug and 1x JST XH 4 pin plug).

If making your own and soldering the JST headers, you'll need the below additions:
- 3x JST XH Series Straight Through Hole PCB Header, 4 Contact
- 3x JST XH Series Straight Through Hole PCB Header, 2 Contact 


## This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg

## License clarification regarding non-commercial use:
The non-commercial aspect of this license is for cases where EMU is the product, not the use of EMU to create products.<br/>
I.e. If you wish to sell EMU as a product, you would need to seek a commercial license before doing so. </br>
It is NOT intended to prevent the use of EMU with a printer that you use to provide commercial services. If you want to run EMU in your print farm, go right ahead.

