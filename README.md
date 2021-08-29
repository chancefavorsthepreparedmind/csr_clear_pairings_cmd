# Bluetooth-USB-Bridge / HID Proxy Dongle Command Line Interface

This is the main repository for the Windows version of the command line interface.  
I will update this page to include links to ports to various platforms when there are any.

#### Dependencies

Libusb is necessary to compile the source. The included static libray is based on Libusb v1.0.24.
Reference: https://libusb.info/

#### Questions

Just drop me a message at `jaygheezy@hotmail.com`

#### Usage

This is the default usage statement displayed when no valid argument has been provided. The commands are mostly self-explanatory. I have added explanatory remarks for clarification.

    +++ Blusb configuration tool +++

    Usage: blusb_cmd [-option] [-optional parameter] [filename]

    Options:

    -read_matrix
    -read_pwm
    -write_pwm [value_USB value_BT] (Valid range: 0-255)  
    -read_debounce  
    -write_debounce [value] (Valid range: 1-255)  
    -read_macros  
    -write_macros [filename]  
    -read_layout [-no_print] [-names]  
    -write_layout [filename]  
    -configure_layout [-update filename]  
    -read_version  
    -update_firmware [filename]  
    -enter_bootloader  
    -exit_bootloader  
    -layout_dec_to_hex [filename]  
    -macros_dec_to_hex [filename]  
    -h, --help, /?

#### Explanation

`-read_matrix`  
Displays row and column of a key pressed.

`-read_pwm`  
Displays brightness value for USB mode and BT mode.

`-write_pwm`  
Writes brightness value for USB mode and BT mode.  
Example: `-write_pwm 50 150` will set the brightness values to 50 in USB mode and 150 in BT mode.

`-read_debounce`  
Displays current debounce period in ms.

`-write_debounce`  
Sets the debounce period in ms to a value between 0 and 255.  
Example: `-write_debounce 15` will set the debounce period to 15ms.

`-read_macros`  
Displays the macro table currently stored in the EEPROM and prompts to enter a file name to store the macro table in a file. If no macros have been programmed yet, no table is displayed.

`-write_macros`*`filename`*  
Loads a macro table stored in *filename*.  
Example: `-write_macros randommacrotable.dat` will transfer the macro table stored in *randommacrotable.dat* to the controller's EEPROM.

`-read_layout`*`-no_print -names`*  
Displays all layers currently configured, prompts for file name and saves to a file.

You can format the output by (not) providing optional parameter *-names*. Default output is a table of hexadecimal key code values stored at each position of the key matrix. If *-names* is provided, key code names are displayed instead of hexadecimal values. Make sure your command line window provides enough horizontal spacing or the table will not be displayed correctly when names are displayed instead of numbers.  
If optional parameter *-no_print* is provided, layout data will only be displayed and not saved to a file.  
Optional parameters may be specified in arbitrary order.  
Example: `-read_layout`*`-no_print -names`* will work just the same as `-read_layout`*`-names -no_print`*.   

`-write_layout`*`filename`*  
Transfers the layout data contained in *filename* to the controller's flash memory.  
Example: `-write_layout randomlayoutfile.dat` will transfer the layout data contained in *randomlayoutfile.dat* to the controller.

`-configure_layout`*`-update filename`*  
Run layout configuration wizard to create a new layout from scratch or, if optional parameter *-update* is provided, update the layout stored in *filename*.

`-read_version`  
Display the version of the firmware currently installed on the controller.

`-update_firmware`*`filename.hex`*  
Performs a firmware update based on firmware binary file *filename.hex*. Once all firmware data has been transferred, you will be prompted to press a key to reboot the device.  
Example: `-update_firmware newversion.hex`  

`-layout_dec_to_hex`*`filename`*  
If you have a layout file that represents key codes in decimal format, you can use this convenience function to convert the values to hexadecimal format and store them in a new file. `-macros_dec_to_hex` serves the same purpose with regard to files containing macro tables.

#### Layout and macro configuration

The CLI allows configuration of the key map and macro key table. To change either, it will suffice to edit the example files and then upload them with the CLI. <br>
*Note: By default the macro table is empty. Hence reading the (empty) macro table with the CLI will only return an error message.*

The layout files contain preconfigured key maps. The file type is plain text, the file ending may be chosen arbitrarily.

Editing the layout files works thus:

Unless you need to reconfigure all keys, it will suffice to edit the default key map files. You can use any text editor you like.

For every layer, there are 160 values that comprise the key matrix. The key matrix consists of a maximum of 20 columns and 8 rows, i.e. for each row there are 20 columns, which gives 8x20 positions in total.
We will start counting from 0, so the first row is row 0, the last row is row 7, and likewise the first column is column 0, the last column is column 19.
The first 20 positions belong to row 0, the second 20 positions belong to row 1, the next 20 belong to row 2, and so forth. If you need to change a couple of values, look up the given values in the layout header file (`layout.h`). This file contains the values representing all the keys that can be mapped. All you need do is change a value, save the file and transfer the contents of the file to the controller using the CLI.

#### Understanding the internal representation of special key codes (s.a. modifiers)

There are special key codes that are 16 bit in length, e.g. the modifier keys. The lower byte represents the key type (to determine the position of the key value within the HID key report to be sent by the microcontroller to the USB host).
For modifiers, the higher byte is always 0x01 (hexadecimal). The Alt_left key, for instance, is represented by the value 0x04, hence the complete key code generated internally by the microcontroller will be 0x0104.
This scheme is not intuitive and may seem confusing, mainly due to the following two aspects: the higher byte for character keys is always 0x00, i.e. zero, while it is non-zero for other key types, for one. Second, the representation of modifier keys in the key map file is not identical with the value given in the HUT table (which the layout header file, `layout.h` is based on). 
Macro keys are defined according to the same scheme as modifier keys, i.e. they are 16 bit in length (cp. `layout.h`).

Alternatively, there is also the possibility of loading an existing key map file into memory and using the CLI to change the given key map. For this method, a second keyboard is necessary or you will not be able to enter keys that have not been mapped yet.
Any changes made with the CLI are not rendered effective at runtime. Configuring a key map and transfering of a key map to the controller comprise two consecutive steps. This precludes the event of ending up with a garbled key map if things go awry.

Starting the CLI by typing `blusb_cmd -configure_layout`*`keymap_filename.bin`* will load the specified key map file into memory. After the desired changes have been made, the new configuration can be saved to a new file or the existing file may be overwritten.

The CLI also allows tweaking the debounce period. The default value is 7ms. You can probably get away with less, say, 3ms. Conversely, if you should encounter bouncing at 7ms, you can also crank it up. For instance, Unicomp recommends 15ms.
*Note: the ms value given does not represent the actual or total delay that occurs from when a key is pressed until a key press is registered. The actual delay depends on other factors besides the debounce period.*
