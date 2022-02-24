# ESPHome IR reciever & transmitter

## Description:
You can use the old remote to control other smart devices connected to Home Assistant. 

For example, changing lights, opening doors, sending camera snapshots to the TV and more.

I have the following configuration on an ESP8266 connected via relay to an old motherboard from a laptop running libreelec OS with Kodi.

Using the relay via USB port, the program recognizes whether the board or TV is on or off and performs adequate automation accordingly.

[**ESPHome library for IR transmitter**](https://esphome.io/components/remote_transmitter.html)

[**ESPHome library for IR reciever**](https://esphome.io/components/remote_receiver.html)

## Basic wiring:

![Schema](https://github.com/peca2345/ESPHome-IR-reciever/blob/main/IMG/schema.png?raw=true)

## Photo:

![MB](https://github.com/peca2345/ESPHome-IR-reciever/blob/main/IMG/TVMB-IR.png?raw=true)

## ESPHome code:

```
esphome:
  name: home-tv-server
  on_boot:
    if:
      condition:
        - binary_sensor.is_on: TVMB_USB_TV
        - binary_sensor.is_off: TVMB_USB_MB
      then:
        - delay: 3s
        - switch.turn_on: TVMB_relay_DC # preventivni zapnuti (mozna uz bylo zapnuto)
        - delay: 1s
        - switch.turn_on: TVMB_run
      else:
        - if:
            condition:
              - binary_sensor.is_off: TVMB_USB_TV
              - binary_sensor.is_on: TVMB_USB_MB
            then:
              - delay: 3s
              - switch.turn_off: TVMB_relay_DC

esp8266:
  board: esp01_1m

logger:
  level: DEBUG
api:
ota:
  password: "030d1163de9f2c43228895a7d801d306"
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
captive_portal:

## IR RECIEVER ###################################

remote_receiver:
  pin: 
    number: GPIO5
    inverted: true
    mode: INPUT_PULLUP
  dump: all
  id: ir_reciever

remote_transmitter:
  pin: GPIO14
  carrier_duty_percent: 50%

## SENSOR ########################################
binary_sensor:

  # DETECT USB POWER ON TV
    - platform: gpio 
      pin:
        number: GPIO13
        inverted: TRUE
        mode:
          input: true
          pullup: true
#filters:
#          - delayed_on: 150ms
#          - delayed_off: 150ms
      id: TVMB_USB_TV
      on_press:
        if:
          condition:
            - binary_sensor.is_on: TVMB_USB_MB
          then:
            - switch.turn_on: TVMB_run
          else:
            - switch.turn_off: TVMB_relay_DC
            - delay: 100ms
            - switch.turn_on: TVMB_relay_DC
            - delay: 500ms
            - switch.turn_on: TVMB_run

      on_release:
        if:
          condition:
            - binary_sensor.is_on: TVMB_USB_MB
          then:
            - switch.turn_on: TVMB_run
          
  # DETECT USB POWER ON MB (ON / OFF STATUS)
    - platform: gpio
      pin:
        number: GPIO12
        inverted: TRUE
        mode:
          input: true
          pullup: true
#      filters:
#          - delayed_on: 150ms
#          - delayed_off: 150ms
      id: TVMB_USB_MB


          
## IR REMOTE ######################################

  #REMOTE1 - INPUT FOR HA
    #1
    - platform: remote_receiver
      name: TVMB_IR_blue_num1
      pioneer:
        rc_code_1: 0x804E
      filters:
        - delayed_off: 200ms
    #2
    - platform: remote_receiver
      name: TVMB_IR_blue_num2
      pioneer:
        rc_code_1: 0x800D
      filters:
        - delayed_off: 200ms
    #3
    - platform: remote_receiver
      name: TVMB_IR_blue_num3
      pioneer:
        rc_code_1: 0x800C
      filters:
        - delayed_off: 200ms
    #4
    - platform: remote_receiver
      name: TVMB_IR_blue_num4
      pioneer:
        rc_code_1: 0x804A
      filters:
        - delayed_off: 200ms
    #5
    - platform: remote_receiver
      name: TVMB_IR_blue_num5
      pioneer:
        rc_code_1: 0x8009
      filters:
        - delayed_off: 200ms
    #6
    - platform: remote_receiver
      name: TVMB_IR_blue_num6
      pioneer:
        rc_code_1: 0x8008
      filters:
        - delayed_off: 200ms
    #7
    - platform: remote_receiver
      name: TVMB_IR_blue_num7
      pioneer:
        rc_code_1: 0x8046
      filters:
        - delayed_off: 200ms
    #8
    - platform: remote_receiver
      name: TVMB_IR_blue_num8
      pioneer:
        rc_code_1: 0x8005
      filters:
        - delayed_off: 200ms
    #9
    - platform: remote_receiver
      name: TVMB_IR_blue_num9
      pioneer:
        rc_code_1: 0x8004
      filters:
        - delayed_off: 200ms
    #0
    - platform: remote_receiver
      name: TVMB_IR_blue_num0
      pioneer:
        rc_code_1: 0x8001
      filters:
        - delayed_off: 200ms

## SWITCH #########################################
switch:

  # MB pwrbtn
  - platform: gpio 
    id: "TVMB_relay_run"
    pin: GPIO4
    restore_mode: ALWAYS_OFF
    inverted: false

  # MB DCIN    
  - platform: gpio 
    id: "TVMB_relay_DC"
    pin: GPIO15
    restore_mode: ALWAYS_ON
    inverted: false
    
  # run automation    
  - platform: template 
    name: "TVMB_run"
    id: TVMB_run
    turn_on_action:
    - switch.turn_on: TVMB_relay_run
    - delay: 500ms
    - switch.turn_off: TVMB_relay_run

## IR TRANSMITTER #################################

  # IR LGTV POWER TRANS
  - platform: template 
    name: TVMB_IR_TRANS_LGTV_POWER
    optimistic: True
    turn_on_action:
      - remote_transmitter.transmit_lg:
          data: 0x00FB38C7
          nbits: 32
    turn_off_action:
      - remote_transmitter.transmit_lg:
          data: 0x00FB38C7
          nbits: 32
```
