.. _atm34evk:

###############
Atmosic ATM34xx
###############

********
Overview
********

The ATM34/e Wireless SoC Series is part of the Atmosic family of extremely low-power Bluetooth® system-on-chip (SoC) solutions. ATM34/e integrates a Bluetooth 6.0 compliant radio, ARM® Cortex® M33F application processor with ARM® TrustZone® enabled security features, and state-of-the-art power management to enable maximum lifetime in battery-operated devices.
For detailed product specifications and features, please refer to https://atmosic.com/products_atm34/.

*************
SoCs and EVKs
*************

.. _board:


==================  ==================  ==================  ==================  ========  ==========
SoC Part #          EVK Part #          Board List          On-chip             Package   Energy 
                                        <BOARD>             Flash                         Harvesting
==================  ==================  ==================  ==================  ========  ==========
ATM3405-2PCAQK      ATMEVK-3405-PQK-2   ATMEVK-3405-PQK-2   512KB               QFN 5x5   
ATM3425-2PCAQK      ATMEVK-3425-PQK-2   ATMEVK-3425-PQK-2   512KB               QFN 5x5   
ATM3430e-2WCAQN     ATMEVK-3430E-WQN-2  ATMEVK-3430e-WQN-2  1536KB              QFN 7x7   x 
==================  ==================  ==================  ==================  ========  ==========

================
Pin Multiplexing
================

The PinMux tool provides a graphical interface to summarize the mapping between Px pins and their supported functionalities. 
It is available at https://atmosic.com/public/Pinmux/index.html for all Atmosic Wireless SoCs.

***************
Getting Started
***************

Follow the instructions_ from the official Zephyr documentation on how to get started.

=============================
Connecting an ATMEVK on Linux
=============================

Special udev and group permissions are required by OpenOCD, which is the primary
debugger used to interface with Atmosic EVKs, to access the USB FTDI
SWD interface or J-Link OB.  When following Step 4 "Install udev rules, which
allow you ..." for Ubuntu_, add the following line to
`60-openocd.rules`::

 ATTRS{idVendor}=="1366", ATTRS{idProduct}=="1050", MODE="660", GROUP="plugdev", TAG+="uaccess"

.. _Ubuntu: https://docs.zephyrproject.org/3.7.0/develop/getting_started/index.html#install-the-zephyr-sdk

.. _instructions: https://docs.zephyrproject.org/3.7.0/develop/getting_started/index.html

===============================
Connecting an ATMEVK on Windows
===============================

The J-Link OB device driver must be replaced with the WinUSB driver to
become available as a USB device and usable by OpenOCD.
This can be done using Zadig.

Windows Administrator privileges are required to replace the driver.

Zadig can be obtained from:

https://github.com/pbatard/libwdi/releases

At the time of this writing, the latest version -- 2.4 -- can be
obtained using the following direct link.

https://github.com/pbatard/libwdi/releases/download/b721/zadig-2.4.exe

To replace the driver:

#. From the "Options" menu of Zadig, click "List all devices".
#. From the drop-down menu, find "BULK interface" corresponding to
   the Atmosic board.  It should show "jlink (v...)" as
   the current driver on the left.
#. Select "WinUSB (v...)" as the replacement on the right.
#. Click "Replace Driver"

Verify the successful installation of WinUSB by going to the Windows
Device Manager and confirm that the "BULK interface" shows
as such rather than the "J-Link driver".  (In Device Manager, expand the category
"Universal Serial Bus devices" and look for "BULK interface".)

*************************
Programming and Debugging
*************************

It is recommended to set the environment variables ZEPHYR_TOOLCHAIN_VARIANT to ``zephyr`` and ZEPHYR_SDK_INSTALL_DIR to the directory where Zephyr SDK is installed. For example, assuming the installed SDK version 0.16.8 is in the home directory, for reference, it will be like this in a bash shell environment: (use ``setenv`` in a C shell environment, or ``set`` for Windows)::

 export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
 export ZEPHYR_SDK_INSTALL_DIR=<$HOME/zephyr-sdk-0.16.8>

