# LenovoSmart2Google

How to get Home Assistant on the Lenovo Smart Clock 2 - What Cameron doesn't tell you
![image](https://github.com/PhillyGilly/LenovoSmart2Google/assets/56273663/2236ede7-44ed-41d8-9d9c-1258f030f9b4)
Gameron Gray @camerongray1515 has published a brilliant video on Youtube on how to customise the Lenovo Smart Clock. Thank you!
This gives a comprehensive tutorial which is easy to follow right up to the point where he adds the Wallpanel kiosk software at about 29:30.
https://youtu.be/uSHpvbbvz7Q?si=PDhgQF8J-3IWhbml&t=1779
After this he skips over the details of how to make the clock actually display HA and more importantly and how to make the clock dim in your bedroom.

This is my Lenovo on my bedside table and below I will try to explain how I got from where Cameron leaves off ant minute 29 to here:
![2023-09-09 07 30 14](https://github.com/PhillyGilly/LenovoSmart2Google/assets/56273663/f8ce972c-8bd8-4b7c-b9d4-4e2babac02c1)


You need to set up the Wallpanel app to use MQTT and to be discoverable. When HA has found it you will see a new device with properites that will be useful later.
Here is my clock device in HA.
picture---

The Wallpanel app can display any URL so obviously any Home Assistant dashboard but taking into account the dimensions of the display it is probably best to use a "compact" xxxx and I used the Mushroom theme with five aaa across the screen and two rows.

Paul Bottein @piitaya publishes regular videos on Youtube and this one https://youtu.be/gouMnPxYHDc?si=mbHS4UbbNmC-IDrC explains everything you need to know about setting up and using the mushroom theme.

My yaml for this dashboard (not including actions and sub-menus) is:

```
views:
  - title: Bed2
    icon: mdi:bed-double-outline
    badges: []
    cards:
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-entity-card
            entity: sensor.boiler_outside_temperature
            name: Outside
            layout: vertical
            icon_color: light-blue
          - type: custom:mushroom-entity-card
            entity: sensor.bedroom_2_temperature
            name: Inside
            layout: vertical
            icon_color: red
          - type: custom:mushroom-cover-card
            entity: cover.blind_bedroom_2
            layout: vertical
            name: Blind
          - type: custom:mushroom-cover-card
            entity: cover.window_bedroom_2
            layout: vertical
            name: Velux
          - type: custom:mushroom-entity-card
            entity: switch.electric_blanket
            name: Blanket
            layout: vertical
            icon_color: yellow
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-entity-card
            entity: sensor.givtcp_dw2224g041_battery_soc
            layout: vertical
            name: SOC
            icon_color: deep-orange
          - type: custom:mushroom-entity-card
            entity: sensor.smart_meter_electricity_import_today
            name: Import
            layout: vertical
          - type: custom:mushroom-entity-card
            entity: sensor.forecast_today
            layout: vertical
            name: F'cast
          - type: custom:mushroom-entity-card
            entity: sensor.givtcp_ed2248g390_pv_power
            layout: vertical
            icon: mdi:solar-power-variant-outline
            name: PV
          - type: custom:mushroom-fan-card
            entity: fan.vallox
            layout: vertical
            name: MVHR
title: Clock
```

The most important feature of Wallpanel (IMHO) is the ability to automate dimming of the display and here is my automation.
(note I have revised this - update).
```
alias: "57. Dim Bedside Clock Wallpanel "
description: ""
trigger:
  - type: illuminance
    platform: device
    device_id: 7ec34a498e995c51bb776b47b9f9270c
    entity_id: 1cf4bee26bf414d073d4c6bcf2df6ff9
    domain: sensor
    below: 50
    for:
      hours: 0
      minutes: 0
      seconds: 30
    id: night
  - type: illuminance
    platform: device
    device_id: 7ec34a498e995c51bb776b47b9f9270c
    entity_id: 1cf4bee26bf414d073d4c6bcf2df6ff9
    domain: sensor
    above: 50
    below: 500
    for:
      hours: 0
      minutes: 0
      seconds: 30
    id: dull
  - type: illuminance
    platform: device
    device_id: 7ec34a498e995c51bb776b47b9f9270c
    entity_id: 1cf4bee26bf414d073d4c6bcf2df6ff9
    domain: sensor
    above: 500
    for:
      hours: 0
      minutes: 0
      seconds: 30
    id: bright
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - night
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: true
              topic: wallpanel/mywallpanel/command
              payload: "{'brightness':10}"
      - conditions:
          - condition: trigger
            id:
              - dull
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: false
              topic: wallpanel/mywallpanel/command
              payload: "{'brightness':35}"
      - conditions:
          - condition: trigger
            id:
              - bright
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: false
              topic: wallpanel/mywallpanel/command
              payload: "{'brightness':60}"
mode: single
```


When you have everything in Wallpanel/HA running to your satisfation, you can set Wallpanel as the boot App 
