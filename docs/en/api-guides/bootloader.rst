Bootloader
=====================

Bootloader performs the following functions:

1. Minimal initial configuration of internal modules;
2. Select the application partition to boot, based on the partition table and ota_data (if any);
3. Load this image to RAM (IRAM & DRAM) and transfer management to it.

Bootloader is located at the address `0x1000` in the flash.

FACTORY reset
---------------------------
The user can write a basic working firmware and load it into the factory partition. 
Next, update the firmware via OTA (over the air). The updated firmware will be loaded into an OTA app partition slot and the OTA data partition is updated to boot from this partition. 
If you want to be able to roll back to the factory firmware and clear the settings, then you need to set :envvar:`CONFIG_BOOTLOADER_FACTORY_RESET`.
The factory reset mechanism allows to reset the device to factory settings:

- Clear one or more data partitions. 
- Boot from "factory" partition. 

:envvar:`CONFIG_BOOTLOADER_DATA_FACTORY_RESET` allows customers to select which data partitions will be erased when the factory reset is executed. 
Can specify the names of partitions through comma-delimited style with optional spaces for readability. (Like this: "nvs, phy_init, nvs_custom, ..."). 
Make sure that the names specified in the partition table and here are the same. 
Partitions of type "app" cannot be specified here.

:envvar:`CONFIG_BOOTLOADER_OTA_DATA_ERASE` - the device will boot from "factory" partition after a factory reset. The OTA data partition will be cleared.

:envvar:`CONFIG_BOOTLOADER_NUM_PIN_FACTORY_RESET`- The number of the GPIO input used to trigger a factory reset must be pulled low on reset. 

:envvar:`CONFIG_BOOTLOADER_HOLD_TIME_GPIO`- this is hold time of GPIO for reset/test mode (by default 5 seconds). The GPIO must be held low continuously for this period of time after reset before a factory reset or test partition boot (as applicable) is performed.

Partition table.::

	# Name,   Type, SubType, Offset,   Size, Flags
	# Note: if you change the phy_init or app partition offset, make sure to change the offset in Kconfig.projbuild
	nvs,      data, nvs,     0x9000,   0x4000
	otadata,  data, ota,     0xd000,   0x2000
	phy_init, data, phy,     0xf000,   0x1000
	factory,  0,    0,       0x10000,  1M
	test,     0,    test,    ,         512K
	ota_0,    0,    ota_0,   ,         512K
	ota_1,    0,    ota_1,   ,         512K

Boot from TEST firmware
------------------------
The user can write a special firmware for testing in production, and run it as needed. The partition table also needs a dedicated partition for this testing firmware (See `partition table`). 
To trigger a test app you need to set :envvar:`CONFIG_BOOTLOADER_APP_TEST`. 

:envvar:`CONFIG_BOOTLOADER_NUM_PIN_APP_TEST` - number of the GPIO input to boot TEST partition. The selected GPIO will be configured as an input with internal pull-up enabled. To trigger a test app, this GPIO must be pulled low on reset. 
After the GPIO input is deactivated and the device reboots, the old application will boot (factory or any OTA slot). 

:envvar:`CONFIG_BOOTLOADER_HOLD_TIME_GPIO` - this is hold time of GPIO for reset/test mode (by default 5 seconds). The GPIO must be held low continuously for this period of time after reset before a factory reset or test partition boot (as applicable) is performed.

Customer bootloader
---------------------
The current bootloader implementation allows the customer to override it. To do this, you must copy the folder `/esp-idf/components/bootloader` and then edit `/your_project/components/bootloader/subproject/main/bootloader_main.c`.
In the bootloader space, you can not use the drivers and functions from other components. The required functionality should be placed in the folder bootloader (note that this will increase its size).
It is necessary to monitor its size because there can be overlays in memory with a partition table leading to damage. The bootloader is limited to the partition table from the address `0x8000`.

