---
title: "DH/PRR-0"
author: "lemons"
description: "Pocket radio."
created_at: "2026-09-18"
---

# September 20: Start

AM/FM Receiver

originally, i wanted to do a pure passive AM/FM radio, but quickly realized that my idea was pretty stupid considering that there are so many reliable ICs that handle everything for you in a smaller form factor than any discrete solution could make. As such, i landed with the SI4735-D60-GU AM/FM radio IC. This uses a dedicated FM antenna, and relies on a ferrite loop inductor to pick up AM radio. On the SI4735, there is an address selection pin that sets its address to 0x11 or 0x63 or something like that based on whether its pulled high or low. I put a switch on the address but this is probably useless now that i think about it. I saw the crystal the datasheet used for this IC and was confused at first; usually RF ICs have clipped sine wave or some sort of fancy TCXO, however I realized my mistake and found out that it was just an RTC clock, which should have been obvious given its resonant frequency of 32.768kHz.

Microcontroller

Since battery life is of the essence of this project, I naturally gravitated towards the STM32 series of microcontroller, despite having very little experience with it. I actually started this project with the hope that I could familiarize myself with the STM32 before diving into more complex/difficult projects like my other analog video codec project. Thanks to the research I previously did for that other project, I was able to cobble together some information much more quickly, meaning I knew what to look for. The STM32L4+ series instantly jumped out to me, and from there I just chose the smallest packaged one. I knew that this project probably wouldnt require that many GPIO pins considering that the SI4735 already had its own analog audio output pins. That left the STM32L4P5CEU6 free to just handle memory SDMMC and USB D+D- and maybe a display and a few buttons. 

Power Management

This project required a relatively simple 3v3 power solution, or so I thought. Initially, I thought I could just use the BQ24075RGT and get away with 3.7V VBAT nominal, however, USB-A requires 5V operating voltage, and without VBUS power, 5V is nowhere to be found on my PCB. This means I would have to instead use a 7.4V 2s battery, which the BQ24075RGT is not capable of charging. However, this would fix my issue of not having 5v on USB-A, requiring a single step down to 5V and then another to 3v3 for the rest of the circuit. This is rather inconvenient, but i guess the alternative would be putting a noisy boost converter to step up to 5v right before the USB, something I want to avoid with an AM loop antenna. I first landed on the BQ25887RGE as my replacement to the BQ24075RGT, as it supported 2 cell charging and even had a neat feature of automatic sensing(?) I also gave thermistors a try, and ill probably have these functional. Anyways, I quickly realized that this IC didnt have OTG power support, which means that I couldnt use it while charging. This isnt a huge issue, but i also didnt like the idea of that so I switched to the BQ25886RGE, a battery charging IC that did allow for USB OTG through the use of a SYS load pin. Also, because of this new topology, i need a LDO to step 7v4 down to 5V (for USB VBUS), and 5V down to 3v3 for general logic/power. The primary step down is done through an MIC5219-5.0YMM, and then is fed to the TLV75533PDRVR, which is connected to regulated 5V and VBUS. There is a switch from the battery charger VOUT sys pin to V_{sw} or Vswitch that diverts power to the MIC5219-5.0YMM that steps down the battery voltage to 5V, but this is bypassed by VBUS which goes straight to +3V3.

**Total time spent: 6 hours**

# September 21: idk more stuff

Power Management cont.

I have since deleted the other battery charging circuits since they honestly serve no purpose to me as of now, considering that the evaluations I had made last time have since disqualified them from application in my circuit. 

Antenna Considerations

The Si4735, being a dual-band AM/FM radio receiver, requires respective antennas for both bands. With such a small footprint, a good AM loop antenna is rather important, and a small package inductor might not be enough to receive a good signal. Currently the decision is between using a coaxial loopstick antenna inside the housing or using a ferrite core inductor. I did some research with AM receivers, and for the most part, large radios use respective large loop antennas, however I have yet to find clear documentation on how to achieve reliable low-profile AM reception. On the topic of reception though, FM reception is rather simple, as I can just use a telescoping whip antenna, and it will work just fine.

Time justification: reading through a lot of forums. 

**Total time spent: 2.5 hours**

# October 5

Its been a while.

Memory Management

Considering that I plan to use this not only has a pocket radio but as a music player in general, I plan to provide 2 means of memory storage for music in the form of SDMMC and USB. The problem with this is that I dont want to waste an entire USB bus line just for memory. For this reason I selected the MCP2210 USB-to-SPI IC to handle data transfer. Additionally, I also selected the CP2102C, a USB-to-UART programming IC that allows me to free up the USB bus for the BQ25886RGE for power sense. This is connected to my USB-C receptacle, and allows me to program with UART over USB-C. Due to the limited pincount of the STM32L4P5CEU6, I can only support 1 bit SDMMC between the microSD card reader and the MCU. I also added an audio jack, connected to the Si4735 from ROUT/LOUT, and connected them to the ring and tip respectively, and grounded the sleeve. Ill have to impedance match this route later on during PCB layout.

**Total time spent: 3.5 hours**

# October 6

Memory Management cont. 

I made a mistake in my previous journal; the MCP2210 is not a USB-to-SPI memory interface but rather lives in the same vein as the CP2102C as a programming IC that converts flash over USB to flash over SPI. I have since switched to the MAX3421, which is a true USB host IC that can effectively read USB and relay the data over SPI to the microcontroller. Firmware-wise, this probably isnt the best option but I dont imagine ill have many other SPI signals in the DMA channel, since it will just be a (most likely mono e-ink and persistent) display.
