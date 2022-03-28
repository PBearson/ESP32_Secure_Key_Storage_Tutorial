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
