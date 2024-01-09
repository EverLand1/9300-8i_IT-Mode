# IT-Mode
This is a tutorial on how to flash IT mode firmware to your LSI/Avago/Broadcom 9300-8i RAID Controller. It is largely based on [this guide](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/), but this repo contains extra information and makes all of the files used easily accessible. 

## Required Items
- Circuit Board Jumper Pin
- USB Flash Drive

## Software Programs
- [Balena Etcher](https://etcher.balena.io/ "Balena Etcher")
- [Rufus](https://rufus.ie/en/ "Rufus")


## 1. Getting the files
- sas3flash.efi      :      Flashing tool
- sas93008i.bin      :      
(This file was named "SAS9300_8i_IT.bin" in the linked tutorial. However, my system did not allow me to use the shift or underscore keys while in the UEFI shell, so I renamed it. This will not effect the file in any way.)
- mptsas3.rom        :      Legacy BIOS OROM
- mpt3x64.rom        :      UEFI BIOS OROM

## 2. Preparing the Adapter
Here, you will put on the jumper pin as instructed by the tutorial. Do not remove it until it says so. If you take the RAID controller off of the motherboard, you can also record the SAS Address of the controller. You can do this later, but if possible, I would recommend doing it now.

## 3. Booting the Server and Resetting the Adapter
Here, boot into the system's UEFI shell. This can be accessed through the UEFI menu or with a USB flash drive formatted into a bootable Free-DOS drive using Rufus. You also must create an EFI/Boot/ folder filepath inside of the USB drive and add the bootx64.efi file into the Boot folder.

```sas3flash.efi -list```
- This command shows you your SAS Address as mentioned earlier. Record it is you haven't already. It also test that the system can correctly get/use the files.

```sas3flash.efi -f sas93008i.bin -noreset```
- Again, the tutorial has a different name for the sas93008i.bin file and uses this command:```sas3flash.efi -f SAS9300_8i_IT.bin -noreset```. You can rename the file to the tutorial name or any name you'd like, as long as you change the command accordingly.


## 4. Flashing the Controller
```sas3flash.efi -o -e 7```

```sas3flash.efi -f SAS9300_8i_IT.bin -b mptsas3.rom -b mpt3x64.rom```

```sas3flash.efi -o -sasadd 50060XXXXXXXXXXXX```
