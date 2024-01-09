# IT-Mode
This is a tutorial on how to flash IT mode firmware to your LSI/Avago/Broadcom 9300-8i RAID Controller. It is largely based on [this guide](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/), but this repo contains extra information/commentary and makes all of the files used easily accessible.

Some reasons you may want to do this:
- Using software RAID or filesystem like [ZFS](https://itsfoss.com/what-is-zfs/ "What is ZFS?")
- Avoiding RAID Limitations/Compatibility

This video explains this topic very well: https://youtu.be/xEbQohy6v8U


## Required Items
- Circuit Board Jumper Pin
- USB Flash Drive

## Software Programs
- [Balena Etcher](https://etcher.balena.io/ "Balena Etcher")
- [Ventoy](https://www.ventoy.net/en/index.html "Ventoy")
- [Rufus](https://rufus.ie/en/ "Rufus")


## 1. Getting the files
Download these files from this repo and put them on a USB. They are necessary for this guide.

- sas3flash.efi      :      Flashing tool

Note: Other tutorials aimed at flashing different controllers may use sas2flash.efi instead. The 9300 does not work with sas2flash.efi and requires sas3flash.efi.
- sas93008i.bin      :      IT Mode Firmware

Note: This file was named "SAS9300_8i_IT.bin" in the linked tutorial. However, my system did not allow me to use the shift or underscore keys while in the UEFI shell, so I renamed it. This will not effect the file in any way.
- mptsas3.rom        :      Legacy BIOS OROM
- mpt3x64.rom        :      UEFI BIOS OROM

## 2. Preparing the Adapter
Here, you will put on the jumper pin as instructed by the tutorial. Do not remove it until it says so, which should be at the end of #3.
If you take the RAID controller off of the motherboard, you can also record the SAS Address of the controller. You can do this later, but if possible, I would recommend doing it now.

## 3. Booting the Server and Resetting the Adapter
Here, boot into the system's UEFI shell. This can be done in one of two ways:
### Option 1. 
  - This can be accessed through the UEFI menu.
### Option 2. 
  - If you cannot do Option #1, you can also access the shell with a USB flash drive formatted into a bootable Free-DOS drive using Rufus. You also must create an EFI/Boot/ folder filepath inside of the USB drive and add the bootx64.efi file into the Boot folder. Formatting the drive to Free-DOS will delete all files on the drive, so make sure to add all of the files back onto the drive.

#### After completing one of these options, proceed below:

```sas3flash.efi -listall```
- This command shows you your SAS Address as mentioned earlier. Record the SAS Address if you haven't already. It also tests that the system can correctly get/use the files on the flash drive.

```sas3flash.efi -f sas93008i.bin -noreset```
- Again, the tutorial has a different name for the sas93008i.bin file and uses this command:```sas3flash.efi -f SAS9300_8i_IT.bin -noreset```. You can rename the file to the tutorial name or any name you'd like, as long as you change the command accordingly.


## 4. Flashing the Controller
```sas3flash.efi -o -e 7```

- Erases the card's NVRAM

```sas3flash.efi -f SAS9300_8i_IT.bin -b mptsas3.rom -b mpt3x64.rom```
- This flashes the new UEFI and Legacy BIOS firmware with ROM

```sas3flash.efi -o -sasadd 50060XXXXXXXXXXXX```
- Here, replace the X's with your SAS Address. DO NOT USE HYPENS, SPACES, ETC.

```sas3flash.efi -list```
- Use this command to double check that all the information on your new HBA are correct.

## Resources
- MAIN TUTORIAL [How to flash a LSI SAS 3008 HBA (e.g. IBM M1215) to IT mode - ServeTheHome Forum](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/)
- [Crossflashing of LSI 9341-8i to LSI 9300-8i - ServeTheHome Forum](https://forums.servethehome.com/index.php?threads/crossflashing-of-lsi-9341-8i-to-lsi-9300-8i-success-but-no-smart-pass-through.3522/)
- [Detailed newcomers' guide to crossflashing LSI 9211/9300/9305/9311/9400/94xx HBA and variants - TrueNAS Forum](https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/)

## File Download Pages
- [LSI 3008 Firmware - DELL](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=jmx6t)
