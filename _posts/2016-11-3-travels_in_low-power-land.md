## Travels in Low-Power-Land
After having build two small wireless sensors for specialiced uses, we found more small use cases for something "physical". The we came up with a more generic approach to develop a generic sensor board that fullfills the following requirements:

* ASAP (As small as possible ;-) )
* Low-power, works for months with one CR2032 coin cell or similar on average use of 2-3 times per day
* Fast reaction on user action (<200ms)
* Works in an area of 100 x 100 m

Besides simple GPIO-Inputs for Buttons etc., it could contain the following sensors for many nice ideas:

* Accelerometer
* Compass
* Brightness / RGB
* Capacitive touch

We evaluated two technolgies as base:

### ESP8266

Esp8266 is a tiny SoC with Wifi and several GPIO-Pins. It's available on a couple of breakout boards, starting at about 2 € per piece.
What makes it really interesting is the very fast WiFi connection speed. After waking up from deepsleep, it connects to WPA2-Wifi with a static IP and publishes the first message in about 200ms.

It's completely 3,3V driven.
It is definetely a great value for the price.

### Arduino + RFM69HCW