Applications for the Atmosic EVK boards can be built, flashed, and debugged using the familiar `west build` and `west flash`. 

The atm34evk boards require at least two images to be built: the SPE and the application.  SPE is the Secure Processing Environment, and the application typically resides in the non-secure (NSPE) portion.

The Atmosic SPE can be found under ``<WEST_TOPDIR>/openair/samples/spe``.

.. _variable assignments:

In the remainder of this document, substitute for ``<ZEPHYR_TOOLCHAIN_VARIANT>``, ``<ZEPHYR_SDK_INSTALL_DIR>``, ``<WEST_TOPDIR>``, ``<SPE>``, ``<APP>``, ``<APP_NAME>``, ``<MCUBOOT>``, ``<ATMWSTK>``, ``<BOARD>``, and ``<DEVICE_ID>`` appropriately.  For example::

 <ZEPHYR_TOOLCHAIN_VARIANT>: zephyr
 <ZEPHYR_SDK_INSTALL_DIR>: /absolute/path/to/zephyrSDK
 <WEST_TOPDIR>: /absolute/path/to/zephyrproject
 <SPE>: openair/samples/spe
 <APP>: zephyr/samples/bluetooth/peripheral
 <APP_NAME>: APP Name for ISP section
 <MCUBOOT>: bootloader/mcuboot/boot/zephyr
 <ATMWSTK>: PD50
 <BOARD>: ATMEVK-3430e-WQN-2
 <DEVICE_ID>: 900036960

* Use any board from the `board`_ list as ``<BOARD>``.
* <DEVICE_ID> is the unique ID from the J-Link device used to program. For FTDI, the format will be ATRDIXXXX.

=====================
Building and Flashing
=====================

----------------------------
Enabling a Random BD Address
----------------------------

Non-production ATM34 EVKs in the field have no BD address programmed in the secure journal.  On such boards, upon loading a BLE application, an assert error occurs with a message appearing on the console similar to the one below::

  ASSERT ERR(0) at <zephyrproject-root>/atmosic-private/modules/hal_atmosic/ATM34xx/drivers/eui/eui.c:129

To avoid this error, the BLE application must be built with an option to allocate a random BD address.  This can be done by adding ``-DCONFIG_ATM_EUI_ALLOW_RANDOM=y`` to the build options.

---------------
Build and Flash
---------------

Applications can be built with MCUboot or without the MCUboot option. If a device firmware update (DFU) is not needed, you can choose the option without MCUboot. If you require DFU, then the MCUboot option is required.

There are two main options as stated above:

---------------------
A. Non-MCUboot Option
---------------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Using ATMWSTKLIB (PD50, CPD200, and "" where "" represents LL)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Build the SPE:

::

  west build -p -s <SPE> -b <BOARD> -d build/<BOARD>/<SPE>

2. Build the Application:

Note: ``<BOARD>//ns`` is the non-secure variant of ``<BOARD>``.

Build the app with the non-secure board variant and the SPE configured as follows::

  west build -p -s <APP> -b <BOARD>//ns -d build/<BOARD>_ns/<APP> -- -DCONFIG_SPE_PATH=\"<WEST_TOPDIR>/build/<BOARD>/<SPE>\" -DCONFIG_ATMWSTKLIB=\"<ATMWSTK>\" -DCONFIG_ATM_EUI_ALLOW_RANDOM=y

Passing the path to the SPE is for linking in the non-secure-callable veneer file generated in building the SPE.

With this approach, each built image has to be flashed separately.  Optionally, build a single merged image by enabling ``CONFIG_MERGE_SPE_NSPE``, thereby minimizing the flashing steps::

  west build -p -s <APP> -b <BOARD>//ns -d build/<BOARD>_ns/<APP> -- -DCONFIG_SPE_PATH=\"<WEST_TOPDIR>/build/<BOARD>/<SPE>\" -DCONFIG_ATMWSTKLIB=\"<ATMWSTK>\" -DCONFIG_ATM_EUI_ALLOW_RANDOM=y -DCONFIG_MERGE_SPE_NSPE=y

3. Flashing the SPE and the Application:

