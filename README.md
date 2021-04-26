# adhan_hassio_dresden
Setting up Hassio to run the Adhan following Dresden times

This project uses the steps outlined [here](https://community.home-assistant.io/t/adhan-automation-using-home-assistant-and-google-home-mini/135622) but modifies some steps to utilize the prayer times calculations used in the city of Dresden, Germany. This method of calculation is not part of the list of calculation methods supported by https://aladhan.com.

Please follow steps 1-7 as they are written in the aforementioned guide.

In step 8, please use the following code instead:

## configuration.yaml

```yaml
# Configure a default setup of Home Assistant (frontend, api, etc)
default_config:

# Text to speech
tts:
  - platform: google_translate

group: !include groups.yaml
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

sensor:
  - platform: time_date
    display_options:
      - 'time'
      - 'date'
      - 'date_time'
      - 'time_date'
      
  - platform: scrape
    name: "Fajr Time"
    resource: "https://namazvakitleri.diyanet.gov.tr/tr-TR/11035"
    select: "#today-pray-times-row > div:nth-child(1) > div > div.tpt-time"
    scan_interval: 10800
    
  - platform: scrape
    name: "Duhr Time"
    resource: "https://namazvakitleri.diyanet.gov.tr/tr-TR/11035"
    select: "#today-pray-times-row > div:nth-child(3) > div > div.tpt-time"
    scan_interval: 10800
    
  - platform: scrape
    name: "Asr Time"
    resource: "https://namazvakitleri.diyanet.gov.tr/tr-TR/11035"
    select: "#today-pray-times-row > div:nth-child(4) > div > div.tpt-time"
    scan_interval: 10800
    
  - platform: scrape
    name: "Maghreb Time"
    resource: "https://namazvakitleri.diyanet.gov.tr/tr-TR/11035"
    select: "#today-pray-times-row > div:nth-child(5) > div > div.tpt-time"
    scan_interval: 10800
    
  - platform: scrape
    name: "Isha Time"
    resource: "https://namazvakitleri.diyanet.gov.tr/tr-TR/11035"
    select: "#today-pray-times-row > div:nth-child(6) > div > div.tpt-time"
    scan_interval: 10800
   
```

Follow steps 9-10 as-is, but replace the automation.yaml file with the following:

## automation.yaml

```yaml
- id: '1517693010922'
  alias: Fajr Adhan
  trigger:
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) == as_timestamp(strptime(states("sensor.fajr_time"), "%H:%M")) }}'
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
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) == as_timestamp(strptime(states("sensor.duhr_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) == as_timestamp(strptime(states("sensor.asr_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) == as_timestamp(strptime(states("sensor.maghreb_time"), "%H:%M")) }}'
  - platform: template
    value_template: '{{ as_timestamp(strptime(states("sensor.time"), "%H:%M")) == as_timestamp(strptime(states("sensor.isha_time"), "%H:%M")) }}'
  condition: []
  action:
  - data:
      entity_id: media_player.everywhere
      volume_level: '0.2'
    service: media_player.volume_set
  - alias: ''
    data:
      entity_id: media_player.everywhere
      media_content_id: https://media.sd.ma/assabile/adhan_3435370/b45e93f1efb3.mp3
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
```

Don't forget to replace _media_player.everywhere_ with your Google Home Mini name.

You can also replace the _media_content_id_ sections with links to alternative **online** calls for prayer. I found this [site](http://www.assabile.com/adhan-call-prayer) to be full of options.

You'll notice that there are two _media_content_ids_. This is because I have split the Fajr Adhan from the other Adhans to have different calls for prayer.

Now click on the floppy drive icon on the top blue bar and then click on the gear at the top right corner and select Restart HASS (2nd from the bottom).

Home Assistant will restart. If everything goes smoothly, your Google Home Mini will raise the call for prayer at the Dresden Prayer Times.
