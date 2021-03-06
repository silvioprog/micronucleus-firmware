# Micronucleus USB Bootloader for ATtinies / Digisparks
### Version 2.5.1
[![License: GPL v2](https://img.shields.io/badge/License-GPLv2-blue.svg)](https://www.gnu.org/licenses/gpl-2.0)
[![Commits since latest](https://img.shields.io/github/commits-since/ArminJo/micronucleus-firmware/latest)](https://github.com/ArminJo/ServoEasmicronucleus-firmwareing/commits/master)
[![Hit Counter](https://hitcounter.pythonanywhere.com/count/tag.svg?url=https://github.com/ArminJo/micronucleus-firmware)](https://github.com/brentvollebregt/hit-counter)

Micronucleus is a bootloader designed for AVR ATtiny microcontrollers with a minimal usb interface, cross platform libusb-based program upload tool, and a strong emphasis on bootloader compactness. To the authors knowledge this is, by far, the smallest USB bootloader for AVR ATtiny.<br/>
**The V2.0 release is a complete rewrite of the firmware and offers significant improvements over V1.x.**<br/>
Due to the many changes, also the upload tool had to be updated. The V2.0 upload tool is backwards compatible to the V1.X tool, though.

![Digisparks](pictures/Digisparks.jpg)

# Usage
The bootloader allows **uploading of new firmware via USB**. In its usual configuration it is invoked at device power or on reset and will identify to the host computer. If no communication is initiated by the host machine within a given time (default are 6 seconds), the bootloader will time out and enter the user program, if one is present.<br/>
For proper timing, the command line tool should to be started on the host computer **before** the bootloader is invoked / the board attached.<br/>
The bootloader resides in the same memory as the user program, since the ATtiny series does not support a protected bootloader section. Therefore, special care has to be taken not to overwrite the bootloader if the user program uses the self programming features. The bootloader will patch itself into the reset vector of the user program. **No other interrupt vectors are changed**.<br/>
Please invoke the command line tool with "micronucleus -help" for a list of available options.

# Driver installation
For Windows you must install the **libusb driver** before you can program the board. Download it [here](https://github.com/digistump/DigistumpArduino/releases/download/1.6.7/Digistump.Drivers.zip), open it and run `InstallDrivers.exe`.
Clean Micronucleus devices without uploaded user program will not time out and allow sufficient time for proper driver installation. Linux and OS X do not require custom drivers.

# Updated configuration for Digispark boards
**The new [Digistump AVR version](https://github.com/ArminJo/DigistumpArduino) shrinks generated code size by 5 to 15 percent**. Just replace the old Digispark board URL **http://digistump.com/package_digistump_index.json** (e.g.in Arduino *File/Preferences*) by the new  **https://raw.githubusercontent.com/ArminJo/DigistumpArduino/master/package_digistump_index.json** and install the latest **Digistump AVR Boards** version.<br/>
![Boards Manager](https://github.com/ArminJo/DigistumpArduino/blob/master/pictures/Digistump1.6.8.jpg)<br/>

# Update the bootloader to the new version
To **update** your old, flash consuming **bootloader**, you have 2 choices.
1. Using the [new Digistump board manager](https://github.com/ArminJo/DigistumpArduino#update-the-bootloader) (see above).<br/>
2. Run one of the Windows [scripts](https://github.com/ArminJo/micronucleus-firmware/tree/master/utils)
like e.g. the [Burn_upgrade-t85_default.cmd](utils/Burn_upgrade-t85_default.cmd). The internal mechanism is described [here](https://github.com/ArminJo/micronucleus-firmware/blob/master/firmware/upgrades/README.md).

### If you want to burn the bootloader to an **ATtiny87** or **ATtiny167** with avrdude, you must use the [avrdude.config file](https://raw.githubusercontent.com/ArminJo/micronucleus-firmware/master/windows_exe/avrdude.conf) in `windows_exe` where [ATtiny87](https://github.com/ArminJo/micronucleus-firmware/blob/master/windows_exe/avrdude.conf#L15055) and [ATtiny167](https://github.com/ArminJo/micronucleus-firmware/blob/master/windows_exe/avrdude.conf#L15247) specifications are added.

# Configuration overview
If not otherwise noted, the OSCCAL value is calibrated (+/- 1%) after boot for all ATtiny85 configurations
| Configuration | Free FLASH | Boot-<br/>loader size | Non default config flags set |
|---------------|-----------------|-----------------|------------------------------|
| t85_aggressive<br/><br/>It works for my Digispark boards without any problems :-) | 6780 | 1364 | [Do not provide calibrated OSCCAL, if no USB attached](/firmware/configuration/t85_aggressive/bootloaderconfig.h#L220), [ENABLE_UNSAFE_OPTIMIZATIONS](#enable_unsafe_optimizations)<br/>Relying on calibrated 16MHz internal clock stability, not using the 16.5 MHz USB driver version with integrated PLL. This causes the main memory saving. |
|  |  |  |  |
| t85_default | 6650 | 1528 | - |
| t85_entry_on_powerOn | 6586 | 1570 | [ENTRY_POWER_ON](#entry_power_on-entry-condition), LED_MODE=ACTIVE_HIGH |
| t85_entry_on_powerOn_<br/>fastExit | 6586 | 1578 | [ENTRY_POWER_ON](#entry_power_on-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), LED_MODE=ACTIVE_HIGH |
| **t85_entry_on_powerOn_<br/>activePullup_fastExit**<br/>[recommended configuration](#recommended-configuration) | 6586 | 1582 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), LED_MODE=ACTIVE_HIGH |
| t85_entry_on_powerOn_<br/>activePullup_fastExit_noLED| 6586 | 1564 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit) |
| t85_entry_on_powerOn_<br/>pullupAt0 | 6586 | 1558 | [ENTRY_POWER_ON](#entry_power_on-entry-condition), USB_CFG_PULLUP_IOPORTNAME + USB_CFG_PULLUP_BIT |
| t85_fastExit | 6586 | 1554 | [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), LED_MODE=ACTIVE_HIGH |
| t85_entry_on_reset_<br/>activePullup | 6586 | 1582 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_EXT_RESET](#entry_d_minus_pullup_activated_and_entry_ext_reset-entry-condition), LED_MODE=ACTIVE_HIGH |
| t85_entry_on_reset_<br/>activePullup_fastExit | 6586 | 1582 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_EXT_RESET](#entry_d_minus_pullup_activated_and_entry_ext_reset-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit),<br/>AUTO_EXIT_MS=15000 Bootloader timeout increased to 15 seconds (if connected to USB),<br/>LED_MODE=ACTIVE_HIGH. |
|  |  |  |  |
| t88_default | 6714 | 1442 | LED_MODE=ACTIVE_HIGH |
| **t88_entry_on_powerOn_<br/>activePullup_fastExit**<br/>[recommended configuration](#recommended-configuration) | 6650 | 1478 |  [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), LED_MODE=ACTIVE_HIGH |
|  |  |  |  |
| t167_default | 14970 | 1322 | - |
| t167_entry_on_powerOn_<br/>no_pullup | 14842 | 1366 | [ENTRY_POWER_ON](#entry_power_on-entry-condition), [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition), LED_MODE=ACTIVE_HIGH |
| **t167_entry_on_powerOn_<br/>activePullup_fastExit**<br/>[recommended configuration](#recommended-configuration) | 14842 | 1378 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), LED_MODE=ACTIVE_HIGH |
| t167_entry_on_reset_<br/>activePullup_fastExit | 14842 | 1378 | [ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_EXT_RESET](#entry_d_minus_pullup_activated_and_entry_ext_reset-entry-condition), [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit), <br/>AUTO_EXIT_MS=15000 Bootloader timeout increased to 15 seconds (if connected to USB),<br/>LED_MODE=ACTIVE_HIGH. |
|  |  |  |  |
| Nanite841 |  | 1566 |  |
| BitBoss |  | 1560 |  |
| t84_default |  | 1492 |  |
|  |  |  |  |
| m168p_extclock |  | 1482 |  |
| m328p_extclock |  | 1482 |  |

### Legend
- [ENTRY_POWER_ON](#entry_power_on-entry-condition) - Only enter bootloader on power on, not on reset or brownout.
- [ENTRY_EXT_RESET](#entry_ext_reset-entry-condition) - Only enter bootloader on reset, not on power up.
- [ENTRY_D_MINUS_PULLUP_ACTIVATED](#entry_d_minus_pullup_activated_and_entry_power_on-entry-condition) - Only enter if pull-up connected and powered.
- [FAST_EXIT_NO_USB_MS=300](#fast_exit_no_usb_ms-for-fast-bootloader-exit) - If not connected to USB (e.g. powered via VIN) the user program starts after 300 ms (+ initial 300 ms) -> 600 ms.
- [ENABLE_SAFE_OPTIMIZATIONS](#enable_safe_optimizations) - jmp 0x0000 does not initialize the stackpointer.
- [LED_MODE=ACTIVE_HIGH](https://github.com/ArminJo/micronucleus-firmware/blob/master/firmware/main.c#L527) - The built in LED flashes during the 5 seconds of the bootloader waiting for commands.

# Configuration Options

## [`FAST_EXIT_NO_USB_MS`](/firmware/configuration/t85_fastExit/bootloaderconfig.h#L195) for fast bootloader exit
If the bootloader is entered, it requires minimum 300 ms to initialize USB connection (disconnect and reconnect). 
100 ms after this 300 ms initialization, the bootloader receives a reset, if the host application wants to program the device.<br/>
The 100 ms time to reset may depend on the type of host CPU etc., so I took 200 ms to be safe.<br/>
This configuration waits for 200 ms after initialization for a reset and if no reset detected it exits the bootloader and starts the user program.<br/>
With this configuration the **user program is started with a 500 ms delay after power up or reset** even if we do not specify a special entry condition.

## [`ENTRY_POWER_ON`](/firmware/configuration/t85_entry_on_powerOn/bootloaderconfig.h#L108) entry condition
The `ENTRY_POWER_ON` configuration adds 18 bytes to the ATtiny85 default configuration.<br/>
The content of the `MCUSR` is copied to the `GPIOR0` register to enable the user program to evaluate it and then cleared to prepare for next boot.<br/>
In this configuration **a reset will immediately start your user program** without any delay.

## [`ENTRY_EXT_RESET`](/firmware/configuration/t85_entry_on_reset_activePullup/bootloaderconfig.h#L122) entry condition
The ATtiny85 has the bug, that it sets the `External Reset Flag` also on power up. To guarantee a correct behavior for `ENTRY_EXT_RESET` condition, it is checked if only this flag is set **and** `MCUSR` is cleared before start of user program. The latter is done to avoid bricking the device by fogetting to reset the `PORF` flag in the user program.<br/>
For ATtiny167 it is even worse, it sets the `External Reset Flag` and the `Brown-out Reset Flag` also on power up.<br/>
The content of the `MCUSR` is copied to the `GPIOR0` register before clearing it. This enables the user program to evaluate its original content.<br/>
**ATTENTION! If the external reset pin is disabled, this entry mode will brick the board!**

## [`ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON`](/firmware/configuration/t85_entry_on_powerOn_activePullup_fastExit/bootloaderconfig.h#L138) entry condition
Activate the bootloader only if the D- pin is high, i.e. a pull-up resistor is attached and powered **and** we have an `ENTRY_POWER_ON` condition (ref. described above).<br/>
Useful if the pull-up is powered by USB V+ and NOT ATtiny VCC to save power.
In this case often a schottky diode is connected between USB V+ and VCC (5V).<br/>
The `ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_POWER_ON` configuration adds 54 bytes to the ATtiny85 default configuration.<br/>
The content of the `MCUSR` is copied to the `GPIOR0` register to enable the user program to evaluate it and then cleared to prepare for next boot.<br/>
In this configuration **a power up with USB disconnected or a reset will immediately start your user program** without any delay.<br/>
This configuration flag overrides `START_WITHOUT_PULLUP` (using both flags is useless and wastes bytes).

## [`ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_EXT_RESET`](/firmware/configuration/t85_entry_on_reset_activePullup/bootloaderconfig.h#L138) entry condition
Activate the bootloader only if the D- pin is high, i.e. a pull-up resistor is attached and powered **and** we have an `ENTRY_EXT_RESET` condition.<br/>
Useful if the pull-up is powered by USB V+ and NOT ATtiny VCC to save power.
In this case often a schottky diode is connected between V* and VCC.<br/>
The `ENTRY_D_MINUS_PULLUP_ACTIVATED_AND_ENTRY_EXT_RESET` configuration adds 54 bytes to the ATtiny85 default configuration.<br/>
The content of the `MCUSR` is copied to the `GPIOR0` register to enable the user program to evaluate it and then cleared to prepare for next boot.<br/>
In this configuration **a power up with USB disconnected or a reset will immediately start your user program** without any delay.<br/>
This configuration flag overrides `START_WITHOUT_PULLUP` (using both flags is useless and wastes bytes).

## [`START_WITHOUT_PULLUP`](/firmware/configuration/t85_default/bootloaderconfig.h#L220)
### To support low energy applications
The `START_WITHOUT_PULLUP` configuration adds 16 to 18 bytes for an additional check. It is only required if the bootloader can be entered without a pull-up attached activated at the D- pin. Since this check was contained by default in all pre 2.0 versions, it is obvious that **it can also be used for boards with a pull-up**.

## [`ENABLE_SAFE_OPTIMIZATIONS`](/firmware/crt1.S#L99)
This configuration is referenced in the [Makefile.inc](/firmware/configuration/t85_fastExit/Makefile.inc#L18) file and will [disable the restoring of the stack pointer](https://github.com/ArminJo/micronucleus-firmware/blob/master/firmware/crt1.S#L102) at the start of program, whis is normally done by reset anyway. These optimization disables reliable entering the bootloader with `jmp 0x0000`, which you should not do anyway - better use the watchdog timer reset functionality.<br/>
- Gains 10 bytes.

## [`ENABLE_UNSAFE_OPTIMIZATIONS`](/firmware/crt1.S#L99)
- Includes [`ENABLE_SAFE_OPTIMIZATIONS`](#enable_safe_optimizations).
- The bootloader reset vector is written by the host and not by the bootloader itself. In case of an disturbed communication the reset vector may be wrong -but I have never experienced it.

You have a slightly bigger chance to brick the bootloader, which reqires it to be reprogrammed by [avrdude](windows_exe) and an ISP or an Arduino as ISP. Command files for this can be found [here](/utils).

## [Recommended](/firmware/configuration/t85_entry_on_powerOn_activePullup_fastExit) configuration
For ATtiny85, ATtiny88 and ATtiny167 the recommended configuration is *t85_entry_on_powerOn_activePullup_fastExit*, *t88_entry_on_powerOn_activePullup_fastExit.hex* and *t167_entry_on_powerOn_activePullup_fastExit.hex* respectively.

This configuration has the following features:
- A hardware reset (e.g., Digispark P5 shorted to ground or brownout happened) will always immediately run the user program.
- If no user program is loaded, the LED quickly flashes indefinitely.
- At power-on, we distinguish 3 cases.
  1 The board is connected to an USB. In this case, the bootloader attempts to load a program from the host (*entry_on_powerOn* / `ENTRY_POWER_ON`) and starts the user program after 6 s timeout or after upload completion. The built-in LED flashes while the bootloader is waiting for user program upload and during the upload phase (`LED_MODE=ACTIVE_HIGH`).
  2 The board is **not** connected to an USB. In this case, the bootloader starts the already installed user program after 600 milliseconds (`FAST_EXIT_NO_USB_MS=300` means *Fast exit of bootloader after 300+300 ms*). The 600 ms are required to decide if there is an USB host attached or not.
  3 The **board is modified** and not connected to an USB **or the pull-up resistor is removed**. In this case, the pull-up is inactive and the bootloader immediately starts the installed user program (without startup delay, without hang-up and without affecting the internal LED at boot). This is referred to the `ENTRY_D_MINUS_PULLUP_ACTIVATED` part of the configuration flags. The builtin LED remains off in this case (`LED_MODE=ACTIVE_HIGH` has no effect here).

### Hex files for these configuration are available in the [releases](/firmware/releases) and [upgrades](/firmware/upgrades) folders.

## Create your own configuration
You can easily create your own configuration by adding a new *firmware/configuration* directory and adjusting *bootloaderconfig.h* and *Makefile.inc*. Before you run the *firmware/make_all.cmd* script, check the arduino directory path in the [`firmware/SetPath.cmd`](https://github.com/ArminJo/micronucleus-firmware/firmware/SetPath.cmd#L1) file.<br/>
If changes to the configuration lead to an increase in bootloader size, it may be necessary to change the bootloader start address as described [above](#computing-the-values) or in the *Makefile.inc*.
Feel free to supply a pull request if you added and tested a previously unsupported device.

# Compile instructions for the bootloader are [here](firmware#compiling)

## Computing the values
The actual memory footprint for each configuration can be found in the file [*firmware/build.log*](firmware/build.log).<br/>
Bytes used by the mironucleus bootloader can be computed by taking the data size value in *build.log*, 
rounding it up to the next multiple of the page size which is e.g. 64 bytes for ATtiny85 and 128 bytes for ATtiny176.<br/>
Subtracting this (+ 6 byte for postscript) from the total amount of memory will result in the free bytes numbers.
- Postscript are the few bytes at the end of programmable memory which store tinyVectors.

E.g. for *t85_default.hex* with the new compiler we get 1548 as data size. The next multiple of 64 is 1600 (25 * 64) => (8192 - (1600 + 6)) = **6586 bytes are free**.<br/>
In this case we have 52 bytes left for configuration extensions before using another 64 byte page.<br/>
For *t167_default.hex* (as well as for the other t167 configurations) with the new compiler we get 1436 as data size. The next multiple of 128 is 1536 (12 * 128) => (16384 - (1536 + 6)) = 14842 bytes are free.

# Bootloader memory comparison of different releases for [*t85_default.hex*](firmware/releases/t85_default.hex).
- V1.6  6012 bytes free
- V1.11 6330 bytes free
- V2.3  6522 bytes free
- V2.04 6522 bytes free
- V2.5  **6586** bytes free

# USB device manager entry / disconnect from USB
To avoid periodically disconnect->connect if no sketch is loaded and an unknown USB Device (Device Descriptor Request Failed) entry in device manager when entering user program, **the bootloader finishes without an active disconnect from USB**.<br/>
This means that you still can see a libusb-win32 decive / Digispark Bootloader in the Device manager, even when it is not alive, since your program has taken over the control of the CPU.<br/>
This behavior is compatible to the old v1 micronucleus versions, which also do not disconnect from the host.
**You can avoid this by actively disconnecting from the host by pulling the D- line to low for around 300 milliseconds.**<br/>
E.g a short beep at startup with tone(3, 2000, 200) will pull the D- line low and keep the module disconnected.

# Measured Digispark (fast 64 MHz PLL clock) supply current
| Current | Voltage | Clock | Configuration | 
|-:|-:|-:|:-:|
| **20 mA** | 5 V | 16.5 MHz | Standard Hardware |
| 13 mA   | **3.8 V** | 16.5 MHz | " |
| 16 mA   | 5 V |    8 MHz | " |
| 11 mA   | 5 V |    1 MHz | " |
| 6 mA    | 3.8 V |  1 MHz | " |
| 4.5 mA  | 3 V |  1 MHz | " |
| **7.7 mA** | 5 V | 1 MHz | SLEEP_MODE_PWR_DOWN + ADC disabled |
| 17 mA   | 5 V | 16.5 MHz | Voltage regulator removed |
| 14 mA   | 5 V | 16.5 MHz | Power LED and voltage regulator removed |
| 9.3 mA  | 5 V |    8 MHz | " |
| 4.3 mA  | 5 V |    1 MHz | " |
| **14.3 mA** | 5 V | 16.5 MHz | **Power LED, voltage regulator removed + USB D- pull-up reconnected and powered directly at VCC** (loop with delay)|
| 9.5 mA  | **3.8 V** | 16.5 MHz | " |
| 8.3 mA  | 5 V |    8 MHz | " |
| 7.5 mA  | 5 V |    8 MHz | All Hardware changes + empty loop + Timer and ADC disabled |
| **3.0 mA** | 5 V | 1 MHz | **All Hardware changes (loop with delay)** |
| 2.6 mA    | **3.8 V** | 1 MHz | " |
| 2.9 mA    | 5 V |  1 MHz | All Hardware changes + empty loop |
| 2.4 mA    | 5 V |  1 MHz | All Hardware changes + empty loop + Timer and ADC disabled |
| 230 �A    | 5 V |  1 MHz | All Hardware changes + SLEEP_MODE_PWR_DOWN |
| **27 �A** | 5 V |  1 MHz | All Hardware changes + SLEEP_MODE_PWR_DOWN + ADC disabled + Watchdog enabled |
| 25 �A     | 3 V |  1 MHz | " |
|  **7 �A** | 5 V |  1 MHz | All Hardware changes + **SLEEP_MODE_PWR_DOWN + ADC disabled + BOD disabled** + Watchdog enabled |
| 5.5 �A   | 3.8 V | 1 MHz | " The Watchdog draws around 5 �A |

BOD can only be disabled by setting fuses via ISP programmer](https://www.google.de/search?q=arduino+as+isp) and a connecting adapter.
For reprogramming the fuses, you can use [this script](https://github.com/ArminJo/micronucleus-firmware/blob/master/utils/Write%2085%20Fuses%20E1%20DF%20FE%20-%20Digispark%20default%20without%20BOD%20and%20Pin5.cmd).<br/>

# Current per device @ 5 volt
| Current | Device |
| -:|:-:|
|   3 mA | Voltage regulator (1.5 mA at 3.8 V)|
|   2 mA | Power LED |
|   1 mA | USB D- pull-up |
| 14.3 mA | CPU + timer @16.5 MHz |
|  14 mA | CPU + timer @16 MHz |
|   8 mA | CPU + timer @8 MHz |
|   3 mA | CPU + timer @1 MHz |
|  20 �A | BOD |
| 212 �A | ADC |

With fast PLL Clock and standard fuses, the **start-up time from sleep is around 64ms and requires 2/3 of regular CPU power**!<br/>
If we use the longest sleep time of 8 seconds and an empty loop, this result in an **average current consumption of 23 �A** (1 year with a 200 mAh button cell 2032).<br/>
The start-up time from sleep can be reduced (at own risk of unstable clock) to 5 ms using [this fuse settings](https://github.com/ArminJo/micronucleus-firmware/blob/master/utils/Write%2085%20Fuses%20C1%20DF%20FE%20-%20Digispark%20default%20without%20BOD%20and%20Pin5%20and%20fast%20startup.cmd).
This results in an average current consumption of **9 �A** (2.5 years with a 200 mAh button cell 2032).<br/>
This long startup time can be dramatically reduced to 6 clock cycles by [changing fuses to use the internal 8Mhz clock](https://github.com/ArminJo/micronucleus-firmware/blob/master/utils/Write%2085%20Fuses%20E2%20DF%20FF%20-%20ISP%20Mode%20%3D%208MHz%20without%20BOD%20and%20Pin5.cmd), but this **disables the possibility to program the Digispark board via USB** and enables availability of whole memory for your program.

# Reducing current of digispark bords for low power applications
Here is an [Instructable](https://www.instructables.com/Reducing-Battery-Power-Consumption-for-Digispark-A/) covering the topic.
### Disabling the power LED
Break the copper wire that connects the power LED to the diode with a knife or remove / disable the 102 resistor.
### Removing the VIN voltage regulator
First lift the outer pins with the help of a solder iron and a pin. Then solder the big connector and remove the regulator. For small regulators, use much solder and heat up all 3 pins together, then remove it.
### Disconnecting the USB D- Pullup resistor (marked 152) from 5 volt (VCC)
Break the copper wire on the side of the resistor that points to the ATtiny.<br/>
**This disables the USB interface** and in turn the possibility to program the Digispark board via USB. To **enable it again**, but still save power, **connect the resistor (marked 152) directly to the USB V+** that is easily available at the outer side of the schottky diode.<br/>
The diode and its correct sides can be found by using a continuity tester. One side of this diode is connected to pin 8 of the ATtiny (VCC) and Digispark 5V. The other side is connected to the USB V+.<br/>
Now the USB pull-up resistor is only activated if the Digispark board is connected to USB e.g. during programming.

![Final power reduction](https://github.com/ArminJo/Arduino-OpenWindowAlarm/blob/master/pictures/Final-Version-Detail_annotated.jpg)

## Another example for current reduction

<img src="pictures/modding_front.png" width="300" title="Modding front"/><img src="pictures/modding_back.png" width="300" title="Modding back"/>

### Moddings:

- Connect A to C (A is the right terminal of the 1kohm resistor; C is USB V+)
- Cut B with a cutter (power supply rail interconnecting the 1kohm resistor with 5V)
- Connect C with D (C is USB V+, D is the left terminal of the green LED resistor)
- Remove the 78M05 linear regulator G
- Connect E with F (short the two input/output terminals of the removed linear regulator)
- Cut H with a cutter (power supply rail connecting 5V with the output of the linear regulator)

Now we have 3 power supply variants:
1 Use the USB to upload a sketch (the green LED is lighted) - Disconnect 5V and VIN.
2 Use 5V - GND to test the device with D- pull-up (the green LED is lighted) - Disconnect USB and VIN.
3 Use VIN - GND for standard operation in low current consumption. The power supply can be 3V to 5V (the green LED can be used as debug output of PB3). - Disconnect USB and 5V.


# Pin layout
### ATtiny85 on Digispark

```
                       +-\/-+
 RESET/ADC0 (D5) PB5  1|    |8  VCC
  USB- ADC3 (D3) PB3  2|    |7  PB2 (D2) INT0/ADC1 - default TX Debug output for ATtinySerialOut
  USB+ ADC2 (D4) PB4  3|    |6  PB1 (D1) MISO/DO/AIN1/OC0B/OC1A/PCINT1 - (Digispark) LED
                 GND  4|    |5  PB0 (D0) OC0A/AIN0
                       +----+
  USB+ and USB- are each connected to a 3.3 volt Zener to GND and with a 68 Ohm series resistor to the ATtiny pin.
  On boards with a micro USB connector, the series resistor is 22 Ohm instead of 68 Ohm.
  USB- has a 1.5k pull-up resistor to indicate a low-speed device.  
  USB+ and USB- are each terminated on the host side with 15k to 25k pull-down resistors.
```

### ATtiny167 on Digispark pro
Digital Pin numbers in braces are for ATTinyCore library

```
                  +-\/-+
RX   6 (D0) PA0  1|    |20  PB0 (D8)  0 OC1AU
TX   7 (D1) PA1  2|    |19  PB1 (D9)  1 OC1BU - (Digispark) LED
     8 (D2) PA2  3|    |18  PB2 (D10) 2 OC1AV
INT1 9 (D3) PA3  4|    |17  PB3 (D11) 4 OC1BV USB-
           AVCC  5|    |16  GND
           AGND  6|    |15  VCC
    10 (D4) PA4  7|    |14  PB4 (D12)   XTAL1
    11 (D5) PA5  8|    |13  PB5 (D13)   XTAL2
    12 (D6) PA6  9|    |12  PB6 (D14) 3 INT0  USB+
     5 (D7) PA7 10|    |11  PB7 (D15)   RESET
                  +----+
  USB+ and USB- are each connected to a 3.3 volt Zener to GND and with a 51 Ohm series resistor to the ATtiny pin.
  USB- has a 1k5 pull-up resistor to indicate a low-speed device.
  USB+ and USB- are each terminated on the host side with 15k to 25k pull-down resistors.

```
# Revision History
### Version 2.5.1 - work in progress
- Saved 28 bytes by `#define USB_CFG_DESCR_PROPS_STRINGS 1`. See #10, thanks to [nerdralph](https://github.com/nerdralph).
- Use 12 byted by re enabling push and pop of Y register in *usbdrvasm165.inc*. See #11.
- Swapped D+ and D- for t88 to support MH-ET LIVE Tiny88 boards.
- Renamed the different pre-built configurations.

### Version 2.5
- Saved 2 bytes by removing for loop at leaveBootloader().
- Saved 2 bytes by defining __DELAY_BACKWARD_COMPATIBLE__ for _delay_ms().
- Saved 22 bytes by converting USB_INTR_VECTOR in *usbdrvasm165.inc* from ISR with pushes to a plain function.
- Saved 2 bytes by improving small version of usbCrc16 in *usbdrvasm.S*.
- Saved 4 bytes by inlining usbCrc16 in *usbdrvasm.S*.
- Renamed `AUTO_EXIT_NO_USB_MS` to `FAST_EXIT_NO_USB_MS` and implemented it.
- New configurations using `FAST_EXIT_NO_USB_MS` set to 300 ms.
- Light refactoring and added documentation.
- No USB disconnect at bootloader exit. This avoids "Unknown USB Device" entry in device manager.
- Gained 128 byte for `t167_default` configuration.

- Fixed wrong condition for t85 `ENTRYMODE==ENTRY_EXT_RESET`.
- ATtiny167 support with MCUSR bug/problem at `ENTRY_EXT_RESET` workaround.
- `MCUSR` handling improved.
- no_pullup targets for low energy applications forever loop fixed.
- `USB_CFG_PULLUP_IOPORTNAME` disconnect bug fixed.
- New `ENTRY_POWER_ON` configuration switch, which enables the program to start immediately after a reset.
- Copy `MCUSR` to `GPIOR0` and clear it on exit for all `ENTRY_POWER_ON` and `ENTRY_EXT_RESET` configurations.
