# ESPHome Leak Sensor

Physical water leak sensor device that can be deployed in your home using [Home Assistant](https://www.home-assistant.io/) and [ESPHome](https://esphome.io/).


## Prerequisites

Before you get started, you will need the following. Getting these things set up and configured is outside the scope of this project.

1. You have a Home Assistant server set up in your home
2. You have installed the [HACS](https://hacs.xyz/) plugin in your Home Assistant instance
3. You have installed the ESPHome plugin using HACS
4. _Optional:_ You have a soldering iron and know how to solder


## Device

### Parts

You will need the following to build each sensor device:

* 1x [Adafruit Feather HUZZAH with ESP8266](https://www.adafruit.com/product/2821)
* 1x 10k Ohm resistor
    * _Note: The Feather board contains internal pull-up resistors that can be enabled programmatically, but they seem to be insufficient for this use case based on my testing._
* 2x [1N4001 diodes](https://www.adafruit.com/product/755)
    * _Optional: Needed if you are connecting two or more sensors to the same board/device._
* 1x DC buzzer [like this one](https://www.adafruit.com/product/1536)
    * _Optional: Needed if you want to program your device to emit an audible sound if a leak is detected._
* 1x perfboard [like this one](https://www.adafruit.com/product/2670)
    * _Optional but Recommended: Needed if you want to create a tidy backpack that contains the pull-up resistor, optional diodes, and pin headers to plug the Feather board into._
* 1x Micro-USB cable
    * _Note: Get a cable that is long enough to reach from the wall plug to wherever the board/device will be placed. If that distance is more than a few feet, you might need to get a specialty cable [like this one](https://smile.amazon.com/dp/B081TQTXVC)._
* 1x USB wall charger [like this one](https://smile.amazon.com/dp/B07L1N7RG8)
* 2-pin insulated wire to connect device to detector strips
    * _Optional: Needed for tidyness if the location of the potential leak is more than a few inches away from the board/device itself. Note that you can get something [like this 4-pin cable](https://smile.amazon.com/dp/B01M0QTT7B) and pull it in halves to get two 2-pin cables that you can cut to length._
* 2-pin connector cable plugs (JST-style [plug](https://www.adafruit.com/product/261) and [jack](https://www.adafruit.com/product/1769) OR [pig-tail cables](https://www.adafruit.com/product/1003) to plug directly into header pins)
* 4mm-wide conductive fabric tape [like this one](https://smile.amazon.com/dp/B092G4NC6K)
    * _Optional: Needed if you have hard floors and want to detect any leaks along a long part of the floor (like along an entire wall) instead of a single point._
    * _Note: This tape will stick to hard floors in two parallel strips to detect leaks, so get enough for double the length of the wall(s) that you want to monitor._


### Assembly

The following diagrams were generated from the [`Leak Sensor Device.fzz`](Leak%20Sensor%20Device.fzz) file in [Fritzing](https://fritzing.org/).

#### Schematic

![electronic circuit schematic diagram](Leak%20Sensor%20Device_schem.png?raw=true "Leak Sensor Device circuit schematic")

#### Breadboard

![breadboard diagram](Leak%20Sensor%20Device_bb.png?raw=true "Leak Sensor Device breadboard diagram")


## Configuration

In the ESPHome tab (in your Home Assistant app):

1. Click the `+ New Device` button in the bottom-right corner
1. Name the new device something like `leak-detector-1` and click _Next_
1. Choose _Pick specific board_, pick `Adafruit HUZZAH ESP8266` from the drop-down menu below it, and click _Next_
1. Click _Skip_ to skip the install for now
1. Click the _Edit_ button below your new device and edit its YAML configuration as follows...
1. Add a [GPIO Binary Sensor](https://esphome.io/components/binary_sensor/gpio.html) as a _moisture_ sensor on pin `GPIO4` (inverted):

    ```
    binary_sensor:
      - platform: gpio
        device_class: moisture
        name: "Leak Sensor 1A"
        pin:
          number: GPIO4
          inverted: true
          mode:
            input: true
            pullup: true
        filters:
          - delayed_on: 1000ms    #  1 second
          - delayed_off: 10000ms  # 10 seconds
    ```

1. Add a second GPIO Binary Sensor on pin `GPIO12` if you want a second leak sensor on the same device:

    ```
    # after binary_sensor:
      - platform: gpio
        device_class: moisture
        name: "Leak Sensor 1B"
        pin:
          number: GPIO12
          inverted: true
          mode:
            input: true
            pullup: true
        filters:
          - delayed_on: 1000ms    #  1 second
          - delayed_off: 10000ms  # 10 seconds
    ```

1. _Optional:_ Add a [GPIO Switch](https://esphome.io/components/switch/gpio.html) if you want to turn on a (built-in or external) LED when a leak is detected:

    ```
    switch:
      - platform: gpio
        name: "Leak LED 1"
        id: leak_led_1
        pin:
          number: GPIO2
          inverted: true
    ```
    
    Then add the following [`on_press`](https://esphome.io/components/binary_sensor/index.html#binary-sensor-on-press) and [`on_release`](https://esphome.io/components/binary_sensor/index.html#binary-sensor-on-release) [automations](https://esphome.io/guides/automations.html) to the bottom of each of your binary sensors:
    
    ```
    # binary_sensor:
    # - platform: gpio ...
        on_press:
          then:
            - switch.turn_on: leak_led_1
        on_release:
          then:
            - switch.turn_off: leak_led_1
    ```

1. When you are done editing the configuration, click _Install_ in the bottom-right corner
    1. The first time you install an ESPHome configuration to a new device, you will have to plug the Feather board directly into your Home Assistant server via a USB cable and choose _Plug into the computer running ESPHome Dashboard_.
    1. After that, you can choose _Wirelessly_ instead to install via Wi-Fi as long as the device is powered and has the _Online_ status next to it.
1. Wait for the installation to complete and the device to restart. If the installation was successful, you should see a new Notification in Home Assistant.
1. Click on the _Notifications_ tab and find the notification about Home Assistant detecting new devices
1. Click _Check it out_ in the notification to go to the Integrations Settings page
1. Click _Configure_ next to the new ESPHome device (it should be the first one listed) to add the leak sensor device and entities to your Home Assistant
1. _Example Next Step:_ Create a Home Assistant [Automation](https://www.home-assistant.io/docs/automation/) to send you a notification if a leak is detected by the `binary_sensor.leak_sensor_1a` or `binary_sensor.leak_sensor_1b` entity:

    ```
    alias: Leak in Kitchen Notification
    description: >-
      Send a push notification if a leak is detected in the kitchen.
    trigger:
      - type: moist
        platform: device
        entity_id: binary_sensor.leak_sensor_1a
        domain: binary_sensor
      - type: moist
        platform: device
        entity_id: binary_sensor.leak_sensor_1b
        domain: binary_sensor
    action:
      - service: notify.notify
        data:
          message: A leak has been detected in the kitchen!
    mode: single
    ```

**Note:** See [`leak-detector-1.yaml`](leak-detector-1.yaml) for the full example ESPHome device YAML configuration file.


## Circuit Simulation

I used a circuit simulator web app to create this simplified version of the leak sensors circuit to make sure the resistor values and diodes I used allowed allowed enough voltage to pass through each sensor when tripped.

![simplified sensor circuit diagram](Dual%20GPIO%20Resistive%20Buttons.png?raw=true "Dual GPIO Resistive Buttons simulation screenshot")

* The 10k Ohm resistor is the single pull-up resistor
* The 1k Ohm resistors represent the estimated resistance through two parallel 5-feet-long strips of conductive fabric tape
* The 33M Ohm resistors represent the high-impedance input pins on the microcontroller
* The two switches represent two separate dual-strip leak sensors (e.g. for detecting leaks in two separate areas or along two distinct walls)

You can find the live simulation of this circuit by [clicking here](https://falstad.com/circuit/circuitjs.html?ctz=CQAgjCAMB0l3BWcMBMcUHYMGZIA4UA2ATmIxAUgoqoQFMBaMMAKACcRs8AWTyXrgIQooyeCzAYRgvgJ4hMIquWydRVGEgBqAewA2AFwCGAczrtO2QiG5xL17tzyjsueHBYn7Np9+6FeKhQWAHdvK2dXBztgr25sEUdneMSA0WCOFJs7LKSXN3dQmwTsqiyIhRYAE3DCSPkKkSq6ADMjAFdDauLpOs4GvqbWjq6AZwUpTj6oqecqCAM2dvNx8r7uYmsK+ZBF5c8JxN9FGzSNA42HX0vTwKgimWxhfoE+yAsT2yDJvPnxTM2pRsgN+YneYROjR+dnBwK26xBMJYQA). If the link doesn't work you can also try [going here instead](https://falstad.com/circuit/circuitjs.html) and click _File > Open File..._ to import the [`Dual GPIO Resistive Buttons.circuitjs.txt`](Dual%20GPIO%20Resistive%20Buttons.circuitjs.txt) file from this project.


## Future Work

* Add surface-mount JST 2-pin connectors to PCB so leak sensor wires can be easily connected and disconnected from backpack board
    * This might require moving the perfboard/PCB from behind the Feather to in front of the feather to have enough room for the JST connectors
    * Alternatively, pig-tail connectors should work with the existing design since they could plug direclty into the pin headers poking through the Feather board
* Add a DC buzzer to the schematics and explain how to use it with ESPHome
