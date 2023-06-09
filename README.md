# homebridge-pentair-screenlogic
[![verified-by-homebridge](https://badgen.net/badge/homebridge/verified/purple)](https://github.com/homebridge/homebridge/wiki/Verified-Plugins)
[![NPM version](https://img.shields.io/npm/v/homebridge-pentair-screenlogic)](https://npmjs.org/package/homebridge-pentair-screenlogic)
![License](https://img.shields.io/npm/l/homebridge-pentair-screenlogic)
[![Downloads](https://img.shields.io/npm/dt/homebridge-pentair-screenlogic.svg)](https://npmjs.org/package/homebridge-pentair-screenlogic)

Fork of [schemers/homebridge-screenlogic](https://github.com/schemers/homebridge-screenlogic) with some of my own improvements and fixes.

## ScreenLogic plug-in for [Homebridge](https://github.com/nfarina/homebridge) using the [node-screenlogic](https://github.com/parnic/node-screenlogic) library.

<img src="https://user-images.githubusercontent.com/249172/86522408-04f3d980-be12-11ea-8fc2-e85904a8b415.png" width="300"> <img src="https://user-images.githubusercontent.com/249172/86522409-0a512400-be12-11ea-8c9f-4165e9fe76a5.png" width="300">

## Requirements

This plugin was recently rewritten in Typescript to use the latest capabilities of Homebridge 1.x, so it requires at a minimum:

1. Homebridge >= 1.0.0
2. Node >= 10.17.0

## Installation

<!-- 2. Clone (or pull) this repository from github into the same path Homebridge lives (usually `/usr/local/lib/node_modules`). Note: the code currently on GitHub is in beta, and is newer than the latest published version of this package on `npm` -->

1. Install homebridge using: `npm install -g homebridge`
2. Install this plug-in using: `npm install -g homebridge-pentair-screenlogic`
3. Update your configuration file. See example `config.json` snippet below.

## Configuration

Configuration samples (edit `~/.homebridge/config.json`):

### UDP Broadcast

This performs a UDP broadcast on 255.255.255.255, port 1444, so ensure your network supports UDP broadcasts and the device is on the same subnet.

```json
"platforms": [
        {
            "platform": "ScreenLogic"
        }
    ],
```

### Direct connection via IP Address

Use this when you know the local static IP address.

```json
"platforms": [
        {
            "platform": "ScreenLogic",
            "ip_address": "192.168.0.100"
        }
    ],
```

- `"port"` is optional and defaults to 80
- `"username"` is optional, but is recommended as it is used to keep accessory UUIDs consistent. Should be in the format `"Pentair: XX-XX-XX"`.

### Remote connection via gateway

Use this to go through Pentair servers.

```json
"platforms": [
        {
            "platform": "ScreenLogic",
            "username": "Pentair: XX-XX-XX",
            "password": "..."
        }
    ],
```

## Optional fields:

- `"hidden_circuits"` comma-separated list of circuit names to hide. Set this for circuits you don't want showing up as switches. (ie.., `"Aux 6,Floor Cleaner"`).

- `"hideAirTemperatureSensor"` hides the air temperature sensor. Default is `false`.

- `"hidePoolTemperatureSensor"` hides the pool temperature sensor, which is redundant if you are showing pool thermostat. Default is `true`.

- `"hideSpaTemperatureSensor"` hides the spa temperature sensor, which is redundant if you are showing spa thermostat. Default is `true`.

- `"hidePoolThermostat"` hides the pool thermostat (aka, pool heater) if you don't want to allow changes via HomeKit. Default is `false`.

- `"hideSpaThermostat"` hides the spa thermostat (aka, spa heater) if you don't want to allow changes via HomeKit. Default is `false`.

- `"statusPollingSeconds"` time in seconds to poll for pool statu. Default is 60 seconds.

- `createLightColorSwitches` will create a "Pool Lights" accessory that contains switches for turing on light modes/colors. Default is `false`.

- `disabledLightColors` an array of strings with the names of light mode/colors you want to _disable_. Default is an empty array.

## Sample config

```json
{
  "platform": "ScreenLogic",
  "hidePoolTemperatureSensor": false,
  "hideSpaTemperatureSensor": true,
  "hideAirTemperatureSensor": true,
  "hidePoolThermostat": false,
  "hideSpaThermostat": false,
  "statusPollingSeconds": 60,
  "createLightColorSwitches": true,
  "disabledLightColors": ["Pool Mode Party", "Pool Mode Romance"],
  "hidden_circuits": "Fountains,Floor Cleaner,Aux 6"
}
```

# Implemented HomeKit Accessory Types

## Air Temperature

- _TemperatureSensor_ accessory (Air) indicating the ambient temperature where thee screenlogic hardware is located

## Pool

- _TemperatureSensor_ accessory (Pool) indicating the ambient temperature of the pool (last known temperature if pool isn't running)

## Spa

- _TemperatureSensor_ accessory (Spa) indicating the ambient temperature of the Spa (last known temperature if pool isn't running)

## Circuits

- creates a _Switch_ accessory for each discovered circuit (i.e., Pool, Spa, Jets, Pool Light, Spa Light, etc)

## Pool Heater

- _Thermostat_ accessory with ambient temperature, mode control (heat/cool/auto/off), and target temperature control. See also [Note on Pool/Spa Heater](#note-on-poolspa-heater) below.

## Spa Heater

- _Thermostat_ accessory with ambient temperature, mode control (heat/cool/auto/off), and target temperature control. See also [Note on Pool/Spa Heater](#note-on-poolspa-heater) below.

## Light Colors

- _Light Colors_ accessory with multiple switches for setting a light mode/color. See also [Note on Light Colors](#note-on-light-colors)

# "On" State

Pool and spa heaters do _not_ turn the pool and/or spa on or off, they just change the heat mode.

i.e., if you want to heat the spa, you need to do two things:

1. turn on the Spa (via the Spa Switch)
2. make sure the Spa Heater is set to something other than off (most likely Heat)

This should work well in practice though, as you will generally have a set target temperature and mode, and then just turn the spa on/off without mucking with the thermostat.

This also means that even if the Pool/Spa is turned off and you open the Pool/Spa Heater it mght say "HEATING TO". It will not actually being heating unless the corresponding Pool/Spa switch is turned on.

# Note on Light Colors

The light color commands (for modes and colors) are exposed as switches in HomeKit if you enable `createLightColorSwitches` (it is `false` by default).

## Semantics

Since I don't have the ability to query the current state of which color/mode is active (Screenlogic app doesn't show it either), I implemented the following behavior:

1. light mode/color switch is turned on (i.e., Pool Mode Sunset)
1. light command is sent to Screenlogic controller
1. after a few seconds, the same switch is turned off to give feedback that the command was sent
1. all pool/spa lights will be turned on (if they aren't already), and set to that mode/color. This is done by the screenlogic controller itself.

## Single Tile

Instead of cluttering the room with a bunch of switches, they are all shown in HomeKit as a single accessory called "Light Colors". Tap on the tile to expand, and then turn on the individual switch for the desired mode/color.

If you'd like to show them as separate tiles, you can tap the gear icon at the bottom of the expanded tile (or slide up at the bottom) and then select "Show as Separate Tiles". If you are showing as separate tiles and want to revert back to a single tile, you can select any switch and then select "Show as Single Tile"

## Renaming

While showing the light switches as separate tiles, you can rename them if desired, which will let you pick a different name to use with Siri. After renaming you can then show as a single tile again if you'd like.

## Siri

You can set a mode/color by saying the mode/color name:

- "Hey Siri, turn on pool mode sunset"
- "Hey Siri, turn on pool color blue"

Which will set the selected mode/color and turn on all the lights if they aren't on.
