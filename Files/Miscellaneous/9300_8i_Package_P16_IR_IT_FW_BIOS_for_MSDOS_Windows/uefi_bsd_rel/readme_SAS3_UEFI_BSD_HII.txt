***********************************************************************

Avago Technologies SAS3 MPT IT/IR UEFI BSD & HII Configuration Utility

***********************************************************************

Supported Controllers:

SAS3004
SAS3008
SAS3108_1
SAS3108_2
SAS3108_3
SAS3108_4
SAS3108_5
SAS3108_6
SAS3216
SAS3224
SAS3316_1
SAS3316_2
SAS3316_3
SAS3316_4
SAS3324_1
SAS3324_2
SAS3324_3
SAS3324_4

Component:
=========
Binary image name: mpt3x64.rom
 
Installation:
=============
Use sas3flash.efi to install the SAS3 UEFI BSD & HII Configuration Utility.
The sas3flash utility is included in the package zip file.
UEFI version of sas3flash.efi can be downloaded from the Support & Downloads section of www.lsi.com.

The command line installation instruction to flash the SAS3 UEFI BSD & HII Configuration Utility is:

sas3flash -c <n> -b mpt3x64.rom

where <n> is the controller number (starting with zero (0)).

If you need further help, please contact the Avago FAE associated with your Organization.

Notes: 
1) The SAS3 UEFI BSD & HII Configuration Utility does not require Legacy BIOS to be loaded on to the controller.
2) The firmware in the controller should be fully operational for flashing the SAS3 UEFI BSD & HII Configuration Utility.
3) The release supports X64 platforms.


