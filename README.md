# ESP32 Secure Key Storage Tutorial

## Goals of this Tutorial

In this tutorial, the student will learn how to use the secure key storage capabilities of IoT devices. The ESP32 will be used in this lab. The student will first learn how to communicate and program the ESP32. Then they will learn how to perform a physical readout of the flash and steal WiFi credentials in the firmware. Finally, the student will learn how to enable flash encryption and protect the firmware against attacks.

### Author's Note

The attack and defense presented in this lab are identical to our [flash encryption tutorial](https://github.com/PBearson/ESP32_Flash_Encryption_Tutorial). However, this lab places a greater emphasis on the ESP32's secure key storage capabilities. We also enable flash encryption in Release Mode instead of Development Mode.

### Overview

* Student will learn about the secure key storage capabilities of the ESP32.
* Student will learn how to perform a physical readout of the ESP32 flash and identify sensitive data in the firmware.
* Student will learn how to enable the flash encryption of the ESP32 by leveraging the secure key storage.

### Hardware Needed
* USB A to Micro USB cable.
* An Hiletgo ESP-WROOM-32 development board.

### Software Needed
* Ubuntu 20.04 disc image (ISO)
* VirtualBox
* ESP-IDF version 4.1 or higher: ESP-IDF is a framework for ESP32 that comes with all necessary software, including the compiler toolchain and various tools for programming and inspecting the ESP32's secure key storage.

A preconfigured VM with all the necessary software can be downloaded [here](http://cyberforensic.net/labs/iot_secure_key_storage/files/ESP32%20Secure%20Key%20Storage%20Lab.ova).

## Environment Setup

### Setting up the Virtual Machine

Download the VM image from the lab materials link. Once it finishes downloading, you need to import it into VirtualBox. In VirtualBox, click on <ins>File → Import Appliance</ins>. Select the downloaded image. Click Next, then Import. After a minute or so, the VM will appear on the left side of the window, as shown below:

![vm_setup](https://user-images.githubusercontent.com/11084018/160431335-67865a6e-2494-490c-9bbb-247b14f10d3a.png)

You can also click <ins>Settings</ins> to modify the VM before booting it up. By default, it will use 8 GB of RAM and 2 CPU cores, so change these settings to suit your environment. Once you are done, click <ins>Start</ins> to boot up the machine.

### Verifying the Setup

TODO

![vm_verify](https://user-images.githubusercontent.com/11084018/160431442-33fa2cc7-35ff-4f91-8216-c099e1f2f584.png)

### Setting up the Serial Port

TODO

![usb_device_highlighted](https://user-images.githubusercontent.com/11084018/160431513-580652e5-1bf5-4e42-8bec-a4af1f95f1a2.png)

### Running the Example Application

TODO

![project_build_complete](https://user-images.githubusercontent.com/11084018/160431536-f400cc48-5f35-46b8-beca-4d2794655456.png)

![running_flash_command](https://user-images.githubusercontent.com/11084018/160431565-1a5d21c0-f227-48d9-b4e5-1198493d3c2e.png)

![output_monitor](https://user-images.githubusercontent.com/11084018/160431581-532f511d-a9ca-4bcf-a3ea-fad5e98e4133.png)

### Viewing the Secure Key Storage

TODO

![efuse_fields_initial](https://user-images.githubusercontent.com/11084018/160431621-197ecff4-d243-4bb7-9c2f-37ba665e982f.png)

## Physical Attack

### Running a Sensitive Application

TODO

![menuconfig_password](https://user-images.githubusercontent.com/11084018/160431684-f3e81c93-33c3-4f94-abdb-04c8599bd18e.png)

![connected_to_wifi](https://user-images.githubusercontent.com/11084018/160431718-3942bde4-fc1b-4280-8257-a2bcadcfb646.png)

### Stealing WiFi Credentials

TODO

![read_flash](https://user-images.githubusercontent.com/11084018/160431738-c4a6ab85-ce6c-4f74-a09a-a16e24a7c7cd.png)

![find_wifi_credentials_in_firmware](https://user-images.githubusercontent.com/11084018/160431760-1c468773-52a4-462e-b2dc-bb5ca12722e3.png)

## Defense Against the Physical Attack

### Generating the Flash Encryption Key

TODO

### Enabling Flash Encryption

TODO

![burn_flash_encryption_key](https://user-images.githubusercontent.com/11084018/160431784-aa84b6e0-ffcc-4b52-9b9e-4115594db4d0.png)

### Running an Encrypted Application

![flash_encryption_bootloader](https://user-images.githubusercontent.com/11084018/160431813-e35ea1fa-2ec0-4605-91a9-bc5e793487d1.png)

![efuse_fields_set](https://user-images.githubusercontent.com/11084018/160431839-be4ebc42-06a8-49ca-872e-f84a9a5e7051.png)

![burn_key_error](https://user-images.githubusercontent.com/11084018/160431859-f500a0c0-43db-41f9-a4fa-c7e1a76ffe08.png)

### Verifying the Defense

TODO

### Uploading New Firmware

TODO

![flash_read_error](https://user-images.githubusercontent.com/11084018/160431981-5fe5413e-1263-4e0d-ba55-a53672474456.png)

### Decrypting the Firmware

TODO

![find_wifi_credentials_in_decrypted_firmware](https://user-images.githubusercontent.com/11084018/160432011-3fe31709-46ed-42fa-b862-0b630a0b0e4e.png)

## Take Away

TODO
