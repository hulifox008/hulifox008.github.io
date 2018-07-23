---
layout: default
title: "Recover BIOS of a bricked Lenovo G50-70"
date:  2018-07-21 16:00:00 -0600
---
## Recover BIOS of a bricked Lenovo G50-70

I cannot remember what exactly happened to this laptop. It just went black screen when upgrading Windows 8, and never came back. Now, every time I turn on the power, the power LED will light up, but the screen will not turn on. You can notice the screen is flashing, even though there's no back light.

After some researching online, it seems like there's an emergency bios recovery mode, by pressing and holding down the Fn+R key, then turn on the power. I was able to get a copy of the recovery bios, named aclu1x64.fd, put it on a thumb drive, plug it into a USB port. I'm pretty sure the emergency mode is activated, since I can see the screen is not flashing, and thumb drive flashes a little bit after powering on. The thumb drive will stop flashing for a while and start flashing again. I though that's the system reading the thumb drive. But the system will suddenly reboot just after the thumb drive started flashing. You can tell that by noticing the screen starts flashing again...Well, maybe the bios is damaged so bad, that the emergency mode cannot work. Or, maybe it's not the bios at all. Anyway, I decided to reflash the BIOS using a external programmer.

First, remove the back cover and keyboard. There are some flat cables between the keyboard and main board. They are very difficult to remove and put back, since the space between the keyboard and main board are pretty tight. I don't know what method the used at manufacturing, but it did take me a lot of time to remove and put them back. After the main board is removed. You can locate the BIOS chip pretty easily. It's the only SOIC8 package on the board and is label "U3". This is an 8MB SPI flash rom. Instead of de-soldering the chip from the board, I ordered a SOIC8 clip from Amazon. The I used a FT2232H based breakout board as the USB-SPI adapter. The programming software is [flashrom](https://github.com/flashrom/flashrom). The whole setup looks like:

![recovery_image1]({{site.url}}/assets/bios_recovery_1.jpg)

The green board is a FT2232H breakout board. And this is how the clip attached to the chip:

![recovery_image2]({{site.url}}/assets/bios_recovery_2.jpg)

After every thing is connected. I was able to run flashrom, and it detected the chip without problem. Then I read out the whole chip and save it to a file. I then load the file to UEFItool, and immediately found out there's a section had checksum error and failed to decompress. I managed to extract the 8MB bios image from the latest BIOS upgrade package, downloaded from Lenovo website. And also open it with UEFItool. I then only replace the damaged section in the backup BIOS, with the corresponding section extracted from the latest bios. After writing the fixed BIOS back to the SPI rom, I connected the power and LCD screen. The LCD lighted up after turning power on!! Put everything back, the laptop is working again!
