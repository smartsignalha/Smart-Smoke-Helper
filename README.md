# Smart-Smoke-Helper
esphome:
  name: smartsmokehelper
  friendly_name: Smart_Smoke_Helper

esp8266:
  board: esp01_1m

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "Secret"



    
# Ota
ota:
  password: !secret ota_password 
# Wifi
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Optional manual IP
  manual_ip:
    static_ip: 888.888.8.888
    gateway: 888.888.8.1
    subnet: 255.255.255.0     

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Makesmokesmart Fallback Hotspot"
    password: !secret ap_password 

captive_portal:


# I2C    

i2c:
  sda: 04
  scl: 05
  scan: true
  id: bus_a
  
# Sensor

sensor:

  - platform: ens160
    eco2:
      name: "eCO2"
    tvoc:
      name: "Total Volatile Organic Compounds"
    aqi:
      id: "air_quality_index"
    update_interval: 60s
    address: 0x53


  - platform: aht10
    variant: AHT20
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 60s    

  - platform: uptime
    name: "Uptime"
    id: uptime_s
    update_interval: 60s  
  
  - platform: wifi_signal 
    name: "WiFi Signal dB"
    id: wifi_signal_db
    update_interval: 60s
    entity_category: "diagnostic"

  - platform: copy 
    source_id: wifi_signal_db
    name: "WiFi Signal Percent"
    filters:
      - lambda: return min(max(2 * (x + 100.0), 0.0), 100.0);
    unit_of_measurement: "Signal %"
    entity_category: "diagnostic"
    device_class: ""
  

# Web Server
web_server:
     port: 80  
     Password: 123

# Time     

time:
  - platform: homeassistant
    id: homeassistant_time


# Button    
button:
  - platform: restart
    name: "Restart"    

# Binary Sensor
binary_sensor:   

  - platform: gpio
    name: "Smoke TESTER"
    pin:
      number: 14
      # One of INPUT or INPUT_PULLUP
      mode:
        input: true
        pullup: true
      inverted: false 
    device_class: smoke
    icon: "mdi:smoke-detector-variant-alert"


  - platform: gpio
    pin:
      number: 16
      mode: 
        input: True
        pulldown: True
    name: "SMOKE DETECTOR"
    filters:
      - delayed_on: 100ms
      - delayed_off: 15s
    id: smokedetector
    device_class: SMOKE
    icon: "mdi:smoke-detector-variant-alert"
output:
  - platform: gpio
    pin: 12
    id: Buzzer
interval:
  - interval: 1s
    then:
      if:
        condition:
           binary_sensor.is_on: smokedetector
        then:
          - output.turn_on: Buzzer
          - delay: 500ms
          -  output.turn_off: Buzzer
        else:
          - output.turn_on: Buzzer

  
 
# Light

light:
  - platform: status_led
    name: "Switch state"
    pin: 
      number: 13
         
# Text_sensor

text_sensor:
  - platform: version
    name: ESPHome Version
  - platform: wifi_info
    ip_address:
      name: IP
    ssid:
      name: SSID         


  - platform: template
    name: "Air Quality Rating"
    lambda: |-
      switch ( (int) (id(air_quality_index).state) ) {
        case 1: return {"EXCELLENT"};
        case 2: return {"GOOD"};
        case 3: return {"MODERATE"};
        case 4: return {"POOR"};
        case 5: return {"UNHEALTHY"};
        default: return {"NOT AVAILABLE"};
      }      



  
          
    
