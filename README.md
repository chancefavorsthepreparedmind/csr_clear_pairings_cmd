# Bluetooth-USB-Bridge / HID Proxy Dongle Command Line Interface

This is the main repository for the Windows version of the command line interface.  
I will update this page to include links to ports to various platforms when there are any.

#### Dependencies

Libusb is necessary to compile the source. The included static libray is based on Libusb v1.0.24.
Reference: https://libusb.info/

#### Known Issues

There has been an issue with Windows 10 version 1809 and Libusb v.1.0.23 where Libusb failed to get a valid handle and could not access USB devices. For details see https://stackoverflow.com/questions/53761417/createfile-over-usb-hid-device-fails-with-access-denied-5-since-windows-10-180
If you experience any similar issues please let me know.

#### Questions

Just drop me a message at `jaygheezy@hotmail.com`

#### Description

This CLI will let you configure a few basic settings of a CSR based HID proxy dongle. Nothing fancy, just a few SET FEATURE requests.

#### Usage

This is the default usage statement displayed when no valid argument has been provided. The commands are mostly self-explanatory. I have added explanatory remarks for clarification.

	+++ Bluetooth-USB-Bridge / HID Proxy Dongle Command Line Interface +++

	Usage: btusbcmd [-option]

	Options:

	-hid2hci (switch from HID proxy to HCI mode)
	-clear_pairings
	-version

#### Explanation

`-hid2hci`  
Toggles the mode of operation. A power cycle is required to reset the module and re-enable HID mode.

`-clear pairings`  
Clears the list of paired devices.

`-version`  
Displays the current version of the CLI.