Atmosic provides a mechanism to increase the legacy programming time called FAST LOAD. Apply the option ``--fast_load`` to enable the FAST LOAD.

Flash the SPE and the application separately if ``CONFIG_MERGE_SPE_NSPE`` was not enabled::

  west flash --device=<DEVICE_ID> --jlink --fast_load --verify -d build/<BOARD>/<SPE> --noreset
  west flash --device=<DEVICE_ID> --jlink --fast_load --verify -d build/<BOARD>_ns/<APP>

Alternatively, if ``CONFIG_MERGE_SPE_NSPE`` was enabled in building the application, the first step (programming the SPE) can be skipped.

-----------------
B. MCUboot Option
-----------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Using ATMWSTKLIB (PD50, CPD200, and "" where "" represents LL)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _MCUboot option:

1. Build the MCUboot and the SPE:

To build with MCUboot, for example, DFU is needed, first build MCUboot::

  west build -p -s <MCUBOOT> -b <BOARD>@mcuboot -d build/<BOARD>/<MCUBOOT> -- -DCONFIG_BOOT_SIGNATURE_TYPE_ECDSA_P256=y -DCONFIG_BOOT_MAX_IMG_SECTORS=512 -DDTC_OVERLAY_FILE="<WEST_TOPDIR>/zephyr/boards/atmosic/atm34evk/<BOARD>_mcuboot_bl.overlay"

and then the Atmosic SPE::

  west build -p -s <SPE> -b <BOARD>@mcuboot -d build/<BOARD>/<SPE> -- -DCONFIG_BOOTLOADER_MCUBOOT=y -DCONFIG_MCUBOOT_GENERATE_UNSIGNED_IMAGE=n -DDTS_EXTRA_CPPFLAGS=";"

Note that make use of "board revision" to configure our board partitions to work for MCUboot.  On top of the "revisions," MCUboot currently needs an additional overlay that must be provided through the command line to give it the entire SRAM.

2. Build the Application with MCUboot and SPE:

Build the application with MCUboot and SPE as follows::

  west build -p -s <APP> -b <BOARD>@mcuboot//ns -d build/<BOARD>_ns/<APP> -- -DCONFIG_ATM_EUI_ALLOW_RANDOM=y -DCONFIG_BOOTLOADER_MCUBOOT=y -DCONFIG_MCUBOOT_SIGNATURE_KEY_FILE=\"bootloader/mcuboot/root-ec-p256.pem\" -DCONFIG_SPE_PATH=\"<WEST_TOPDIR>/build/<BOARD>/<SPE>\" -DCONFIG_ATMWSTKLIB=\"<ATMWSTK>\" -DDTS_EXTRA_CPPFLAGS=";" -DEXTRA_CONF_FILE="<WEST_TOPDIR>/openair/doc/dfu/overlay-bt-dfu.conf"

This is somewhat of a non-standard workflow.  When passing ``-DCONFIG_BOOTLOADER_MCUBOOT=y`` on the application build command line, ``west`` automatically creates a signed, merged image (``zephyr.signed.{bin,hex}``), which is ultimately used by ``west flash`` to program the device.  The original application binaries are renamed with a ``.nspe`` suffixed to the file basename (``zephyr.{bin,hex,elf}`` renamed to ``zephyr.nspe.{bin,hex,elf}``) and are the ones that should be supplied to a debugger.

3. Flashing the MCUboot, SPE, and the Application:

Flash MCUboot

Atmosic provides a mechanism to increase the legacy programming time called FAST LOAD. Apply the option ``--fast_load`` to enable the FAST LOAD.::

   west flash --verify --device=<DEVICE_ID> --jlink --fast_load -d build/<BOARD>/<MCUBOOT> --noreset

Note that adding ``--erase_flash`` is an option to erase Flash if needed.

Flash the signed application image (merged with SPE)::

   west flash --verify --device=<DEVICE_ID> --jlink --fast_load -d build/<BOARD>_ns/<APP>
    
---------------------------
BLE Link Controller Options
---------------------------

