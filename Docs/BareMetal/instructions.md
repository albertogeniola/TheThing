# Guest image preparation

# Prepare the server

## Install ISCSI target
Our setup is based on an iSCSI service on the top of Windows Server 2012 R2.
The very first step of the tutorial consists in installing the iSCSI File Target service on the server.
This can be done via the ServerManager, by selecting "Install server role" and specifying the following role:

File & ISCSI Services / ISCSI Target Server

## Prepare the base image disk



# Installing the Windows OS on the iSCSI target
1. Download the ipxe iso and flash it on a USB drive
2. Insert the USB drive into the Guest OS. Also, insert the DVD of Windows 7 32 bit on the same machine.
3. Configure the BIOS so that the first boot device is the USB pen, followed by the CDROM
4. Start the system. Once iPXE appears, press CTRL+B, and type the following commands

dhcp net0
set keep-san 1
set skip-san-boot 1
sanhook --drive 0x80 iscsi:<ip_iscsi_target>::::<iqn_iscsi_target>:<target_name>
exit

At this point, the windows installation procedure should begin. Thus, proceed with normal installation.
Note: some BIOSes will not fall back to next boot device when exiting the iPXE environment. In such case, the 
installation of the operating system must be done via WinPE. 

Windows installer will reboot a couple of times before installation is completed. When this happens, the system will
fail to continue the installation and the windows installation procedure will start over again. To fix this problem,
the user has to manually boot from iscsi after every reboot, until the installation is completed. Thus, when the system
reboot, the user has to boot into the iPXE enviroment and type the following two commands:

dhcp net0
sanboot --drive 0x80 iscsi:<ip_iscsi_target>::::<iqn_iscsi_target>:<target_name>

Hopefully, the system will boot again and the installation will continue like normal. Repeat this process until the 
installation is done and the system, is ready to be customized.