Notes on INF files

					Introduction

INF files are in general not a feature in Windows CE, however to configure a USB device to load the appropriate device driver corresponding to a particular VID and PID requires access to a registry editor which (unlike windows) is not always available on a windows CE platform. Using INF files to configure the driver to load properly and initialise some device settings helps users from a windows background to use the drivers without the need for a registry editor or a complete recompile of the driver with each new VID and PID.
On loading the driver parses the INF file (which must be located in the Windows directory) to extract settings such as VID/PID and Latency timer. It then populates the registry with the appropriate values to enable all future removal and insertion of the device to load the FTDI driver.
Please note: INF files are for INSTALLATION ONLY. After a particular device has been installed the inf file becomes redundant and changes to the inf file will not affect the device.

					Altering the INF file 

The following sections guide you through the steps to alter the FTDIPORT INF file. Please do not alter the name of this file as the driver will not recognise any other file name.

WARNING: YOU MUST NOT CHANGE THE GENERAL FORMAT OF THE INF FILE - DOING THIS COULD CAUSE YOUR INSTALLATION TO HALT THE DEVICE AND THEREFORE REQUIRE A REBOOT. BEFORE PERFORMING THE FOLLOWING PROCEDURE IT IS RECOMMENDED YOU BACK UP ALL ESSENTIAL DATA. If you have any doubt on how to alter the INF file then contact support who can help with this process.

					Alternative VID/PID

By default the VCP driver for windows CE (ftdi_ser.dll) will work with devices of VID and PID 0x0403 and 0x6001. Therefore if you do not copy the INF file to the Windows directory the driver will create registry settings for a USB device with VID and PID of 0x0403 and 0x6001.

The INF files provided in this distribution package correspond to a VID and PID of 0x0403 and 0x6001/0x6010. If you have a device with an alternative VID and PID (use USBView on a desktop Windows PC to find out device VID and PID) you will need to edit the INF file to correspond to your device.

The INF file format is a cut down version of the Windows equivalent. On loading (after you type in the driver name in the install driver dialog box) the driver will read the INF file (if it is present in the \Windows\ directory) and use the settings you provide. 

The section [FtdiHw] must be altered to correspond to your device VID and PID (you can obtain this using USBView on a desktop PC).

For example for a device with VID 0x1234 and PID 0x4321:
Original (0x0403/0x6001) INF entry
[FtdiHw]
%VID_0403&PID_6001.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_0403&PID_6001

New entry with a VID of 0x1234 and PID of 0x4321
[FtdiHw]
%VID_1234&PID_4321.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_1234&PID_4321

Multiple VIDs and PIDs

The INF file now supports multiple VID and PIDs - if you want to install 2 devices with different VIDs and PIDs you can have the following entry in the INF file:
[FtdiHw]
%VID_0403&PID_6001.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_0403&PID_6001
%VID_0403&PID_6010.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_0403&PID_6010
%VID_0403&PID_6011.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_0403&PID_6011
%VID_0403&PID_6012.DeviceDesc%=FtdiPort232,FTDIBUS\COMPORT&VID_0403&PID_6012

					Additional INF Settings 

Additional settings are under the [FtdiPort232.NT.HW.AddReg] section of the INF file. All settings are added to the registry and take effect when the port is opened.

LatencyTimer

This sets the latency on an open (CreateFile) to a device - 1..255

HKR,,"LatencyTimer",0x00010001,25 - this will set the latency timer to 25

InitialIndex 

This is the initial index that will be used for this COM device. Currently the driver will assign COM ports 0 to 9 but if you require 
the COM port to start at 2 you could have the following setting in the INF file:
HKR,,"InitialIndex",0x00010001,2

ConfigDataFlags

Various settings for control lines
HKR,,"ConfigDataFlags",0x00010001,33

ConfigDataFlags:	bit 0: Ignore all set/clear DTR requests
			bit 1: Ignore all set/clear RTS requests
			bit 2: DTR initial state high (on open device)
			bit 3: RTS initial state high (on open device) 
			bit 4: Ignore set DTR low on close device
			bit 5: Ignore set RTS low on close device
So for example if you want all requests to DTR set or clear to be ignored and the initial state of DTR on open to be high use a ConfigDataFlags value of 5 decimal.

ConfigData

Aliasing of baud rate can be used to set a particular baud rate not supported by a legacy application. You in effect replace a standard baud rate for the one you require so the application calls set baud rate for say 9600 but your alias table has set this to 10000 therefore your com port will behave at 10000 baud. For more info please see app note "Configuring FT232BM Baud Rates"
Use the following setting in the INF file to allow for aliasing of baud rates.
HKR,,"ConfigData",1,01,00,3F,3F,10,27,88,13,C4,09,E2,04,71,02,38,41,9c,80,4E,C0,34,00,1A,00,0D,00,06,40,03,80,00,00,d0,80

Persistant unplug/replug.
A feature to allow the COM port to still be available on an unplug/replug or suspend resume has been added. To enable this feature you must set bit 2 of the second byte of the ConfigData flags as follows:
HKR,,"ConfigData",1,01,04,00,00,10,27,88,13,C4,09,E2,04,71,02,38,41,9c,80,4E,C0,34,00,1A,00,0D,00,06,40,03,80,00,00,d0,80
PLEASE NOTE in addition to the above setting you must have unique serial numbers programmed into the devices. This is required to determine which data to attach to which device on an unplug/replug.

