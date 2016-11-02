---
layout: post
title: Phygital @ ART+COM
---
Building a small wireless sensor for a prototyped "Phygital magazine"


## Preface

The goal of the described project is to build a small wireless short-distance sender that is can be hidden in a printed magazine. It is connected to tiny contact pads printed on an advertisment page. When these pads are touched, a corresponding signal is sent to a receiver that forwards it to the network.
Although I had some experience with small microcontroller projects before, I never had to take battery consumption into account. 

## Goals
* Small
* As simple as possible
* Maintenance free. 
* Power efficient 
* Delay-free
* No LiPo batteries because of fire hazard while charging them (especially between paper)

## Wireless technology
The follwoing crieterias were taken into account when choosing the wireless technology:

* To save as much power as possible, any wireless technology that has to maintain a constant connection to base station like WiFi or Bluetooth AUSSCHEIDEN. Establishing the connection on demand was assumed to take take too long (after having evaluated the ESP8266, I might give it a chance for something like that -> see LINK).
* Communication has to be uni-directional only.
* No sensitive data is transferred.
* The distance between sender and receiver is just a few meters.
* The sender hardware should be small.
* The technology should be legal and licence free

We chose a very simple 433Mhz ASK-Modulated technology, often called "RFM433". There are very similar modules sold for a few Euros. They are very similar to the 433Mhz wireless modules used for wireless power outlets, for cheap headphones, simple doorbells etc.
They can be considered a _little_ like a wire - they just transmit the level on the transmitter's input pin to the receiver's output pin. In an ideal world, we could just use them as wire and connect them to the UART-pins of the microcontroller. That might work sometimes, but not reliable.

The whole transmitting logic is pretty simple and done in software. It has to deal with bad receiption and has to do some things like sending a preamble to give the receiver a chance to train its auto gain control to the signal. Additionally, the data is encoded 4 to 6 bits to achieve a better DC-balance for wireless transmission.

There are a couple of useful libraries for that purpose. VirtualWire, ManchesterTX, RadioHead. I chose VirtualWire, since it has a small footprint and is well documented. As a nice feature, it handles transimtting via interrupt service routines in the "background" (If we can talk about "background" on a 8 Mhz single core microcontroller ;-) ).

## Touch
There was a prior prototype of the magazine that worked with a capacitive touch sensor. The problem was that it couldn't distinguish between the paper of the magazine when it is opened /closed and the finger and sent many gost touches. 

To keep it as simple as possible, I preferred the most simple touch principle known: Resistive touch between two (nearly) invisible contacts. They are painted with black conductive color onto a black area of the magazine page. Contact is made with self adhesive copper tape on the back of the page.
These contacts are connected to ground and to an interrupt-enabled pin. Normally, it is pulled down by 10 MOhm resistor. When touched, skin resistance is low enough to pull it up to high level to detect the touch.

After redesigning the page and printing it with a professional inkjet printer, we expercienced that black printing ink is pretty conductive :-(. Probably it contains some RUSS particles.
We had to reprint it with a laser printer.

## MCU

## Power consumption

The sender is powered by two CR2032 coin cells to provide ca. 6 Volts for the transmitter.

### Sleep modes
Atmel microcontrollers can be very power efficient. It is possible to send them to deep sleep and to disable many integrated units to minimize the power consumption from about 25mA to few microamps. In that case, the AtMega is woken up by a high level on one pin.
Normally, Arduino PCBs are not made to be very power efficient. So I had to remove some parts on the Arduino Pro Mini PCB: Power Led and voltage regulator. The board can be powered directly by the CR2032 coin cells with a simple diode to drop the voltage a little.

After switching off as much as possible (Analog-digital-converter, brown-out-detection etc.) power consumption went down to 30uA. That's fine and gives around 250 days of standby - which is pretty fine. The whole thing is used only a few times a day - so it should still give about 200 days of battery lifetime during normal use.

## Summary
That tiny project gave me a chance to discover the first coasts of low-power-land. It is really fun to drive things to even lower currents and to minimize the footprints of the boards. The 