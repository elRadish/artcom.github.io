---
layout: post
title: Phygital @ ART+COM
---
Building a small wireless sensor for a prototyped "Phygital magazine"


## Preface

The goal of the described project is to build a small short-distance wireless sender with touch sensor that can be hidden in a printed magazine to make it "Phygital" = Physical and Digital. The user sees a printed advertisment with an integrated touch area. If he touches the area in the paper, the corresponding product is offered on a nearby screen and can be customized by the reader.

## Requirements
* Small
* As simple as possible
* Maintenance free
* Power efficient 
* Fast reaction on user action
* No LiPo batteries because of fire hazard while charging them (especially between paper)

## Wireless technology
The follwoing criteria were taken into account when choosing the wireless technology:

* Communication needs to be uni-directional only.
* No sensitive data is transferred. Encryption is not neccessary.
* The distance between sender and receiver is just a few meters.
* The sender hardware should be small.
* The technology should be legal and licence free
* No constant connection to base station (like WiFi or Bluetooth) to save energy. 
* No time consuming handshakes to establish a connection.

We chose a very simple 433Mhz [ASK-Modulated](https://en.wikipedia.org/wiki/Amplitude-shift_keying) technology, often called "RFM433". There are very similar modules sold for a few Euros. They are very similar to the 433Mhz wireless modules used for wireless power outlets, for cheap headphones, simple doorbells etc.
They work _nearly_ like a wire - they just transmit the level on the transmitter's input pin to the receiver's output pin. In an ideal world, we could just use them as wire and connect them to the UART-pins of the microcontroller. That might work sometimes, but is not reliable.

![image](https://github.com/elRadish/artcom.github.io/blob/phygital/images/2016-7-26-Phygital/rf433.jpg)

The whole transmitting logic is simple and can be done in software. It has to deal with bad receiption and deals with things like sending a preamble to give the receiver a chance to adjust its auto gain control to the signal. Additionally, the data is encoded 4-to-6 bits to achieve a better DC-balance for wireless transmission.

There are a couple of useful libraries for that purpose. VirtualWire, ManchesterTX, RadioHead. I chose VirtualWire, since it has a small footprint and is well documented. As a nice feature, it handles transimtting via interrupt service routines in the "background" (If we can talk about "background" on a 8 Mhz single core microcontroller ;-) ).

### Sender

The sender is pretty small. Is gives the best range when equipped with a plain lambda/4 wire antenna (17,3cm). It has no power consumption as long as it isn't transmitting.

![image](https://github.com/elRadish/artcom.github.io/blob/phygital/images/2016-7-26-Phygital/sender.png)
![image](https://github.com/elRadish/artcom.github.io/blob/phygital/images/2016-7-26-Phygital/senderCr2032_schema.png)

### Receiver

The receiver doesn't need to be very power efficient. It can be powered through 5V or through PoE. It is connected to the backend via ethernet and functions as a bridge from to MQTT. It simply forwards received messages to a given topic.

Additionally, it has a RGB-LED to show its current state:
* Red: Not connected to network
* Yellow: Connected to network, bootstrapping
* Green: Connected to MQTT-Broker
* Blue: Receiving wireless message
* Flashing pink: Transmitter battery low

![image](https://github.com/elRadish/artcom.github.io/blob/phygital/images/2016-7-26-Phygital/receiver.png)


## Touch
To keep it as simple as possible, I preferred the most simple touch principle: Resistive touch between two (nearly) invisible contacts. They are painted with black conductive color onto a black area of the magazine page. Contact is made with self adhesive copper tape on the back of the page.
If the contacts are bridged by a finger, the current is enough to wake up the microcontroller from deep sleep.

(After redesigning the page and printing it with a professional inkjet printer, we expercienced that black printing ink is pretty conductive :-(. Probably it contains some RUSS particles.
We had to reprint it with a laser printer.)

## MCU
I used a AtMega328@16Mhz on a Arduino Pro Mini PCB. It is small and can be made very power efficient when removing some parts (Power regulator, Power LED). The board can be powered directly by the CR2032 coin cells.

## Power consumption

### Sleep modes
Atmel microcontrollers can be very power efficient. It is possible to send them to deep sleep and to disable many integrated units to minimize the power consumption from about 25mA to few microamps. They can be activated by interrupts and watchdog timers.

After switching off as much as possible (Analog-Digital-Converter, Brown-out-detection etc.) power consumption went down to 30uA. That gives around 200-250 days of standby time with 2 CR2032 coin cells. 

![image](https://github.com/elRadish/artcom.github.io/blob/phygital/images/2016-7-26-Phygital/sender_lifecycle.png)

## Summary
That tiny project gave me a chance to make some first steps in low-power-land. 
The technology has been working now for months without problems.
The RF433 technology proved to be reliable enough for that use case - but there are actually better choices on the market. The new RFM69 modules from hopeFM have the same size - but they provide much more reliabilty, are smaller and have bidirectional transmitting including auto gain adaption, acknowledge-management and AES128 encryption.

In retrospective, it hardly makes sense to use a complete Arduino Board to remove most of the parts (LED, regulator) to save power afterwards. I would rather chose a plain Atmel MCU like AtTiny 85 or AtMega168 or 328.
That could be programmed with an Arduino Bootloader for easier deployment through USB if required or simply by ISP connector. 