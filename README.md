# ESP32 Secure Key Storage

## Goals of this Tutorial

In this tutorial, the student will learn how to use the secure key storage capabilities of IoT devices. The ESP32 will be used in this lab. The student will first learn how to communicate and program the ESP32. Then they will learn how to perform a physical readout of the flash and steal WiFi credentials in the firmware. Finally, the student will learn how to enable flash encryption and protect the firmware against attacks.

### Author's Note

The attack and defense presented in this lab are nearly identical to our [flash encryption tutorial](https://github.com/PBearson/ESP32_Flash_Encryption_Tutorial). However, this lab places a greater emphasis on the ESP32's secure key storage capabilities. We also enable flash encryption in Release Mode instead of Development Mode.

### Overview

* Student will learn about the secure key storage capabilities of the ESP32.
* Student will learn how to perform a physical readout of the ESP32 flash and identify sensitive data in the firmware.
* Student will learn how to enable the flash encryption of the ESP32 by leveraging the secure key storage.

### Hardware Needed
* USB A to Micro USB cable.
* A Hiletgo ESP-WROOM-32 development board.

### Software Needed
* Ubuntu 20.04 disc image (ISO)
* VirtualBox
* ESP-IDF version 4.1 or higher: ESP-IDF is a framework for ESP32 that comes with all necessary software, including the compiler toolchain and various tools for programming and inspecting the ESP32's secure key storage.

A preconfigured VM with all the necessary software can be downloaded [here](http://cyberforensic.net/labs/iot_secure_key_storage/files/).

## Environment Setup

### Setting up the Virtual Machine

Download the VM image from the lab materials link. Once it finishes downloading, you need to import it into VirtualBox. In VirtualBox, click on <ins>File → Import Appliance</ins>. Select the downloaded image. Click <ins>Next</ins>, then <ins>Import</ins>. After a minute or so, the VM will appear on the left side of the window, as shown below:

![vm_setup](https://user-images.githubusercontent.com/11084018/160431335-67865a6e-2494-490c-9bbb-247b14f10d3a.png)

You can also click <ins>Settings</ins> to modify the VM before booting it up. By default, it will use 8 GB of RAM and 2 CPU cores, so change these settings to suit your environment. Once you are done, click <ins>Start</ins> to boot up the machine.

### Verifying the Setup

First, log into the machine. The username and password are both **esplab**. Then click on the terminal icon on the left side of the screen. If all goes well, you will be greeted with the following message, indicating that the development environment is successfully installed:

![vm_verify](https://user-images.githubusercontent.com/11084018/160431442-33fa2cc7-35ff-4f91-8216-c099e1f2f584.png)

### Setting up the Serial Port

Now we will verify that we can program and monitor our ESP32. First, we need to install the necessary drivers. Most ESP32 boards will contain a CP210X chip which allows data transfer between UART (used by the ESP32) and USB (used by the cable). However, communicating with this chip, and therefore the ESP32 itself, requires the correct drivers to be installed on your computer. You can download the CP210X drivers to your host machine from [here](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers).

Now, plug the ESP32 into your computer. You can confirm that the driver was installed correctly by navigating to <ins>Devices → USB</ins> in your VM window. You should see a device such as "Silicon Labs CP2102 USB to UART Controller". Click this button in order to enable communication between your VM and the ESP32.

![usb_device_highlighted](https://user-images.githubusercontent.com/11084018/160431513-580652e5-1bf5-4e42-8bec-a4af1f95f1a2.png)

### Running the Example Application

Now we will run the example "hello_world" application for the ESP32, located in the following directory: _/home/esplab/esp/workspace/hello_world_. In your terminal, navigate to this directory. Now run:

```
idf.py build
```

This will compile the application. The compilation will take a moment to complete. If all goes well, you will be greeted with a "Project build complete" message and a command to upload the firmware to the ESP32, as shown below:

![project_build_complete](https://user-images.githubusercontent.com/11084018/160431536-f400cc48-5f35-46b8-beca-4d2794655456.png)

To upload the compiled firmware to your board, run:

```
idf.py flash
```

A common error you may see at this point is "RuntimeError: No serial ports founds", which occurs when the build system cannot identify the ESP32 serial port. You can find this serial port yourself by running:

```
ls /dev/ttyUSB*
```

This should return a file such as "/dev/ttyUSB0". If you see this file, then rerun the command as:

```
idf.py -p /dev/ttyUSB0 flash
```

If you do not see this file, that means your VM cannot see the ESP32. Double-check to ensure that the USB-to-UART driver was successfully installed. Another possibility is that your ESP32 board contains a different serial-to-USB chip than the CP210X. In that case, you should consult the documentation for that board to identify the correct chip and download the appropriate driver.

**NOTE: If using the Hiletgo ESP-WROOM-32 development board, you may need to hold down the IO0 button on the ESP32 when the build system tries to connect to the ESP32's serial port. If you do not hold down the IO0 button during this step, the build system may fail to detect the serial port.**

If the flash command works, you will see that the firmware will upload to the board, as shown below:

![running_flash_command](https://user-images.githubusercontent.com/11084018/160431565-1a5d21c0-f227-48d9-b4e5-1198493d3c2e.png)

To monitor output from the application, run:

```
idf.py monitor
```

Or:

```
idf.py -p /dev/ttyUSB0 monitor
```

This application prints "Hello world", various information about your particular board, and resets after 10 seconds.

![output_monitor](https://user-images.githubusercontent.com/11084018/160431581-532f511d-a9ca-4bcf-a3ea-fad5e98e4133.png)

To exit this monitor, press **Control + ]**.

### Viewing the Secure Key Storage

The ESP32's secure key storage is managed by a 1024-bit eFuse memory region, subdivided into 4 blocks of 256 bits each. Block 1 and Block 2 are used for secure key storage; Block 1 stores a flash encryption key, while Block 2 stores a Secure Boot key. Block 0 is used to configure the ESP32, while Block 3 can be defined by the programmer. In this lab, we will focus on Block 1 (flash encryption key) and some contents in Block 0 which affect the flash encryption behavior.

To view the eFuse memory contents, simply run:

```
espefuse.py summary
```

This will print the identifier, description, value, and read/write permissions of each eFuse. You should see output similar to below:

![efuse_fields_initial](https://user-images.githubusercontent.com/11084018/160431621-197ecff4-d243-4bb7-9c2f-37ba665e982f.png)

The FLASH_CRYPT_CNT and BLK1 eFuses are most important to flash encryption. FLASH_CRYPT_CONFIG is also important as it affects the behavior of the encryption algorithm. For more information on this algorithm, see [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/security/flash-encryption.html#flash-encryption-algorithm).

## Physical Attack

### Running a Sensitive Application

Now we will run a more sensitive application on the ESP32 and show how credentials can be stolen off the device when flash encryption is not enabled. Download this repository into your workplace directory:

```
git clone https://github.com/PBearson/ESP32_Secure_Key_Storage_Tutorial.git
```

Navigate to the new station project folder and run:

```
idf.py menuconfig
```

This will open a configuration window. You can use the arrow keys to navigate to each option, press Enter to enter a sub-menu, and press ESC to leave a submenu. The controls are also shown at the bottom of the window.

Navigate to the `Example Configuration` menu and enter your WiFi SSID and password. (For context to my screenshots, I have opened a mobile hotspot on my machine and set the SSID to **esplab-hotspot** and password to **ilovebagels**.)

![menuconfig_password](https://user-images.githubusercontent.com/11084018/160431684-f3e81c93-33c3-4f94-abdb-04c8599bd18e.png)

Press ESC to return to the main menu, then ESC again to quit. You will be prompted to save the changes. Press Y to confirm. Now run:

```
idf.py build flash monitor
```

This will build, flash, and monitor your application all in one command. In the serial monitor, you should see that the ESP32 was able to successfully connect to your WiFi network.

![connected_to_wifi](https://user-images.githubusercontent.com/11084018/160431718-3942bde4-fc1b-4280-8257-a2bcadcfb646.png)

### Stealing WiFi Credentials

Now we will assume the role of an attacker who has just gained physical access to the ESP32 and wants to discover the WiFi credentials in the flash. We will soon see that this is all too simple.

First, download the firmware from the ESP32:

```
esptool.py read_flash 0x10000 0x10000 flash_contents.bin
```

This will copy the ESP32's flash contents in the address range 0x10000 - 0x20000 into a file named <ins>flash_contents.bin</ins>.

![read_flash](https://user-images.githubusercontent.com/11084018/160431738-c4a6ab85-ce6c-4f74-a09a-a16e24a7c7cd.png)

Now run:

```
strings flash_contents.bin | grep -A 1 {SSID}
```

Replace {SSID} with your network SSID. You should see the SSID return in red text and password immediately after.

![find_wifi_credentials_in_firmware](https://user-images.githubusercontent.com/11084018/160431760-1c468773-52a4-462e-b2dc-bb5ca12722e3.png)

## Defense Against the Physical Attack

### Generating the Flash Encryption Key

Now we will defend against the physical attack by encrypting the firmware. First, generate the flash encryption key:

```
espsecure.py generate_flash_encryption_key flash_encryption_key
```

A flash encryption key will be generated and stored in the file <ins>flash_encryption_key</ins>.

### Enabling Flash Encryption

Now we will store the key in eFuse Block 1. Run the command:

```
espefuse.py burn_key BLK1 flash_encryption_key
```

This will prompt a warning informing you that the operation is irreversible. This is due to the OTP (one time programmable) nature of the eFuses. Type `BURN` to continue on and wite the key into the block.

![burn_flash_encryption_key](https://user-images.githubusercontent.com/11084018/160431784-aa84b6e0-ffcc-4b52-9b9e-4115594db4d0.png)

There are a few other actions needed to enable flash encryption. Open the project configuration menu:

```
idf.py menuconfig
```

Navigate to the `Security Features` menu and enable the option `Enable flash encryption on boot`. Make sure that the usage mode is set to `Release`, not `Development`. These settings will ensure that the bootloader is compiled to support flash encryption. Now save your configuration (hit ESC a few times, followed by Y to save your changes).

### Running an Encrypted Application

Build and upload the firmware to your ESP32, and open the serial monitor:

```
idf.py build flash monitor
```

Before running the application, the bootloader will detect that flash encryption has been enabled. It will prompt the chip to set the corresponding eFuses in Block 0 and encrypt our application, which starts at flash address 0x10000. This encryption step will take a minute or two, so be patient. You will see output resembling the following:

![flash_encryption_bootloader](https://user-images.githubusercontent.com/11084018/160431813-e35ea1fa-2ec0-4605-91a9-bc5e793487d1.png)

**CAUTION: Do NOT interrupt the power flow or serial monitor while the chip is encrypting the application, or you may corrupt the device.**

When this completes, the device will reboot and run your application, just like before. However, the firmware stored inside the flash region is now encrypted.

Now exit the serial monitor and take a look at the eFuse summary again:

```
espefuse.py summary
```

You will see that FLASH_CRYPT_CNT and BLK1 have been modified:

![efuse_fields_set](https://user-images.githubusercontent.com/11084018/160431839-be4ebc42-06a8-49ca-872e-f84a9a5e7051.png)

FLASH_CRYPT_CNT informs the ESP32 whether or not flash encryption is enabled. It is a 7-bit field, meaning its highest value is 127. If an odd number of bits are set to 1, then flash encryption is enabled. Otherwise, flash encryption is disabled. Based on the value of FLASH_CRYPT_CNT, which is 127 (i.e., all 7 bits are 1), we can see that flash encryption is enabled. Furthermore, eFuse bits are OTP (cannot be changed from 1 to 0, only from 0 to 1). Therefore, FLASH_CRYPT_CNT cannot be changed from the value 127. This means flash encryption cannot be disabled.

BLK1 hides the true value of the flash encryption key due to the secrecy of that data. However, the output (`= ?? ?? ... ?? ?? -/-`) indicates that the key was stored successfully and cannot be read or modified. Trying to write a new key into Block 1 results in an error:

![burn_key_error](https://user-images.githubusercontent.com/11084018/160431859-f500a0c0-43db-41f9-a4fa-c7e1a76ffe08.png)

### Verifying the Defense

Now we again assume the role of an attacker. Download the firmware from the ESP32:

```
esptool.py read_flash 0x10000 0x10000 flash_contents.bin
```

When the download completes, try to find the SSID and password again:

```
strings flash_contents.bin | grep -A 1 {SSID}
```

You will find that this command does not return any output (if it does, you did not enable flash encryption!). That means that our WiFi credentials are safely protected by the flash encryption key, which in turn is protected by the secure key storage.

### Uploading New Firmware

The bootloader will only perform the encryption operation once, so uploading another plaintext firmware will result in soft bricking the device, because the ESP32 will assume flash contents are encrypted and attempt to transparently "decrypt" this content before it reaches the processor.

Let's try to upload the "hello_world" application again. Navigate to the "hello_world" project directory and run:

```
idf.py build flash monitor
```

You will see a "flash read err" message indicating that the firmware cannot be read by the processor.

![flash_read_error](https://user-images.githubusercontent.com/11084018/160431981-5fe5413e-1263-4e0d-ba55-a53672474456.png)

However, since we pre-generated the flash encryption key which is stored in eFuse Block 1, we can encrypt the firmware before uploading it to the board. Copy the flash encryption key into the current directory. Then run the following commands:

```
espsecure.py encrypt_flash_data --keyfile flash_encryption_key --output build/bootloader/bootloader.bin.encrypted --address 0x1000 build/bootloader/bootloader.bin
espsecure.py encrypt_flash_data --keyfile flash_encryption_key --output build/partition_table/partition-table.bin.encrypted --address 0x8000 build/partition_table/partition-table.bin
espsecure.py encrypt_flash_data --keyfile flash_encryption_key --output build/helloworld.bin.encrypted --address 0x10000 build/hello-world.bin
esptool.py write_flash 0x1000 build/bootloader/bootloader.bin.encrypted 0x8000 build/partition_table/partition-table.bin.encrypted 0x10000 build/hello-world.bin.encrypted
idf.py monitor
```

In order, each command:

1) Encrypts the bootloader
2) Encrypts the partition table, which points to the starting address of the firmware
3) Encrypts the firmware
4) Uploads the encrypted files to their appropriate addresses in the flash
5) Opens the serial monitor

You will now see the "hello_world" application running now without any errors. This method shows that as long as you have the pre-generated flash encryption key, you can always upload new firmware to the ESP32.

### Decrypting the Firmware

Return to the WiFi station project in the `Station` directory. Recall that the file <ins>flash_contents.bin</ins> contains our encrypted firmware from the ESP32. Run the following command to decrypt this firmware:

```
espsecure.py decrypt_flash_data --keyfile flash_encryption_key --output flash_contents_decrypted.bin --address 0x10000 flash_contents.bin
```

This will use the flash encryption key to decrypt the firmware and store the plaintext version in the file <ins>flash_contents_decrypted.bin</ins>. Now try to find the SSID and password in this new file:

![find_wifi_credentials_in_decrypted_firmware](https://user-images.githubusercontent.com/11084018/160432011-3fe31709-46ed-42fa-b862-0b630a0b0e4e.png)

It is for this reason that in a true production environment, **the flash encryption key should never be pre-generated**. An attacker who can gain access to this key will have the capability to fully decrypt the firmware on your device. Instead, the flash encryption key should be generated by the ESP32 itself. This actually would have occurred if we had not pre-generated the key beforehand; the bootloader would have signaled the chip to generate the key, and it would have never left the chip.

## Take Away

In this lab, the student:

* Programmed and communicated with the ESP32
* Observed the secure key storage of the ESP32
* Performed a physical readout of the ESP32 flash contents
* Stole WiFi credentials from a firmware
* Enabled flash encryption on the ESP32
* Uploaded new firmware after flash encryption was enabled
* Decrypted the firmware using the pre-generated key