When building a Bluetooth application (``CONFIG_BT``) the BLE driver component provides a statically linked BLE link controller library.  The BLE link controller sits at the lowest layer of the Zephyr Bluetooth protocol stack.  Zephyr provides the upper Bluetooth Host stack that can interface with BLE link controllers that conform to the standard Bluetooth Host Controller Interface specification.

To review how the statically linked controller library is used, please refer to the README.rst in modules/hal/atmosic/ATM34xx-5/drivers/ble/.

If the ATM34 entropy driver is enabled without CONFIG_BT=y (mainly for evaluation), the system still requires a minimal BLE controller stack.  Without choosing a specific stack configuration an appropriate minimal BLE controller will be selected.  This may increase the size of your application.

Note that developers cannot use ``CONFIG_BT_CTLR_*`` `flags`__ with the ATM34 platform, as a custom, hardware-optimized link controller is used instead of Zephyr's link controller software.

.. _CONFIG_BT_CTLR_KCONFIGS: https://docs.zephyrproject.org/latest/kconfig.html#!%5ECONFIG_BT_CTLR
__ CONFIG_BT_CTLR_KCONFIGS_

****************************************
Atmosic In-System Programming (ISP) Tool
****************************************

This repository comes with a tool called Atmosic In-System Programming (ISP) Tool for bundling all three types of binaries -- OTP NVDS, flash NVDS, and flash -- into a single binary archive.

+-------------+----------------------------------------------+
| Binary Type | Description                                  |
+=============+==============================================+
| .bin        | Binary file contains flash or NVDS data only |
+-------------+----------------------------------------------+

The ISP tool, which is also shipped as a stand-alone package, can then be used
to unpack the components of the archive and download them on a device.

===================
Python Requirements
===================

Support atm isp archive tool has to install specific python protobuf version 3.20.3 first.

To install with Openair requirement list file::

  pip install -r openair/scripts/requirements.txt

Or install with pip command directly::

  pip install grpcio-tools==1.47.0 protobuf==3.20.3

Note: This install operation will uninstall current python protobuf packages and reinstall python protobuf to version 3.20.3.
Depending on installation, a symbolic link might be required or enable virtual environment::
  
  sudo ln -s $(which python3) /usr/bin/python

======================
West atm_arch Commands
======================

Please use ``west atm_arch --help`` for usage details. 

==================================
Generate the .atm ISP Archive File
==================================

* replace ``<APP_NAME>`` with the application name.

Note: Build the required images using Option A or B from "Building and Flashing" section above before proceeding below.

With MCUboot::

  west atm_arch -o <BOARD>_<APP_NAME>.atm \
  -p build/<BOARD>_ns/<APP>/zephyr/partition_info.map \
  --app_file build/<BOARD>_ns/<APP>/zephyr/zephyr.signed.bin \
  --mcuboot_file build/<BOARD>/<MCUBOOT>/zephyr/zephyr.bin

Without MCUboot::

  west atm_arch -o <BOARD>_<APP_NAME>.atm \
  -p build/<BOARD>_ns/<APP>/zephyr/partition_info.map \
  --app_file build/<BOARD>_ns/<APP>/zephyr/zephyr.bin \
  --spe_file build/<BOARD>/<SPE>/zephyr/zephyr.bin

========================================
Show and Flash the .atm ISP Archive File
========================================

show command::

  west atm_arch -i <BOARD>_<APP_NAME>.atm --show

flash command::

  west atm_arch -i <BOARD>_<APP_NAME>.atm \
  --openocd_pkg_root atmosic-private/modules/hal_atmosic/ATM34xx \
  --burn

**************************
Programming Secure Journal
**************************

The secure journal is a dedicated block of RRAM that has the property of being a write-once, append-only data storage area that replaces traditional OTP memory. This region is located near the end of the RRAM storage array at 0x8F800– 0x8FEEF (1776 bytes).

The secure journal data updates are controlled by a secure counter (address ratchet). The counter determines the next writable location at an offset from the start of the journal. An offset greater than the counter value is writable while any offset below or equal to the counter is locked from updates. The counter can only increment monotonically and cannot be rolled back. This provides the immutability of OTP as well as the flexibility to append new data items or override past items using a find the latest TLV search.

