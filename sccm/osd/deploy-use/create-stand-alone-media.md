---
title: "Create stand-alone media"
titleSuffix: "Configuration Manager"
description: "Use stand-alone media to deploy the operating system on a computer without a network connection."
ms.custom: na
ms.date: 02/09/2018
ms.prod: configuration-manager
ms.reviewer: na
ms.suite: na
ms.technology:
  - configmgr-osd
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: c6b9ccd2-78d9-4f0e-b25a-70d0866300ba
caps.latest.revision: 21
caps.handback.revision: 0
author: aczechowski
ms.author: aaroncz
manager: dougeby

---
# Create stand-alone media with System Center Configuration Manager

*Applies to: System Center Configuration Manager (Current Branch)*

Stand-alone media in Configuration Manager contains everything required to deploy the operating system (OS) on a computer without a network connection. Use stand-alone media with the following OS deployment scenarios:  

-   [Refresh an existing computer with a new version of Windows](refresh-an-existing-computer-with-a-new-version-of-windows.md)  

-   [Install a new version of Windows on a new computer (bare metal)](install-new-windows-version-new-computer-bare-metal.md)  

-   [Upgrade Windows to the latest version](upgrade-windows-to-the-latest-version.md)  

Stand-alone media includes the task sequence that automates the steps to install the OS, and all other required content. This content includes the boot image, operating system image, and device drivers. Because the stand-alone media stores everything to deploy the OS, it requires more disk space than required for other types of media. When you create stand-alone media on a central administration site, the client retrieves its assigned site code from Active Directory. Stand-alone media created at child sites automatically assigns to the client the site code for that site.  

##  <a name="BKMK_CreateStandAloneMedia"></a> Create stand-alone media  
Before you create stand-alone media by using the Create Task Sequence Media Wizard, be sure that the following conditions are met:  

### Create a task sequence to deploy an operating system
As part of the stand-alone media, you must specify the task sequence to deploy an operating system. For the steps to create a new task sequence, see [Create a task sequence to install an operating system in System Center Configuration Manager](create-a-task-sequence-to-install-an-operating-system.md).

The following actions are not supported for stand-alone media:
- The **Auto Apply Drivers** step in the task sequence. Stand-alone media does not support automatic application of device drivers from the driver catalog. Use the **Apply Driver Package** step to make a specified set of drivers available to Windows Setup.
- The **Download Package Content** step in the task sequence. The management point information is not available on standalone media, therefore the step fails trying to enumerate content locations.
- Installing software updates.
- Installing software before deploying the operating system.
- Task sequences for non-operating system deployments.
- Associating users with the destination computer to support user device affinity.
- Dynamic package installs via the **Install Packages** task.
- Dynamic application installs via the **Install Application** task.