Support for CF USB Host
A registry setting may be required to support CF host cards (for example the Ratoc REX-CFU1) the following registry setting should be used in this case
HKR,,"ConfigData",1,01,02,00,00,10,27,88,13,C4,09,E2,04,71,02,38,41,9c,80,4E,C0,34,00,1A,00,0D,00,06,40,03,80,00,00,d0,80
(bit 1 of the second byte of the configuration flag data).

In/Out Transfer Sizes

These 2 settings adjust the bulk transfer size. If you are having problems getting the driver to work (such as unable to read from the port) - try setting the InTransfer size to 64 and working upwards in multiples of 64 to find a suitable value.
HKR,,"InTransferSize",0x00010001,64
HKR,,"OutTransferSize",0x00010001,4096

BulPriority/Ex

HKR,,"BulkPriority",0x00010001,2 - sets the priority of the reading thread of the driver Valid range 0(High priority) to 7(Low priority). Setting this value may cause your hardware to stop functioning therefore take care when setting these values and backup any data you may need. This setting uses the legacy SetThreadPriority function.
HKR,,"BulkPriorityEx",0x00010001,251 - sets the priority of the reading thread of the driver Valid range 0(High priority) to 255(Low priority). Setting this value may cause your hardware to stop functioning therefore take care when setting these values and backup any data you may need. This setting uses the current CeSetThreadPriority function.

Modem Emulation
As per application note AN232B-09 with the exception of using the decimal value instead of hexadecimal value for example HKR,,"EmulationMode",0x00010001,8360 will switch on DTR, DCD, RI and DSR emulation on. This is required to be placed in the \Drivers\USB\FTDI_DEVICE_XXXX\YYYYYYYY where YYYYYYYY is the device serial number or in the \Drivers\USB\FTDI_DEVICE_XXXX if the device has no serial number.

Active Sync Monitoring
Requests have been made to call the CeEventHasOccurred when certain modem lines change within the device.
This requires a separate thread of execution when the device is unopened. The following registry setting is required to turn on this functionality
HKR,,"ActiveSyncSetting",0x00010001,256128
The value corresponds to the following:
0xYYYYYYZZ where YYYYYY is the sleep value in milliseconds between checking for line status changes - a sensible value here is 1000ms. ZZ corresponds to which bit of the line status you want to monitor. 
0x10 = CTS
0x20 = DTR
0x40 = RI
0x80 = DCD
Normally you would have a value of 0x80 here for DCD but you may require to change these.
Again the value in the registry/INF file must be in decimal.
Active sync monitoring only occurs when the device is plugged in and unopened. The driver does NOT monitor the lines and call CeEventHasOccurred if the device is opened and in use. Once the device has closed again the driver will continue to monitor the line setup in the registry.

					Increase COM port count

Windows CE 4.2 and below has no support for only COM0 to 9 which can limit certain hardware. To increase the COM port count requires editing the binary driver file supplied and altering a registry setting as per the following procedure.
The method to increase COM ports is considered an advanced option. If you do not feel comfortable editing binary files then please contact support with your request for additional ports and they can schedule a suitable release to you.
The default prefix of the VCP driver is COM - this means COM0: to COM9: can be used in your applications. However it as been noted that on some systems the COM port prefix has been used by other devices and you may therefore require additional COM ports. To allow this you must change the Prefix setting in the INF file. Remember the INF file will ONLY be read on initial installation and NOT any time after this. 
HKR,,"Prefix",0x00010000,"COM"
In addition to changing the prefix value in the INF file prior to installing you must also change some entries in the supplied dll. 
	1. Open the dll in a binary text editor such as visual c++
	2. Search for COM_Init.
	3. Change the COM part to your own prefix so if you wanted a prefix MOC then COM_Init would become
	   MOC_Init.PLEASE MAKE SURE YOU DO NOT ADD ANY DATA TO THE DLL BY INSERTING CHARACTERS. OVERWRITE THE
	   COM PART - DO NOT INSERT.
	4. do the same for the following entries
	     	COM_Deinit
		COM_Open
		COM_Close
		COM_Read
		COM_Write
		COM_Seek
		COM_PowerDown
		COM_PowerUp
		COM_IOControl
	5. Save the new dll.

Remember that now all your com ports will have your own unique prefix and you must use this instead to open the ports. For example if your prefix is MOC then you will use MOC0: to open port 0 and so on...
It is also important that you choose a prefix that does not clash with any other device to ensure you get the maximum 10 allowable ports.

Unique port settings
If your usb device has a serial number attached to it then an additional registry entry will be created under the \Drivers\USB\FTDI_DEVICE_XXXX\<serial number> entry. This will have a copy of the original device settings (such as Latency and ConfigData). This enables you to set custom settings for each device individually if you require.

Alternatively to have a device install automatically (without the need for INF files) you can use the suggested registry settings in the registry settings.txt file.

[HKEY_LOCAL_MACHINE\Drivers\USB\LoadClients\1027_24577] - your VID and PID must be reflected in this setting (1027_24577 corresponds to a VID and PID of 0x0403(1027 decimal) and 0x6001(24577 decimal)).

