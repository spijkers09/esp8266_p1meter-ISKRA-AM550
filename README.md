# esp8266_p1meter for ISKRA AM550

Software for the ESP2866 that sends P1 smart meter data to a mqtt broker, specifically adjusted for ISKRA AM550. The software changes are limited (removed the inversion). The hardware is updated big time. 

## about this fork
This fork (tries) to add support for the `ISKRA AM550` smartmeter (DSMR5.0). It is based on the fork by Daniel Jong https://github.com/daniel-jong/esp8266_p1meter. It also used some of the info from https://github.com/psvanstrom/esphome-p1reader

3 April 2025: Its working on a breadboard, but I ordered PCB to make a compact version and will create a 3D printable box.

2 May 2025: After some delay I received the PCB I designed. Turned out the bases I used was not entirely accurate, so it was 2 mm wider then the ESP8266. I have changed the design of the PCB to match the ESP8266. The correct design is in the directory.
If you want to use a breadboard, use the schematic.pdf

BE AWARE: I lost 1 ESP8266 because I soldered the PCB directly to the ESP before I had uploaded the software. Not sure why, but then upload is no longer working. Make sure you use the provided pinheaders to make it detachable. That will also provide future update possibilities.

The ![original source](https://github.com/fliphess/esp8266_p1meter) has issues with DSMR5.0 meters who like to send telegrams every 1 second at a high 115200 baud rate. 
This causes the used SoftwareSerial to struggle to keep up and thus only receives corrupted messages. This fork switches to using the main Hardware serial port (RX) for communication with the meter.

# Getting started
This setup requires:
- An esp8266 (I used Wemos d1 mini V4 with USB-C)
- A 10k ohm resistor
- A 4,7k ohm resistor
- A BC547 NPN transistor
- A modular RJ12 jack (see datablad)
- A [6 pin RJ12 cable]

Compiling up using Arduino IDE: 
I will try and extend this section, as it was the first time I actually did something with IDE.
- Ensure you have selected the right board. I selected LOLIN (Wemos) D1 Mini (clone)
- Using the Tools->Manage Libraries... install `PubSubClient (by Nick)` and `WifiManager (by tzapu)`. I installed several other libraries (at Arduino's request, for example ESP82266Autowifi)
- Verify the software. This should indicate errors (like missing functions/libraries)
- Connect the board to your computer. Under <Tools>/Com port select the correct port. For some reason it showed 2 in my case, then just try to select one.
- If the upload does not work, select the other
- Upload the software

I removed the PlatformIO version as I have no experience and have not used it. Please refer to the origin of this fork if you want to use it.

Finishing off:
- You should now see a new wifi network `ESP******` connect to this wifi network, a popup should appear, else manually navigate to `192.168.4.1`
- Configure your wifi and Mqtt settings
- It is usefull to create a separate user in HomeAssistant for MQTT. I created the user mqttuser and gave a password.
- The mqtt host adress is your homeassistant IP and use 1883 for the port.
- You can use the serial monitor function of Arduino IDE (also under <Tools>) to see the ESP waking up and trying to connect.
- To check if everything is up and running you can listen to the MQTT topic `hass/status`, on startup a single message is sent.

## Connecting to the P1 meter
Connect the esp8266 to an RJ12 cable/connector following the diagram.

2 May 2025: Schematic.pdf is the electrical schematic. You can use it wire a breadboard. P1meterv2.kicad_pcb is the updated file I used to order the PCB.
It uses the open source KiCad software.
The datablad PDF suggests a modular jack to use.

BE AWARE: most jacks are reversed (locking clip below/at PCB side). I used the wrong, reversed one (see the pictures in the assets directory), so I had to rotate 1 of the connectors 180. Which means a standard RJ12 cable will not work with the setup. Either order a correct jack or an RJ12 shrink tool.

## Box

30 May 2025: I found a really great tool to create a 3D box for PCB's. As mine is a stacked one, I had to fiddle with the settings to make it fit. It only took me 2 runs to create the perfect box.

This is the website of the creator of the tool. There is some explanation how to use it.

https://willem.aandewiel.nl/index.php/2022/01/02/yet-another-parametric-projectbox-generator/

The download location of the tool
https://github.com/mrWheel/YAPP_Box

The download location of the program used
https://openscad.org/downloads.html

In the assets I uploaded the tool as a zip (YAPP) and the source file of my box.

## Data Sent

All metrics are send to their own MQTT topic.
The software sends out to the following MQTT topics:

```
sensors/power/p1meter/consumption_low_tarif 2209397
sensors/power/p1meter/consumption_high_tarif 1964962
sensors/power/p1meter/returndelivery_low_tarif 2209397
sensors/power/p1meter/returndelivery_high_tarif 1964962
sensors/power/p1meter/actual_consumption 313
sensors/power/p1meter/actual_returndelivery 0
sensors/power/p1meter/l1_instant_power_usage 313
sensors/power/p1meter/l2_instant_power_usage 0
sensors/power/p1meter/l3_instant_power_usage 0
sensors/power/p1meter/l1_instant_power_current 1000
sensors/power/p1meter/l2_instant_power_current 0
sensors/power/p1meter/l3_instant_power_current 0
sensors/power/p1meter/l1_voltage 233
sensors/power/p1meter/l2_voltage 0
sensors/power/p1meter/l3_voltage 0
sensors/power/p1meter/gas_meter_m3 968922
sensors/power/p1meter/actual_tarif_group 2
sensors/power/p1meter/short_power_outages 3
sensors/power/p1meter/long_power_outages 1
sensors/power/p1meter/short_power_drops 0
sensors/power/p1meter/short_power_peaks 0
```

## Home Assistant Configuration

Use this [example](https://raw.githubusercontent.com/spijkers09/esp8266_p1meter-ISKRA-AM550/master/assets/p1_sensors.yaml) for home assistant's `sensor.yaml`

The automations are yours to create.
And always remember that sending alerts in case of a power outtage only make sense when you own a UPS battery :)

## Thanks to

This sketch is mostly copied and pasted from several other projects.
Standing on the heads of giants, big thanks and great respect to the writers and/or creators of:

- https://github.com/jantenhove/P1-Meter-ESP8266
- https://github.com/neographikal/P1-Meter-ESP8266-MQTT
- http://gejanssen.com/howto/Slimme-meter-uitlezen/
- https://github.com/rroethof/p1reader/
- http://romix.macuser.nl/software.html
- http://blog.regout.info/category/slimmeter/
- http://domoticx.com/p1-poort-slimme-meter-hardware/
