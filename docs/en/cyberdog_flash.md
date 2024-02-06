# Cyberdog Developer Flash Operation Instructions Document

## 1. Flashing process

1. First upgrade the sensing motherboard Xavier NX
2. Then upgrade other sub-boards inside the robot dog
3. During the flashing process, no matter whether it is successful or failed, there will be a corresponding prompt sound.

## 2. Flash configuration file description

During the flashing process, the upgrade-aware motherboard Xavier NX is required, and other daughter boards can decide whether to upgrade based on this configuration file.

### 2.1 Specific file format examples

```tex
r329 = on
mr813 = on
power=on
imu=on
head-tof=on
tail-tof=on
mini-led=on
head-uwb=on
tail-uwb=on
check-battery=40
force-update=off
```

Parameter explanation:

- **The first 9 lines, each line includes three parts, take the first line as an example: **
   - r329 represents the name of the motherboard to be upgraded

   - Symbol "="

   - Whether to upgrade other motherboards, upgrade is "on", not upgrade is "off" (it is recommended to update all motherboards simultaneously to ensure system stability)


- **"check-battery" indicates whether to check the power or detect the critical value of the power before upgrading the NX board: **
   - The value range is 0-100
   - 0 means no power detection
   - Greater than 0, such as 30, means that if the current power of the machine is less than 30%, the whole machine will stop flashing. At this time, the machine should be charged until it is greater than 30% and the whole machine will be flashed again.

- **"force-update" indicates whether to force the upgrade of other motherboards after upgrading the NX board, that is, whether to detect the communication status of other motherboards and the NX board. Detecting the communication status in advance can discover other motherboards that cannot be upgraded before upgrading the NX board: **

   - "on" means forced upgrade, that is, the communication status is not detected

   - "off" means not to force the upgrade, that is, to detect the communication status

### 2.2 Configuration file format requirements

- **The file format must be text format under linux (line break LF)**

   ![](./image/cyberdog_flash/conf_file_format.png)

- **The number of lines in the file must be 11, and there must be no line breaks after the 11th line**

- **There must be at least one space separating the three parts of each line**

- **The first and second parts of each line must be the same as the sample file in 2.1**

- **The third part of each line must be one of on or off, or 0-100 of "check-battery"**

### 2.3 Default value without specifying a configuration file

- Each motherboard defaults to "off", which means it will not be upgraded.

- "check-battery" defaults to 0, which means the battery is not checked

- "force-update" defaults to "on", which means forced upgrade. The communication between each motherboard and the NX board is no longer checked before the upgrade.

## 3. Introduction to flashing method

Currently there are two ways to flash the robot dog:

- PC wire brushing method

​ Using this method to flash the computer requires a PC with Ubuntu system and a special download cable. The time depends on the PC configuration, and it basically takes about 15 minutes.

​ This method has no requirement for the name of the flash configuration file.

- U disk card swiping method

​Using this method to flash the phone requires a USB flash drive. The time depends on the speed of the USB flash drive. It basically takes about 10 minutes, and does not require the participation of the flashing line, which will be more convenient.

​ This method requires that the name of the flash configuration file must be fixed to ota_others.conf, and everything else is the same as the PC wire flash method.

​ ** Regarding the USB flash drive card swiping method, you need to pay attention to the following: **

​ 1. In this method, the USB flash drive does not support connection through the USB Hub and must be directly inserted into the robot dog!

2. Currently, solid state drives with USB interface are not supported as flash USB flash drives.

3. Each time you flash the phone, you need to re-enable the flash flag. For details, please refer to "5.7 Need to flash again"

​ 4. U disk only supports type-c interface, and due to the shape design of the robot dog, the space provided for U disk insertion is not very large, so a U disk of suitable size is required. If you need to purchase, you can refer to the following link:

​ https://item.jd.com/5522803.html?bbtf=1#crumb-wrap

The above two flashing methods require a PC with Ubuntu system installed, and currently do not support Windows system.

## 4. PC wire brushing method

### 4.1 Unzip the flash package

Execute the following command to unzip the official release flash package to the PC directory (the path and name of the directory are not required)

```bash
$ sudo tar -xvf flash package path -C directory to extract to
```