The west extension command `secjrnl` is provided by the Atmosic HAL to allow for easy access and management of the secure journal on supported platforms.

The tool provides a help command that describes all available operations through::

 west secjrnl --help

======================
Dumping Secure Journal
======================

To dump the secure journal, run the command::

 west secjrnl dump --device <DEVICE_ID>

This will dump all the TLV tags located in the secure journal.

=====================================
Appending a Tag to the Secure Journal
=====================================

To append a new tag to the secure journal::

 west secjrnl append --device <DEVICE_ID> --tag=<TAG_ID> --data=<TAG_DATA>

* replace ``<TAG_ID>`` with the appropriate tag ID (Ex: ``0xde``)
* replace ``<TAG_DATA>`` with the data for the tag. This is passed as a string. To pass raw byte values format it like so: ``\0xde\0xad\0xbe\0xef``. As such, ``--data="data"`` will result in the same output as ``--data="\0x64\0x61\0x74\0x61``.

The secure journal uses a find latest search algorithm to allow overrides. If the passed tag should NOT be overridden in the future, add the flag ``--locked`` to the append command. See the following section for more information regarding locking a tag.

NOTE: The ``append`` command  does NOT increment the ratchet. The newly appended tag is still unprotected from erasing.

==================
Locking Down a Tag
==================

The secure journal provides a secure method of storing data while still providing options to update the data if needed. However, there are key data entries that should never be updated across the life of the device (e.g. UUID).
This support is provided by software and can be enabled for a tag by passing the ``--locked`` to the command when appending a new tag.

It is important to understand, that once a tag is **locked** (and ratcheted), the specific tag can never be updated in the future - Appending a new tag of the same value will be ignored.

==================================================
Erasing Non-ratcheted Data From the Secure Journal
==================================================

Appended tags are not ratcheted down. This allows for prototyping with the secure journal before needing to lock down the TLVs. To support prototyping, you can erase non-ratcheted data easily through::

 west secjrnl erase --device <DEVICE_ID>

=========================
Ratcheting Secure Journal
=========================

To ratchet data, run the command::

 west secjrnl ratchet_jrnl --device <DEVICE_ID>

This will list the non-ratcheted tags and confirm that you want to ratchet the tags. Confirm by typing 'yes'.

NOTE: This process is non-reversible. Once ratcheted, that region of the secure journal cannot be modified.

**************************
Viewing the Console Output
**************************

===============
Linux and macOS
===============

For Linux or macOS hosts, monitor the console output with a simple terminal program, such as::

  screen /dev/ttyACM<#> 115200 or 
  screen /dev/tty.usbmodem<UNIQUE_ID#> 115200

On Linux OS, the serial console will appear as a USB device (``/dev/ttyACM<#>``).  Use the following 
command to identify the correct port for the serial console. When the EVK is connected, two serial ports will be added. 
The user will need to test each one to determine where the message output is displayed::

 ls /dev/ttyACM*
  /dev/ttyACM0
  /dev/ttyACM1

On macOS, the serial console will appear as a USB device (``/dev/tty.usbmodem<UNIQUE_ID#>``).  Use the following 
command to identify the correct port for the serial console. When the EVK is connected, two serial ports will be added. 
The user will need to test each one to determine where the message output is displayed::

 ls /dev/tty.usbmodem*
  /dev/tty.usbmodem<UNIQUE_ID1>
  /dev/tty.usbmodem<UNIQUE_ID3>

=======
Windows
=======

The console output for the Atmosic ATM34xx is sent to the J-Link CDC UART port. When connected, two UART ports will be displayed. 
The user must test each one to determine where the message output appears.
To view the console output, use a serial terminal program such as PuTTY (available from
https://www.chiark.greenend.org.uk/~sgtatham/putty) to connect to the J-Link CDC UART port. Set the UART configuration to 115200/N/8/1.

**********
Zephyr DFU
**********

Please review the content for DFU Serial and OTA support at Zephyr_DFU_.

.. _Zephyr_DFU: https://github.com/Atmosic/openair/blob/HEAD/doc/dfu/dfu.rst