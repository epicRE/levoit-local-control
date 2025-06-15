# Levoit 200s Fan - Local Control
If you have a Levoit 200s ([https://levoit.com/products/core-200s-smart-air-purifier](https://levoit.com/products/core-200s-smart-air-purifier)) there is a way to control it without any hardware modifications. 

## Configure Home Assistant to work with Levoit
The Levoit makes use of a MQTT (`mqtt/[LEVOITID]/v2/`). All you need to do is setup a MQTT on port 1883 and add TLS support (self-signed is fine). 

Setup a rewrite rule for the DNS values: 
* vdmpmqtt.vesync.com pointing to your [YOUR_MQTT_SERVER] 
* *.vesync.com 127.0.0.1
* Create/update your local DNS server or use something like AdGuard to rewrite DNS entries.
* Setup your DHCP server to serve the IP address of your local DNS server to the Levoit device.
* Setup the device using the official application, but what should happen is that the device will register on to your wifi, get redirected by your DNS server and then connect to your MQTT server.
* Finally you can setup something like Home Assistant and you can control your fan.

### HA configuration example
Edit configuration.yaml and add the following (and replace [LEVOITID] with the ID of your Levoit device (find this using a MQTT client and turn on and off the device)):

```yaml
mqtt:
  button:
    - name: "Fan level 1"
      unique_id: fan_lvl_1_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 1,"type": "wind"},"method": "setLevel","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
    - name: "Fan level 2"
      unique_id: fan_lvl_2_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 2,"type": "wind"},"method": "setLevel","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
    - name: "Fan level 3"
      unique_id: fan_lvl_3_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 3,"type": "wind"},"method": "setLevel","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
      # sleep mode
    - name: "sleepmode"
      unique_id: fan_lvl_sleep_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"mode":"sleep"},"method":"setPurifierMode","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
      #timer
    - name: "timer 30 min"
      unique_id: fan_lvl_timer_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"action":"off","total":1800},"method":"addTimer","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
    - name: "display off"
      unique_id: fan_lvl_disp_off_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"state":false},"method":"setDisplay","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
    - name: "display on"
      unique_id: fan_lvl_disp_on_switch_btn
      command_topic: "mqtt/[LEVOITID]/v2/bypass"
      payload_press: '{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"state":true},"method":"setDisplay","source": "APP"}}'
      qos: 0
      retain: true
      entity_category: "config"
      device_class: "restart"
```


## Breakdown of payloads

Set fan to level 1

```json
{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 1,"type": "wind"},"method": "setLevel","source": "APP"}}

```

Set fan to level 2

```json
{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 2,"type": "wind"},"method": "setLevel","source": "APP"}}

```

Set fan to level 3

```json
{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"id": 0,"level": 3,"type": "wind"},"method": "setLevel","source": "APP"}}

```

Set sleep mode (quite operation)

```json
{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"mode":"sleep"},"method":"setPurifierMode","source": "APP"}}

```

Set timer to 30min (1800 seconds)

```json
{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"action":"off","total":1800},"method":"addTimer","source": "APP"}}
```

Get timer
```json
{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{},"method":"getTimer","source": "APP"}}
```

You should get a `rsp` from `getTimer` at `mqtt/[LEVOITID]/v2/bypass/rsp`
```json
{"traceId":"123456","code":0,"result":{"timers":[{"id":1,"remain":1749,"total":1800,"action":"off"}]}}
```

Set display off

```json
{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"state":false},"method":"setDisplay","source": "APP"}}
```

Set display on

```json
{"traceId": "123456","method": "bypassV2","debugMode":false,"payload":{"data":{"state":true},"method":"setDisplay","source": "APP"}}
```

Get Schedule

```json
{"traceId": "123456","method": "bypassV2","debugMode": false,"payload": {"data": {"maxId": 0},"method": "getSchedule","source": "APP"}}
```