### 4.2 Configure the flashing environment

After decompression is completed, go to the decompressed directory and execute the following command to configure the flashing environment

```bash
$ tools/otf_tools/setup.sh
```

> Note:
>
> 1. If prompted with permission issues, please execute:
>
> $ chmod +x tools/otf_tools/setup.sh
>
> 2. During the execution of the script, you may need to enter "Y" to confirm the installation. If you encounter it, you can enter it directly.

### 4.3 Enter flash mode

With the robot dog's head facing forward, insert the included flash cable into the USB TypeC interface on the far right, and then follow the instructions below:

- If it is currently powered off, press and hold the power button for 5 seconds. This will indicate that the robot dog has entered flash mode.
- If it is currently on, press and hold the power button for 5 seconds, release the power button, and then press and hold the power button for 5 seconds. This means that the robot dog has entered the flash mode.

### 4.4 Flash the machine

After the robot dog enters the flash mode, continue to execute the following command to flash the machine (where **flash_conf_file_path** is the path of the flash configuration file mentioned above in the PC, and the specific path and name are not required)

```Shell
$ sudo ./flashall.sh --others-ota-conf-path flash_conf_file_path
```

### 4.5 Wait for flashing to complete

The time depends on the specific PC configuration, about 15 minutes, and will be carried out in the following order:

1. After executing the command in the previous step, if "Flash Successfully!" is finally displayed, it means that the motherboard Xavier NX has been successfully upgraded. At this time, you can unplug the flash cable;
2. The voice prompt "Start updating other motherboards" means starting to update various sub-boards except the sensing motherboard Xavier NX. During the update process, "Start updating other motherboards" will be broadcast in a loop;
3. The voice prompt "Other motherboards have been updated successfully" means that the other sub-boards have been updated successfully, and all flashing is completed at this time;
4. Restart the robot dog. Specifically, press and hold the power button for 5 seconds, release the power button, and then press and hold the power button for 5 seconds. At this time, the robot dog restarts successfully and enters normal use state.

