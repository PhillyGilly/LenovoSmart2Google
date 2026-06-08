# How to get Home Assistant on the Lenovo Smart Clock 2 - What Cameron doesn't tell you
![image](https://github.com/PhillyGilly/LenovoSmart2Google/assets/56273663/2236ede7-44ed-41d8-9d9c-1258f030f9b4)

Gameron Gray @camerongray1515 has published a brilliant video on Youtube on how to customise the Lenovo Smart Clock. Thank you!
This gives a comprehensive tutorial which is easy to follow right up to the point where he adds the Wallpanel kiosk software at about 29:30.
https://youtu.be/uSHpvbbvz7Q?si=PDhgQF8J-3IWhbml&t=1779
After this he skips over the details of how to make the clock actually display HA and more importantly and how to make the clock dim in your bedroom.

This is my Lenovo on my bedside table and below I will try to explain how I got from where Cameron leaves off at minute 29 to here:
![2023-09-09 07 30 14](https://github.com/PhillyGilly/LenovoSmart2Google/assets/56273663/f8ce972c-8bd8-4b7c-b9d4-4e2babac02c1)

The first thing is that Wallpanel is quite difficult to find and load. The way in which I got it on to my Levono was to use F-Droid to load an app called "GitHub Store" you can use this download and install the apk for Wallpanel from https://github.com/thetimewalker/wallpanel-android/releases. Once you have got the Wallpanel app loaded on the Levono, you need to configure it as set out in the documentation which is here https://wallpanel.xyz/docs/getting-started also you need to set it up to use MQTT and to be discoverable. I recommend setting the Base Topic as 'Levono/Clock/' to make things easier to see in your mqtt explorer.

****Don't forget that you must set up a unique mqtt user name and password in Home Assistant Mosquito for the Wallpanel App. I used "clock-user" and clock-pass".**** When HA has found it you will see a new device with properites that will be useful later.

Here is "my_wall_panel" device in HA.
<img width="835" height="442" alt="image" src="https://github.com/user-attachments/assets/36741ef4-96fa-4392-b6d4-3b331093a9be" />


The Wallpanel app can display any URL so obviously any Home Assistant dashboard but taking into account the dimensions of the display it is probably best to use a "compact" layout I used the Mushroom theme with two horizontal stack cards each containing five mushroom cards. I also created my clock page as a dashboard from Settings-Dashboard +Add dashboard like so
<img width="462" height="390" alt="image" src="https://github.com/user-attachments/assets/dc4bfdaf-ee54-4c69-bae6-74d11e925731" />


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
            entity: sensor.givtcp_********_battery_soc
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
            entity: sensor.givtcp_ed********_pv_power
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
You need to create an mqtt sensor for Levono Clock Brightness in mqtt.yaml
```
#################################################################
# for Clock                                                     #
#################################################################
sensor:
  - name: Levono Clock Brightness
    unique_id: levonoclockbrightness
    state_topic: "levono/clock/state"
    value_template: "{{value_json.brightness}}"
    icon: "mdi:lightbulb"
```
Then you need to create an automation to match the brightness of the display with the ambient light conditions.
You may need to adjust the thresholds depending on your taste.
```
alias: "57. Dim Bedside Clock Wallpanel "
description: ""
trigger:
  - platform: numeric_state
    entity_id: sensor.my_wall_panel_light
    below: 25
    for:
      hours: 0
      minutes: 0
      seconds: 30
    id: night
  - platform: numeric_state
    entity_id: sensor.my_wall_panel_light
    above: 25
    below: 500
    for:
      hours: 0
      minutes: 0
      seconds: 30
    id: dull
  - platform: numeric_state
    entity_id: sensor.my_wall_panel_light
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
            id: night
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: true
              topic: levono/clock/command
              payload: "{'brightness':10}"
      - conditions:
          - condition: trigger
            id: dull
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: false
              topic: levono/clock/command
              payload: "{'brightness':35}"
      - conditions:
          - condition: trigger
            id: bright
        sequence:
          - service: mqtt.publish
            data:
              qos: "1"
              retain: false
              topic: levono/clock/command
              payload: "{'brightness':60}"
mode: single
```


When you have everything in Wallpanel/HA running to your satisfation, you can set Wallpanel as the boot App 
