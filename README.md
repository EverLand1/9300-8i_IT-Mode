# Flash 9300-8i RAID Controller to IT Mode (HBA)

### A 9340-8i was used for this guide. Likely also compatible with IBM/Lenovo M1215 and SAS 3008 models, but have not tested them. This is not a guide for flashing to IR mode or from IR mode to IT mode.

This is an updated guide on how to flash IT mode firmware to your LSI/Avago/Broadcom 9300-8i RAID Controller. It is largely based on [this tutorial](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/) from 2016. Most of the screenshots and techniques used in this guide are from that tutorial and [others](#resources), but this repo compiles them, includes additional updated information/commentary, and makes all of the files used easily accessible.

## Why would I want to flash my RAID controller?
There are multiple reasons why you may want to flash your RAID controller to make it a Host Bus Adapter (HBA):
- Using software RAID or filesystems including [ZFS](https://itsfoss.com/what-is-zfs/ "What is ZFS? - It's FOSS") and [BTRFS](https://itsfoss.com/btrfs/ "What is BTRFS - It's FOSS")
- Avoiding RAID Limitations/Compatibility

[Benefits of Using an HBA](https://www.truenas.com/community/resources/whats-all-the-noise-about-hbas-and-why-cant-i-use-a-raid-controller.139/)

[(Video) RAID vs HBA SAS controllers](https://youtu.be/xEbQohy6v8U)

[(Video) Hardware RAID Is Dead](https://www.youtube.com/watch?v=l55GfAwa8RI)

### Warning: Before proceeding, do note that this is a risky process. Issues with firmware flashes can render your card “bricked” and unusable/unrecoverable. I will not be held responsible if this happens to your card. By following this guide, you accept all risks of damaging your card.

## Required Items
- Circuit Board Jumper Pin
- USB Flash Drive
- [Rufus Software](https://rufus.ie/en/ "Rufus")
- Firmware Files (located in this repo)

## 1. Getting the Files
Download all five of these files from this repo and put them on a USB drive. They are necessary for this guide.

- bootx64.efi&emsp;        :&emsp;      UEFI Shell Boot File
- mptsas3.rom&emsp;        :&emsp;      Legacy BIOS OROM
- mpt3x64.rom&emsp;        :&emsp;      UEFI BIOS OROM
- sas3flash.efi&emsp;      :&emsp;      Flashing tool

&emsp; _Note: Other tutorials aimed at flashing older models including 9200 models may use sas2flash.efi instead. The 9300 does not work with sas2flash.efi and requires sas3flash.efi._
- SAS9300_8i_IT.bin&emsp;  :&emsp;      IT Mode Firmware

&emsp; _Note: Sometimes there can be trouble with using the "__" and "Shift" keys while in the UEFI shell. If any problems occur, change the name of this file to something without those characters and retry, adjusting the following commands accordingly._

## 2. Preparing the USB Drive
1. To access the UEFI shell later on with the required files, you must format your USB flash drive into a bootable Free-DOS drive using Rufus. Launch Rufus, select the correct drive under **"Device"**, select **"Free-DOS"** under **"Boot Selection"**, and click **"Start"**. This should only take a couple seconds.

#### &emsp;&emsp; WARNING: ALL FILES ON THE SELECTED DRIVE WILL BE DELETED.

&emsp;&emsp;&emsp;![Rufus Free-DOS](images/rufus.png)

2. After the drive has completed formatting to Free-DOS, create a folder on the root of the USB named "EFI".
3. Inside that folder, create another folder named "Boot".
4. Move the bootx64.efi file into the "Boot" folder. **(Resulting filepath is EFI/Boot/bootx64.efi)**
5. Now add all required files from this repo to the root of the USB.

#### &emsp;&emsp; Resulting filesystem after formatting drive in Rufus and adding the required files.

&emsp;&emsp;&emsp;![USB Files](images/USB.png)

## 3. Preparing the Adapter
1. Put the jumper pin on the RAID controller pins. Make sure to put them on the correct pins as there may be multiple to choose from. 

&emsp; _Note: I have seen several tutorials claim that this is not necessary. In my testing, however, I have not had success when using their methods, so this tutorial follows the approach of using a jumper pin. If you attempt to do it [that way](https://www.truenas.com/community/threads/how-to-flashing-lsi-sas-hba-controller-efi-uefi.78457/), please note that you must use slightly different commands. If you do not have success with that tutorial, you can fix it by beginning this guide from the beginning._

### DO NOT TAKE THE JUMPER PIN OFF UNTIL AFTER THE SYSTEM POWERS DOWN AT THE END OF SECTION 5.

&emsp;&emsp;&emsp;![Jumper Pin](images/jumper.jpg)

2. Record the SAS Address of the controller. DO THIS NOW. This may involve taking apart the RAID controller in case it is hidden. However, that should only require a Phillips screwdriver. The sticker should look like this:

&emsp;&emsp;&emsp;![SAS Address Sticker](images/sasaddress.jpg)

## 4. Booting the Server into UEFI Shell

1. Before powering on the machine, unplug any external drives that are on the system. This is not necessary, but it may make the process easier later.
2. Next, plug in the USB drive.
3. Access the BIOS/UEFI menu to make sure you can boot to the UEFI shell from this USB drive. This can be done through several ways, depending on your system's manufacturer, as well as the operating system or lack there of. It can usually be triggered by spamming a function key during startup.

&emsp;&emsp;- [This article](https://www.tomshardware.com/reviews/bios-keys-to-access-your-firmware,5732.html) includes UEFI/BIOS keys for popular brands, as well as Windows and Linux specific ways to access the UEFI menu.

5. Enable the system to boot to the drive in BIOS. Once again these settings vary depending on manufacturer. 

&emsp;&emsp;- "Secure Boot" to "Disabled"

&emsp;&emsp;- "Boot Mode" from "Legacy" to "UEFI" or "Auto"

  _Note: On my system, a Lenovo ThinkServer, this setting was found under "Boot Manager" then "Boot Mode". Again, manufacturer specific. THIS MAY NOT BE AVAILABLE OR NECESSARY ON ALL SYSTEMS._
  
5. Save these changes and restart the system.
6. Boot to the USB drive. If possible, you can access the boot menu during startup similarly to the BIOS menu. However, you can also go back to the BIOS menu and change the boot sequence to make the UEFI USB Drive the first device the system boots to upon startup. You will now be able to boot to the USB.

## 5. Resetting the Adapter
1. You should find yourself at a ```Shell>``` command line prompt. If you are experienced with using the terminal or are adventerous, here are some interesting [commands](https://www.sys-hint.com/3893-How-to-Use-UEFI-Interactive-Shell-and-Its-Common-Commands) you can play around with while in this shell.
2. Now, to find all available drives on the system, type ```map```. This may already show up when you are first brought into the shell.
3. Your USB drive should be named something along the lines of "Removable Hard Disk". If not, choose the first drive that shows up first. Mount to the alias of that drive. For example, if the drive is named ```fs0```, type ```fs0:``` (remember the colon). Your terminal prompt should now read ```fs0:\>``` as you have successfully mounted the drive.
4. Type ```dir``` to see the filesystem on that drive (Equivalent of using ```ls``` in Linux.). If you do not see the files you put on the USB drive, retry the previous commands beginning with ```map``` until you find the correct drive.
5. Type: ```sas3flash.efi -list```

&emsp;&emsp;&emsp;This shows the SAS Address as mentioned earlier. Record the SAS Address if you haven't already. 

**You may get an error stating "Controller is not operational. A firmware download is required." Type ```quit``` to exit that prompt and continue with this guide.**

&emsp;![List](images/list.png)

6. Type this command: ```sas3flash.efi -f SAS9300_8i_IT.bin -noreset```. After it completes, power down the system and take the jumper pin off of the RAID controller.

&emsp;&emsp;&emsp;This erases the running flash/bios on the RAID controller. 

&emsp;&emsp;![Noreset](images/noreset.jpg)

## 6. Flashing the Controller
1. Boot back into the UEFI shell through the USB drive.
2. Type this command: ```sas3flash.efi -o -e 7```

- Erases the card's NVRAM

&emsp;![Erase](images/erase.jpg)

3. Next, type: ```sas3flash.efi -f SAS9300_8i_IT.bin -b mptsas3.rom -b mpt3x64.rom```
- This flashes the new UEFI and Legacy BIOS firmware with ROM

&emsp;![Flash Firmware](images/flash.jpg)

4. Type: ```sas3flash.efi -o -sasadd XXXXXXXXXXXXXXXXX```
- Here, replace the X's with your SAS Address. DO NOT USE HYPENS, SPACES, ETC.
- This  flashes the SAS Address back onto the HBA

&emsp;![SASADD](images/sasadd.jpg)

5. Lastly, type this command:```sas3flash.efi -list```
- Verify that all information including the SAS Address are correct.

## 6. Complete!
- You are now able to reboot the system and have completed this guide.

&emsp;&emsp; - Command in UEFI shell: ```reset```

## Please leave any feedback or questions!

# Resources
- MAIN TUTORIAL - [How to flash a LSI SAS 3008 HBA (e.g. IBM M1215) to IT mode - ServeTheHome Forum](https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/)
- [Crossflashing of LSI 9341-8i to LSI 9300-8i - ServeTheHome Forum](https://forums.servethehome.com/index.php?threads/crossflashing-of-lsi-9341-8i-to-lsi-9300-8i-success-but-no-smart-pass-through.3522/)
- [Detailed newcomers' guide to crossflashing LSI 9211/9300/9305/9311/9400/94xx HBA and variants - TrueNAS Forum](https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/)
- [[HOW-TO] Flashing LSI SAS HBA Controller [EFI/UEFI] - TrueNAS Forum](https://www.truenas.com/community/threads/how-to-flashing-lsi-sas-hba-controller-efi-uefi.78457/)
- Older Card Versions - [LSI SAS 2008 RAID Controller/HBA Information - ServeTheHome Forum](https://www.servethehome.com/lsi-sas-2008-raid-controller-hba-information/)
