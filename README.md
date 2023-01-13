# adhan_hassio_dresden
Setting up Hassio to run the Adhan following Dresden times

This project uses the steps outlined [here](https://community.home-assistant.io/t/adhan-automation-using-home-assistant-and-google-home-mini/135622) but modifies some steps to utilize the prayer times calculations used in the city of Dresden, Germany. This method of calculation is not part of the list of calculation methods supported by https://aladhan.com.

Please follow steps 1-7 as they are written in the aforementioned guide.

Then install HACS and Multiscrape as described [here](https://hacs.xyz/docs/setup/prerequisites) and [here](https://github.com/danieldotnl/ha-multiscrape).

In step 8, please use the following code instead:

## configuration.yaml

```yaml
# Loads default set of integrations. Do not remove.
default_config:

# Text to speech
tts:
  - platform: google_translate

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

multiscrape:
  - name: Adhan Scraper
    log_response: true
    resource: "https://www.namaztakvimi.com/almanya/dresden-ezan-vakti.html"
    scan_interval: 10800
    sensor:
      - unique_id: fajr_adhan_time
        name: "Fajr Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(2)"
      - unique_id: shorouk_adhan_time
        name: "Shorouk Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(3)"
      - unique_id: duhr_adhan_time
        name: "Duhr Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(4)"
      - unique_id: asr_adhan_time
        name: "Asr Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(5)"
      - unique_id: maghreb_adhan_time
        name: "Maghreb Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(6)"
      - unique_id: isha_adhan_time
        name: "Isha Time"
        select: "#content > table > tr:nth-child(2) > td:nth-child(7)"


sensor:
  - platform: time_date
    display_options:
      - 'time'
      - 'date'
      - 'date_time'
      - 'time_date'
   
```

Follow steps 9-10 as-is, but replace the automation.yaml file with the following:

## automation.yaml

```yaml
- id: '1517693010922'
  alias: Fajr Adhan
  trigger:
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.fajr_adhan_time"), "%H:%M")) }}'
  condition: []
  action:
  - data:
      entity_id: media_player.everywhere
      volume_level: '0.2'
    service: media_player.volume_set
  - alias: ''
    data:
      entity_id: media_player.everywhere
      media_content_id: https://media.sd.ma/assabile/adhan_3435370/ddb21f7363eb.mp3
      media_content_type: music
    service: media_player.play_media
  - delay: 00:05:00
  - data: {}
    entity_id: media_player.everywhere
    service: media_player.turn_off
  - data:
      entity_id: media_player.everywhere
      volume_level: '0.2'
    service: media_player.volume_set
  mode: restart
- id: '1517693010923'
  alias: Adhan
  trigger:
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.duhr_adhan_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.asr_adhan_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.maghreb_adhan_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.isha_adhan_time"), "%H:%M")) }}'
  condition: []
  action:
  - data:
      volume_level: 0.3
    service: media_player.volume_set
    target:
      device_id:
      - fa97687a6d584283a4fdaed35d5460f1
      - 58d07db9b41647cab164c7fbca183870
      - 08af1f0274fd48d4a6218677d96611cc
      - 3c09753e80be4ecba641855f12fc6a54
  - data:
      entity_id: media_player.everywhere
      media_content_id: https://media.sd.ma/assabile/adhan_3435370/b45e93f1efb3.mp3
      media_content_type: music
    service: media_player.play_media
  - delay:
      hours: 0
      minutes: 4
      seconds: 15
      milliseconds: 0
  - data: {}
    entity_id: media_player.everywhere
    service: media_player.turn_off
  - data:
      volume_level: 0.2
    service: media_player.volume_set
    target:
      entity_id: media_player.everywhere
  mode: restart
- id: '1234321234'
  alias: Adhan Reminder
  trigger:
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M"))  ==
      as_timestamp(strptime(states("sensor.duhr_adhan_time"), "%H:%M"))-900 }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.asr_adhan_time"), "%H:%M"))-900 }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.maghreb_adhan_time"), "%H:%M"))-900 }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) ==
      as_timestamp(strptime(states("sensor.isha_adhan_time"), "%H:%M"))-900 }}'
  condition: []
  action:
  - service: tts.google_translate_say
    data:
      entity_id: media_player.everywhere
      message: Prayer time in 15 minutes
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
```

Don't forget to replace _media_player.everywhere_ with your Google Home Mini name.

You can also replace the _media_content_id_ sections with links to alternative **online** calls for prayer. I found this [site](http://www.assabile.com/adhan-call-prayer) to be full of options.

You'll notice that there are two _media_content_ids_. This is because I have split the Fajr Adhan from the other Adhans to have different calls for prayer. I have also added a reminder 15 minutes before each prayer time.

Now click on the floppy drive icon on the top blue bar and then click on the gear at the top right corner and select Restart HASS (2nd from the bottom).

Home Assistant will restart. If everything goes smoothly, your Google Home Mini will raise the call for prayer at the Dresden Prayer Times.
