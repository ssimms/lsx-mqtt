# KEF LSX speaker control using MQTT

A script and systemd service allowing a KEF LSX speaker (and possibly the LS50
wireless speaker) to be integrated into a home automation system using the MQTT
protocol.

## Credits

Several people in the Home Assistant community [figured out][lsx-hass] how to send and
receive information from KEF networked speakers, resulting in an add-on specific
to Home Assistant.

[lsx-hass]: https://community.home-assistant.io/t/kef-ls50-wireless/70269

I've reimplemented their findings using the MQTT protocol and the Homie
specification, allowing the speakers to be controlled by any automation system
that supports MQTT devices.

## Prerequisites

- An MQTT server (such as Mosquitto) with SSL enabled, which the script can
  access on port 8883.  A valid SSL certificate is required (Let's Encrypt
  provides them for free).
- Perl (most Linux and Mac systems come with it pre-installed).
- `cpan Net::MQTT::Simple::SSL` to install the Perl MQTT library used by this
  script.

## Installation

1. Copy `lsx-mqtt.default` to `/etc/default/lsx-mqtt`.  Edit it to include the
   details for your MQTT server and LSX speaker.
2. Copy `lsx-mqtt.service` to `/etc/systemd/system/lsx-mqtt@USER.service`,
   replacing USER with your username (both here and in the commands below).
   This script does not need root privileges.
3. Copy `lsx-mqtt` to `/usr/local/bin/lsx-mqtt`.
4. `sudo systemctl daemon-reload`.
5. `sudo systemctl enable lsx-mqtt@USER`.
6. `sudo systemctl start lsx-mqtt@USER`.

## openHAB configuration

Install and configure the MQTT binding (version 2) so that openHAB can
communicate with your MQTT server.

The following examples assume you're configuring openHAB using text files.  If
you're using the Paper UI, some or all of this may be detected automatically.

### openHAB Thing Example

```
Bridge mqtt:broker:mosquitto "MQTT Broker"
[
  host="...",
  port=8883,
  secure=true,
  username="openhab",
  password="...",
  clientID="openhab"
]
{
  Thing topic lsx {
    Channels:
      Type switch : power "Power" [
        stateTopic="homie/lsx/speaker/power",
        commandTopic="homie/lsx/speaker/power/set",
        on="true",
        off="false"
      ]
      Type switch : mute "Mute" [
        stateTopic="homie/lsx/speaker/mute",
        commandTopic="homie/lsx/speaker/mute/set",
        on="true",
        off="false"
      ]
      Type dimmer : volume "Volume" [
        stateTopic="homie/lsx/speaker/volume",
        commandTopic="homie/lsx/speaker/volume/set",
        min=0,
        max=100,
        step=1,
        on="60",
        off="0"
      ]
      Type string : source "Source" [
        stateTopic="homie/lsx/speaker/source",
        commandTopic="homie/lsx/speaker/source/set",
        allowedStates="Aux,Bluetooth,Optical,USB,Wifi"
      ]
  }
}
```

### openHAB Item Example

```
Switch LSX_Speaker
  "Speaker"
  <soundvolume>
  { channel="mqtt:topic:mosquitto:lsx:power" }

Switch LSX_Speaker_Mute
  "Mute"
  <soundvolume_mute>
  { channel="mqtt:topic:mosquitto:lsx:mute" }

Dimmer LSX_Speaker_Volume
  "Volume"
  <soundvolume>
  { channel="mqtt:topic:mosquitto:lsx:volume" }

// Aux, Bluetooth, Optical, USB, Wifi
String LSX_Speaker_Source
  "Source [%s]"
  <soundvolume>
  { channel="mqtt:topic:mosquitto:lsx:source" }
```

## Copyright and License

Copyright 2020 Steve Simms

Licensed under the [Apache License, Version 2.0][license] (the "License");
you may not use this file except in compliance with the License.

[license]: http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