> [!NOTE]    
> An error might occur if your task sequence includes the [Install Package](../../osd/understand/task-sequence-steps.md#BKMK_InstallPackage) step and you create the stand-alone media at a central administration site. The central administration site does not have the necessary client configuration policies. These policies are required to enable the software distribution agent during the execution of the task sequence. The following error might appear in the CreateTsMedia.log file:    
>     
> `WMI method SMS_TaskSequencePackage.GetClientConfigPolicies failed (0x80041001)`    
> 
> For stand-alone media that includes an **Install Package** step, create the stand-alone media at a primary site that has the software distribution agent enabled. 
>
> Alternatively, add a [Run Command Line](../understand/task-sequence-steps.md#BKMK_RunCommandLine) step after the [Setup Windows and ConfigMgr](../understand/task-sequence-steps.md#BKMK_SetupWindowsandConfigMgr) step and before the first **Install Package** step in the task sequence. The **Run Command Line** step runs the following WMIC command to enable the software distribution agent before the first Install Package step:    
>    
> `WMIC /namespace:\\\root\ccm\policy\machine\requestedconfig path ccm_SoftwareDistributionClientConfig CREATE ComponentName="Enable SWDist", Enabled="true", LockSettings="TRUE", PolicySource="local", PolicyVersion="1.0", SiteSettingsKey="1" /NOINTERACTIVE`


> [!IMPORTANT]    
> Do not select the **Use pre-production client package when available** setting in the **Setup Windows and ConfigMgr** task sequence step for stand-alone media. Stand-alone media does not support use of this setting. For more information about this setting, see [Setup Windows and ConfigMgr](/sccm/osd/understand/task-sequence-steps#BKMK_SetupWindowsandConfigMgr).


### Distribute all content associated with the task sequence
Distribute all content that is required by the task sequence to at least one distribution point. This  content includes the boot image, operating system image, and other associated files. The wizard gathers the information from the distribution point when it creates the stand-alone media. You must have **Read** access rights to the content library on that distribution point. For more information, see [Distribute content referenced by a task sequence](manage-task-sequences-to-automate-tasks.md#BKMK_DistributeTS).

### Prepare the removable USB drive
*For a removable USB drive:*

If you are using a removable USB drive, connect the USB drive to the computer where the wizard is run. The USB drive must be detectable by Windows as a removable device. The wizard writes directly to the USB drive when it creates the media. Stand-alone media uses a FAT32 file system. You cannot create stand-alone media on a USB flash drive whose content contains a file over 4 GB in size.

### Create an output folder
*For a CD/DVD set:*

Before you run the Create Task Sequence Media Wizard to create media for a CD or DVD set, you must create a folder for the output files created by the wizard. Media that is created for a CD or DVD set is written as .iso files directly to the folder.


 Use the following procedure to create stand-alone media for a removable USB drive or a CD/DVD set.  

## To create stand-alone media  

1.  In the Configuration Manager console, click **Software Library**.  

2.  In the **Software Library** workspace, expand **Operating Systems**, and then click **Task Sequences**.  

3.  On the **Home** tab, in the **Create** group, click **Create Task Sequence Media** to start the Create Task Sequence Media Wizard.  

4.  On the **Select Media Type** page, specify the following options, and then click **Next**.  

    -   Select **Stand-alone media**.  

    -   Optionally, if you want to allow the operating system to be deployed without requiring user input, select **Allow unattended operating system deployment**. When you select this option, the user is not prompted for network configuration information or for optional task sequences. If the media is configured for password protection, the user is still prompted for a password.  

5.  On the **Media Type** page, specify whether the media is a removable USB drive or a CD/DVD set:  

    > [!IMPORTANT]  
    >  By default, stand-alone media uses a FAT32 file system. You cannot create stand-alone media on a removable USB drive whose content contains a file over 4 GB in size.  

    -   If you select **Removable USB drive**, specify the drive where you want to store the content.  

        - **Format removable USB drive (FAT32) and make bootable**: By default, let Configuration Manager prepare the USB drive. Many newer UEFI devices require a bootable FAT32 partition. However, this format also limits the size of files and overall capacity of the drive. If you have already formatted and configured the removable drive, disable this option. 

    -   If you select **CD/DVD set**, specify the capacity of the media and the name and path of the output files. The wizard writes the output files to this location. For example: **\\\servername\folder\outputfile.iso**  

         If the capacity of the media is too small to store the entire content, multiple files are created and you must store the content on multiple CDs or DVDs. When multiple media is required, Configuration Manager adds a sequence number to the name of each output file that it creates. In addition, if you deploy an application along with the operating system and the application cannot fit on a single media, Configuration Manager stores the application across multiple media. When the stand-alone media is run, Configuration Manager prompts the user for the next media where the application is stored.   

         > [!IMPORTANT]  
         >  If you select an existing .iso image, the Task Sequence Media Wizard deletes that image from the drive or share as soon as you proceed to the next page of the wizard. The existing image is deleted, even if you then cancel the wizard.  

     Click **Next**.  

6.  On the **Security** page, choose from the following settings, and then click **Next**:
    - **Protect media with a password**: Enter a strong password to help protect the media. If you specify a password, the password is required to use the media.  

        > [!IMPORTANT]  
        >  On stand-alone media, only the task sequence steps and their variables are encrypted. The remaining content of the media is not encrypted, so do not include any sensitive information in task sequence scripts. Store and implement all sensitive information by using task sequence variables.  

    - **Select date range for this stand-alone media to be valid** (starting in version 1702): Set optional start and expiration dates on the media. These settings are disabled by default. The dates are compared to the system time on the computer before the stand-alone media runs. When the system time is earlier than the start time or later than the expiration time, the stand-alone media is not started. These options are also available by using the New-CMStandaloneMedia PowerShell cmdlet.
7.  On the **Stand-Alone CD/DVD** page, specify the task sequence that deploys the operating system, and then click **Next**. To add content to the stand-alone media for application dependencies, choose **Detect associated application dependencies and add them to this media**.
    > [!TIP]
    > If you do not see expected application dependencies, deselect and then reselect the **Detect associated application dependencies and add them to this media** setting to refresh the list.

    The wizard lets you select only those task sequences that are associated with a boot image.  

8. On the **Select Application** page (available beginning in version 1702), specify application content to include as part of the media file, and then click **Next**.
9. On the **Select Package** page (available beginning in version 1702), specify the package content to include as part of the media file, and then click **Next**.
10. On the **Select Driver Package** page (available beginning in version 1702), specify the driver package content to include as part of the media file, and then click **Next**.
11.  On the **Distribution Points** page, specify the distribution points that contain the required content, and then click **Next**.  

     Configuration Manager only displays distribution points that have the content. Distribute all of the content associated with the task sequence to at least one distribution point before you continue. After you distribute the content, refresh the distribution point list. Remove any distribution points that you already selected on this page, go to the previous page, and then back to the **Distribution Points** page. Alternatively, restart the wizard. For more information, see [Distribute content referenced by a task sequence](manage-task-sequences-to-automate-tasks.md#BKMK_DistributeTS) and [Manage content and content infrastructure](../../core/servers/deploy/configure/manage-content-and-content-infrastructure.md).  

    > [!NOTE]  
    >  You must have **Read** access rights to the content library on the distribution points.  

12. On the **Customization** page, specify the following information, and then click **Next**.  

    -   Specify the variables that the task sequence uses to deploy the operating system.  

    -   Specify any prestart commands that you want to run before the task sequence. Prestart commands are a script or an executable that runs in Windows PE before the task sequence starts. For more information, see [Prestart commands for task sequence media](../understand/prestart-commands-for-task-sequence-media.md).  

         Optionally, select **Files for the prestart command** to include any required files for the prestart command.  

        > [!TIP]  
        >  During task sequence media creation, Configuration Manager writes the package ID and prestart command line to the **CreateTSMedia.log** file on the computer that runs the Configuration Manager console. This output includes the value for any task sequence variables. Review this log file to verify the value for the task sequence variables.  

13. Complete the wizard.  

 The stand-alone media files (.iso) are created in the destination folder. If you selected **Stand-Alone CD/DVD**, you can now copy the output files to a set of CDs or DVDs.  

##  <a name="BKMK_StandAloneMediaTSExample"></a> Example task sequence for stand-alone media  
 To create a task sequence to deploy an operating system using stand-alone media, use the following table as a guide. The table helps you decide the general sequence for your task sequence steps. It also helps organize and structure the task sequence steps into logical groups. The task sequence that you create might vary from this sample and can contain more or fewer task sequence steps and groups.  

> [!NOTE]  
>  You must always use the Task Sequence Media Wizard to create stand-alone media.  

|Task Sequence Group or Step|Description|  
|---------------------------------|-----------------|  
|Capture File and Settings - **(New Task Sequence Group)**|Create a task sequence group. A task sequence group keeps similar task sequence steps together for better organization and error control.|  
|Capture Windows Settings|Use this task sequence step to capture the Windows settings from the destination computer prior to reimaging. Capture the computer name, user and organizational information, and the time zone settings.|  
|Capture Network Settings|Use this task sequence step to capture network settings from the computer that receives the task sequence. You can capture the domain or workgroup membership of the computer and the network adapter setting information.|  
|Capture User Files and Settings - **(New Task Sequence Subgroup)**|Create a task sequence group within a task sequence group. This subgroup contains the steps to capture user state data from the destination computer prior to reimaging. Similar to the initial group that you added, this subgroup keeps similar task sequence steps together for better organization and error control.|  
|Set Local State Location|Use this task sequence step to specify a local location using the protected path task sequence variable. The user state is stored on a protected directory on the hard drive.|  
|Capture User State|Use this task sequence step to capture the user files and settings you want to migrate to the new operating system.|  
|Install Operating System - **(New Task Sequence Group)**|Create another task sequence subgroup. This subgroup contains the steps needed to install the operating system.|  
|Reboot to Windows PE or hard disk|Use this task sequence step to specify restart options for the computer that receives this task sequence. This step displays a message to the user that the computer is restarting to continue the installation.<br /><br /> This step uses the read-only **_SMSTSInWinPE** task sequence variable. If the associated value equals **false**, the task sequence step continues.|  
|Apply Operating System|Use this task sequence step to install the operating system image onto the destination computer. This step deletes all files on that volume, except for Configuration Manager-specific control files. It then applies all volume images contained in the WIM file to the corresponding sequential disk volume. You can also specify a **sysprep** answer file to configure which disk partition to use for the installation.|  
|Apply Windows Settings|Use this task sequence step to configure the Windows settings configuration information for the destination computer. These settings include user and organizational information, product or license key information, time zone, and the local administrator password.|  
|Apply Network Settings|Use this task sequence step to specify the network or workgroup configuration information for the destination computer. You can also specify if the computer uses a DHCP server or you can statically assign the IP address information.|  
|Apply Driver Package|Use this task sequence step to make all device drivers in a driver package available for use by Windows setup. All necessary device drivers must be contained on the stand-alone media.|  
|Setup Operating System - **(New Task Sequence Group)**|Create another task sequence subgroup. This subgroup contains the steps needed to install the Configuration Manager client.|  
|Setup Windows and ConfigMgr|Use this task sequence step to install the Configuration Manager client software. Configuration Manager installs and registers the Configuration Manager client GUID. You can assign the necessary installation parameters in the **Installation properties** window.|  
|Restore User Files and Settings - **(New Task Sequence Group)**|Create another task sequence subgroup. This subgroup contains the steps needed to restore the user state.|  
|Restore User State|Use this task sequence step to initiate the User State Migration Tool (USMT). USMT restores the user state and settings, previously captured with the Capture User State step, to the destination computer.|  
