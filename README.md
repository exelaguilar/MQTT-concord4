# MQTT-concord4

This is based off the original [device-concord4 here](https://github.com/automaton82/device-concord4).

Which is forked from the [original here](https://github.com/csdozier/device-concord4).

The original concord4 was made for SmartThings via a web-server. Since SmartThings is now sun-setting classic along with custom DTH UIs, it no longer works.

As such I'm moving to Home Assistant and still want to use this integration. Since MQTT is very easy to use with HA, I've modified the primary server to use MQTT instead of RESTful.

## Prerequisites

 - Hardware (Concord 4 or equivalent panel) with a Superbus 2000 automation module attached to it
 - RS232 connection (to the AM panel)
 - Python 2.7
 - Python packages: requests, future, pyserial (pip install), paho-mqtt (pip install)
 - Raspberry Pi (recommended)

## Installation

1. Edit *concordsvr_mqtt.conf* with your favourite editor, such as *nano concordsvr_mqtt.conf*
    * Set *host* to the host of your MQTT, and the *port*
    * Set *mqttuser* to the base64 encoded name of the user to login, and *mqttpassword* to the base64 encoded password of that user
2.  Start the program using **python concordsvr_mqtt.py**
3.  If desired, it can be started on every boot by using *crontab -e* and adding:
    * *@reboot cd /home/pi/ && ./start_concordsvr &*
    
    Where *start_concordsvr* contains:
    * *#!/bin/bash*
    * *cd /home/pi/concordsvr/ && python ./concordsvr_mqtt.py*

## Home Assistant

You can create entities in Home Assistant to represent the various sensors by [adding MQTT library to HA](https://www.home-assistant.io/integrations/mqtt/), and then editing the *configuration.yaml* file to include something like:

    # MQTT binary sensor
    binary_sensor:
        - platform: mqtt
        name: "Front Door"
        state_topic: "concord/zone/1"
        payload_on: "open"
        payload_off: "closed"
        device_class: opening

This adds the front door sensor as an entity you can put on any dashboard.

The [alarm entity can be created](https://www.home-assistant.io/integrations/alarm_control_panel.mqtt/) with the following item in the configuration:

    # Alarm setup
    alarm_control_panel:
        - platform: mqtt
        state_topic: "concord/alarm"
        command_topic: "concord/alarm/set"
        code_arm_required: false
        code_disarm_required: false

## MQTT

The topic listened to is *concord/#*. The topic to set is:

* concord/alarm/set

With payloads of:

* arm_home
* arm_away
* disarm

The device will send out messages like:

* concord/zone/x

Where x is the zone number, like 1. Payload is 'open' or 'closed. It also sends:

* concord/alarm

With payloads of:

* armed_home
* armed_away
* disarmed


Sending keypresses to the Concord System:

* concord/keypress

This topic allows you to send commands to the system. 

Reference the KEYPRESS_CODES under concord_commands.py. 

You can leverage MQTT publish in Home Assistant if you use that. 

    service: mqtt.publish
    service_data:
      topic: concord/keypress
      payload: 1

You can use a hex to decimal number and use that number as your payload (0x20 => 32; payload = 32)


* concord/keypad

You can set up a sensor in Home Assistant to read the keypad and it will update every minute or with any keypress. 

    sensor:
      - platform: mqtt
        name: "Concord Keypad"
        state_topic: "concord/keypad"


## Notes

The previous ST version supported 'loud' as an option for arming / disarming, but I didn't implement that as my panel doesn't support it anyways. If desired it could be added back as the payload for the various topics.