> Note: It is currently known that when installing the ros environment, if the following content about the character set is configured:
>
> ````bash
> sudo apt update && sudo apt install locales
> sudo locale-gen en_US en_US.UTF-8
> sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
> export LANG=en_US.UTF-8
> ```
>
> This will cause the subsequent steps to not continue after upgrading the sensing motherboard Xavier NX in the first step above.
>
> At this time, you need to manually restart the robot dog to continue the second and subsequent steps above.

## 5. U disk card swiping method

**The following operations will cause all data on the USB flash drive to be lost! ! Please make a backup in advance! ! **

### 5.1 Prepare script tools

1. If you have already used the PC wire flash method to flash the phone, because the script tool is placed in the official release flash package, you can skip this step, otherwise you need to perform the following operations.

2. Execute the following command to unzip the flash package to the PC directory (the path and name of the directory are not required)

    ```bash
    $ sudo tar -xvf flash package path -C directory to extract to
    ```


### 5.2 Make a USB flash drive that meets the requirements

#### 5.2.1 Determine the USB device name

Because the operation behind this name is needed, it needs to be determined first.

The method can be referenced as follows:

1. Without inserting the USB flash drive into the PC, execute the following command:

    ```bash
    $ ls /dev/sd*
    /dev/sda /dev/sda1 /dev/sda2
    ```

2. Insert the USB flash drive into the PC and execute the above command again:

   ```bash
   $ ls /dev/sd*
   /dev/sda  /dev/sda1  /dev/sda2  /dev/sdb  /dev/sdb1
   ```

​ As you can see, there are "/dev/sdb" and "/dev/sdb1", among which:

​/dev/sdb: Represents the U disk device, here called **device name**

​/dev/sdb1: Represents the first partition of the device, here called **device partition name**

**It should be noted that the above /dev/sdb and /dev/sdb1 will change with different PCs and are not fixed. They need to be determined according to personal circumstances. **

#### 5.2.2 Make

Currently two production methods are supported:

1. The first production method integrates all steps into a script tool, which is convenient and fast. It is recommended to use
2. The second production method breaks down all the steps and requires manual operation step by step. It is suitable for backup in case of problems with the first method.

##### 5.2.2.1 Production method 1

1. Insert the USB flash drive into the PC

2. Execute the following command to enter the directory where the script tool is stored in the directory extracted in step "5.1 Preparing the script tool".

    ```bash
    cd unzip directory/tools/otf_tools
    ```

3. Execute the following command to start production ("U disk device name" is the device name determined in the previous example)

    ```bash
    $ sudo ./mkudisk.sh U disk device name
    ```
   
    If the following display appears, the production is successful.
   
    ```bash
    makeudisk successfully!
    ```

##### 5.2.2.2 Production method 2

1. Unmount the default mounted partition

    In the step "5.2.1 Determine the name of the U disk device", since the U disk is inserted at the end, it will be automatically mounted to a certain directory by default. However, subsequent operations require uninstallation, so execute the following command to uninstall (the parameter "/ "dev/sdb1" is the device partition name determined in step "5.2.1 Determine the U disk device name")

    ```bash
    $ umount /dev/sdb1
    ```

​ **Keep the U disk inserted, do not pull it out, and continue with the following operations! ! **

2. Modify the partition table to gpt format

    Since the flashing function requires the U disk to have a partition table in gpt format, but generally the partition table format of new U disks is msdos, so it needs to be modified. Specifically, you can use the parted command to complete this requirement. The specific operations are as follows (the parameter "/dev/sdb" is the device name determined in step "5.2.1 Determine the U disk device name")

    ```bash
    $ sudo parted /dev/sdb
    GNU Parted 3.3
    Use /dev/sdb
    Welcome to GNU Parted! Type 'help' to see a list of commands.
    (parted) mklabel
    New disk volume label type? gpt
    Warning: The existing disk volume on /dev/sdd will be destroyed and all data on this disk will be lost. Do you want to continue?
    Yes/Yes/No/No? y
    (parted) quit
    INFO: You may need /etc/fstab.
    ```
   
3. Repartition

    After the partition table format is changed to gpt, all previous data will be lost, so repartitioning is required. Specifically, you can also use the parted command, and the specific operations are as follows (the parameter "/dev/sdb" is the device name determined in step "5.2.1 Determine the U disk device name")

    ```bash
    $ sudo parted /dev/sdb
    GNU Parted 3.3
    Use /dev/sdb
    Welcome to GNU Parted! Type 'help' to see a list of commands.
    (parted) mkpart
    Partition name? []?otf
    File system type? [ext2]? ext4
    Starting point? 0%
    Ending point? 100%
    (parted) quit
    INFO: You may need /etc/fstab.
    ```

4. Format and fix the mounting directory name of the USB flash drive

     a. Search for the "disk" software under the Ubuntu system, open and find the corresponding interface of the U disk, in this example it is the fourth one on the left (find the corresponding disk of the U disk according to personal actual situation)
    
     <img src="./image/cyberdog_flash/format_udisk1.png" style="zoom:80%;" />

​ b. Click the Settings button and select format partition, as shown in the figure below

​ <img src="./image/cyberdog_flash/format_udisk2.png" style="zoom:80%;" />

​ c. Enter the formatting interface and make the following selections:

Volume name: Fixed to "**otf_usb**" (note that this name must be used here, otherwise subsequent operations will not be successful)

​ Erase: Just don’t select the default

Type: Select the first item "Internal disk (Ext4) (I) for Linux systems only"

​ Finally, as shown in the figure below

​ ![](./image/cyberdog_flash/format_udisk3.png)

​ d. Finally click Next in the upper right corner, then click Format in the upper right corner and wait for completion

​ <img src="./image/cyberdog_flash/format_udisk4.png" style="zoom:80%;" />

### 5.3 Copy image to U disk

1. Make sure that the created USB flash drive has been mounted on the PC

1. Execute the following command to enter the directory where the script tool is stored in the directory extracted in step "5.1 Preparing the script tool"

    ```bash
    cd unzip directory/tools/otf_tools
    ```

1. Open a terminal in the PC and execute the following command (the script parameter is the full path name of the official release flash package in the PC, and the suffix of the flash package name also needs to be written in full)

    ```bash
    $ ./mkimage.sh The full name of the official release flash package path
    ```

This command may take about 10 minutes, please be patient.

> Note:
>
> 1. You need to enter the password after executing the command. If the terminal remembers it, it is no longer required.
>
> 2. If it prompts bash: ./mkimage.sh: Insufficient permissions, just execute the following command:
>
> $ chmod +x mkimage.sh

4. The following content is finally displayed to indicate success:

​ Make udisk image successfully!

The final display of the following content indicates failure:

​ Fail to make udisk image!

> Note:
>
> ​ If it stays in Wait for the udisk to be ready... for a long time, you need to check whether the USB flash drive has been inserted into the PC. If it has been inserted into the PC, you can try to plug it in again.

### 5.4 Place the flash configuration file into the root directory of the U disk

In this flashing method, the configuration file requirements are exactly the same as the PC card flashing method except that the name must be fixed to ota_others.conf.

### 5.5 Insert the USB disk into the robot dog and restart

1. Insert the USB flash drive into the leftmost USB TypeC port of the robot dog, as shown in the figure below:

<img src="./image/cyberdog_flash/flash_usb_port.png" style="zoom: 67%;" />

2. Restart the robot dog as follows:

- If it is currently powered off, press and hold the power button for 5 seconds. This will indicate that the robot dog has started and will enter the flash mode.

- If it is currently on, press and hold the power button for 5 seconds, release the power button, and then press and hold the power button for 5 seconds. At this time, it means that the robot dog has started and entered the flash mode.

### 5.6 Wait for flashing to complete

#### 5.6.1 Prompt tone

During the upgrade process, the following prompts will be played in sequence:

1. The voice prompt "Start application board flashing" indicates that the upgrade of the sensing motherboard Xavier NX has begun;

1. The voice prompt “App flashing successful” indicates that the Xavier NX motherboard has been upgraded successfully;

1. The voice prompt "Start updating other motherboards" means starting to update various sub-boards except the sensing motherboard Xavier NX. It should be noted that during the update process, "Start updating other motherboards" will be broadcast cyclically;

1. The voice prompt "Other motherboards have been updated successfully" means that the other sub-boards have been updated successfully;

1. At this point, the flashing process is complete and you can unplug the USB flash drive;

1. Restart the robot dog. Specifically, press and hold the power button for 5 seconds, release the power button, and then press and hold the power button for 5 seconds. At this time, the robot dog restarts successfully and enters normal use state.

#### 5.6.2 Time required

Depending on the speed of the USB flash drive, it may take about 10 minutes.

### 5.7 Need to flash again

#### 5.7.1 Prepare the USB disk made above

#### 5.7.2 There are two situations below. Please confirm which one you belong to before proceeding.

##### 5.7.2.1 No updated version required

1. Enter the directory where the prepared USB disk is mounted on the PC, and execute the following command to reopen the flashing logo (the following display means the modification is successful)

```Shell
$ ./enable_otf.sh
enable otf function successfully!
```

2. Perform step "5.4 Place the flash configuration file into the root directory of the USB flash drive"

3. Perform step "5.5 Insert the USB flash drive into the robot dog and restart"

4. Follow the instructions in step "5.6 Wait for the flashing to complete" and wait for the flashing to complete.

##### 5.7.2.2 requires an updated version

1. Perform step "5.3 Copy image to USB disk"

1. Perform step "5.4 Place the flash configuration file into the root directory of the USB flash drive"

1. Perform step "5.5 Insert the USB flash drive into the robot dog and restart"

1. Follow the instructions in step "5.6 Wait for the flashing to complete" and wait for the flashing to complete.

## 6. Debugging method

**By default, all debugging interfaces are blocked. You can apply for developer permissions to enable the debugging interface. **

After the debugging interface permission is turned on, remove the debugging cover. There are two ways to connect to the robot dog in the PC:

1. Connect the PC to the robot dog directly through a network cable, and then enter the following command on the PC to connect to the robot dog (password: 123):

    ```shell
    $ ssh mi@192.168.44.1
    ```

2. Connect to the robot dog through a USB type-c cable (the interface is located on the right side of the middle charging port), and then enter the following command on the PC to connect to the robot dog (password: 123):

    ```shell
    $ ssh mi@192.168.55.1
    ```


