# SmartIR Media Player

Find your device's brand code [here](MEDIA_PLAYER_CODES.md) and add the number in the `device_code` field. If your device is not supported, you will need to learn your own IR codes and place them in the Json file in `smartir/custom_codes/media_player` subfolder. Please refer to [this guide](CODES_SYNTAX.md) to find a way how to do it. Once you have working device file please do not forgot to submit Pull Request so it could be inherited to this project for other users.

## Configuration variables

| Name                         |  Type   | Default  | Description                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------- | :-----: | :------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                       | string  | optional | The name of the device                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `unique_id`                  | string  | optional | An ID that uniquely identifies this device. If two devices have the same unique ID, Home Assistant will raise an exception.                                                                                                                                                                                                                                                                                                               |
| `device_code`                | number  | required | (Accepts only positive numbers)                                                                                                                                                                                                                                                                                                                                                                                                           |
| `controller_data`            | string  | required | The data required for the controller to function. Look into configuration examples bellow for valid configuration entries for different controllers types.                                                                                                                                                                                                                                                                                |
| `delay`                      | number  | optional | Adjusts the delay in seconds between multiple commands. The default is 0.5                                                                                                                                                                                                                                                                                                                                                                |
| `power_sensor`               | string  | optional | _entity_id_ for a sensor that monitors whether your device is actually `on` or `off`. This may be a power monitor sensor. (Accepts only on/off states)                                                                                                                                                                                                                                                                                    |
| `power_sensor_delay`         |   int   | optional | Maximum delay in second in which power sensor is able to report back to HA changed state of the device, default is 10 seconds. If sensor reaction time is longer extend this time, otherwise you might get unwanted changes in the device state.                                                                                                                                                                                          |
| `power_sensor_restore_state` | boolean | optional | If `true` than in case power sensor will report to HA that device is `on` without HA actually switching it `on `(device was switched on by remote, of device cycled, etc.), than HA will report last assumed state and attributes at the time when the device was `on` managed by HA. If set to `false` when device will be reported as `on` by the power sensors all device attributes will be reported as `UNKNOWN`. Default is `true`. |
| `source_names`               |  dict   | optional | Override the names of sources as displayed in HomeAssistant (see below)                                                                                                                                                                                                                                                                                                                                                                   |

## Example configurations

### Example (using broadlink controller)

Add a Broadlink RM device named "Bedroom" via config flow (read the [docs](https://www.home-assistant.io/integrations/broadlink/)).

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 1000
    controller_data:
      controller_type: Broadlink
      remote_entity: remote.bedroom_remote
      delay_secs: 0.5
      num_repeats: 3
    power_sensor: binary_sensor.tv_power
```

### Example (using xiaomi controller)

```yaml
remote:
  - platform: xiaomi_miio
    host: 192.168.10.10
    token: YOUR_TOKEN

media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 2000
    controller_data:
      controller_type: Xiaomi
      remote_entity: remote.xiaomi_miio_192_168_10_10
    power_sensor: binary_sensor.tv_power
```

### Example (using MQTT controller)

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 3000
    controller_data:
      controller_type: MQTT
      mqtt_topic: home-assistant/living-room-tv/command
    power_sensor: binary_sensor.tv_power
```

### Example (using mqtt Z06/UFO-R11 controller)

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 3000
    controller_data:
      controller_type: UFOR11
      mqtt_topic: home-assistant/living-room-tv/command
    power_sensor: binary_sensor.tv_power
```

### Example (using LOOKin controller)

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 4000
    controller_data:
      controller_type: LOOKin
      remote_host: 192.168.10.10
    power_sensor: binary_sensor.tv_power
```

### Example (using ESPHome)

ESPHome configuration example:

```yaml
esphome:
  name: my_espir
  platform: ESP8266
  board: esp01_1m

api:
  services:
    - service: send_raw_command
      variables:
        command: int[]
      then:
        - remote_transmitter.transmit_raw:
            code: !lambda "return command;"

remote_transmitter:
  pin: GPIO14
  carrier_duty_percent: 50%
```

Additionally to `raw` codes, brand specific encoders can also be used with the corresponding encodings in the json file.

The ESPHome service should be changed to the matching `remote_transmitter.transmit_<brand>`
with the appropriate parameters.

Encoding: `Data` (for JVC, Samsung)
```yaml
api:
  services:
    - service: send_jvc_command
      variables:
        data: int
      then:
        - remote_transmitter.transmit_jvc:
            data: !lambda 'return data;'
```
In the `json` codes' file, commands must be a plain integer or a string in hex.


Encoding: `DataNbits` (for LG, Sony)
```yaml
api:
  services:
    - service: send_lg_command
      variables:
        data: int
        nbits: int
      then:
        - remote_transmitter.transmit_lg:
            data: !lambda 'return data;'
            nbits: !lambda 'return nbits;'
```
In the `json` codes' file, commands must be an object with `data` and optionally `nbits` properties. `data` must be a plain integer or a string in hex.

Encoding: `AddressCommand` (for NEC, RC5, Samsung36, Panasonic, Cambridge Audio).
```yaml
api:
  services:
    - service: send_nec_command
      variables:
        address: int
        command: int
      then:
        - remote_transmitter.transmit_nec:
            address: !lambda 'return data;'
            command: !lambda 'return nbits;'
```
In the `json` codes' file, commands must be an object with `address` and  `command` properties. Both must be plain integers or strings in hex.

HA configuration.yaml:

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 2000
    controller_data:
      controller_type: ESPHome
      esphome_service: my_espir_send_raw_command
    power_sensor: binary_sensor.tv_power
```

### Example (using ZHA controller and a TuYa ZS06)

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 5000
    controller_data:
      controller_type: ZHA
      zha_ieee: "XX:XX:XX:XX:XX:XX:XX:XX"
      zha_endpoint_id: 1
      zha_cluster_id: 57348
      zha_cluster_type: "in"
      zha_command: 2
      zha_command_type: "server"
    power_sensor: binary_sensor.tv_power
```

### Overriding Source Names

Source names in device files are usually set to the name that the media player uses. These often aren't very descriptive, so you can override these names in the configuration file. You can also remove a source by setting its name to `null`.

```yaml
media_player:
  - platform: smartir
    name: Living room TV
    unique_id: living_room_tv
    device_code: 1000
    controller_data:
      controller_type: LOOKin
      remote_host: 192.168.10.10
    source_names:
      HDMI1: DVD Player
      HDMI2: Xbox
      VGA: null
```

### Changing channels

Most IR remotes can only send one key at a time (0 to 9) to change your TV channel, changing to other channels requires pressing 2 consecutive keys. SmartIR handles any channel for you with the standard Home Assistant service interface. Here is an example that changes your Kitchen TV to channel 51:

```yaml
service: media_player.play_media
target:
  entity_id: media_player.kitchen_tv
data:
  media_content_id: 51
  media_content_type: "channel"
```

## Available codes for Media Player devices

[**Media Player codes**](/docs/MEDIA_PLAYER_CODES.md)
