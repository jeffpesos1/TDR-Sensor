# The config files

Two ways to run this. Pick one.

**The short config** pulls the packages from GitHub. Your file is about ten lines, and when the project gets a fix or a new feature you rebuild and it comes down automatically. This is what most people want.

**The full config** is everything in one file with nothing external except the SDI-12 components. Use it if you want to read the whole thing, change the maths, or keep working when GitHub is unreachable. You update it by hand.

Both need a `secrets.yaml` next to them:

```yaml
wifi_ssid: "YourNetwork"
wifi_password: "YourPassword"
```

## Installing ESPHome to build locally

Both approaches below need the `esphome` command. You need Python 3.9 or newer.

```
python -m venv .venv
```

Activate it, then install ESPHome into it:

```
# Linux/macOS
source .venv/bin/activate
pip install esphome

# Windows (PowerShell)
.venv\Scripts\Activate.ps1
pip install esphome
```

A venv keeps ESPHome's dependencies out of your system Python. Do this once per machine; after that just re-activate the venv in new terminal sessions.

**Windows note:** run ESPHome from PowerShell or Command Prompt, not Git Bash/MSYS2. ESP-IDF's build system refuses to build under an MSYS shell (`ERROR: MSys/Mingw is not supported`). The first build downloads the ESP-IDF toolchain (roughly 1-2 GB) into `%LOCALAPPDATA%\esphome`, which takes a few minutes. If the build fails with something like `bits/c++config.h: No such file or directory` or `cannot execute 'as'`, your toolchain cache path is hitting Windows' 260 character path limit; either enable long paths as admin (`Set-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' LongPathsEnabled 1`, then reboot) or set `ESPHOME_ESP_IDF_PREFIX` to something short like `C:\ESPHome\idf` and delete the old cache so it reinstalls there.

## The short config, updates from GitHub

Copy this into the ESPHome dashboard as a new device. Change the board line to match your hardware and the data pin to match your wiring.

```yaml
substitutions:
  name: tdr-sensor
  friendly_name: TDR Sensor
  sdi12_data_pin: GPIO32        # G32 Atom Lite/PoE, G1 AtomS3, G2 Dial
  sdi12_address: "0"
  sample_interval: 10s
  timezone: Pacific/Auckland

packages:
  tdr:
    url: https://github.com/JakeTheRabbit/TDR-Sensor
    ref: main
    refresh: 1d
    files:
      - esphome/packages/boards/atom-lite.yaml    # your board file
      - esphome/packages/tdr_sdi12_core.yaml
      - esphome/packages/tdr_analytics.yaml
      - esphome/packages/wifi_extras.yaml

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap: {}

ota:
  - platform: esphome
```

Swap the board file line for your board:

| Board | Board file | Data pin |
|---|---|---|
| M5Stack Atom Lite | `esphome/packages/boards/atom-lite.yaml` | GPIO32 |
| M5Stack AtomS3 Lite | `esphome/packages/boards/atom-s3.yaml` | GPIO1 |
| M5Stack Atom PoE | `esphome/packages/boards/atom-poe.yaml` | GPIO32 |
| M5Stack Dial | `esphome/packages/boards/m5-dial.yaml` | GPIO2 |
| Generic ESP32 | `esphome/packages/boards/esp32-generic.yaml` | GPIO16 |

The Atom PoE has no WiFi, so drop the `wifi:` block and the `wifi_extras.yaml` line for that one. The generic ESP32 also takes a `board:` substitution, default `esp32dev`.

`refresh: 1d` means it re-checks GitHub for changes once a day when you build. Your substitutions always win over the package defaults, so anything you set in your own file sticks.

### Pin it to a version

`ref: main` follows the latest. If you would rather not move until you choose to, point at a tag instead:

```yaml
    ref: v2.0.0
```

Then bump the tag when you want the update.

### Add MQTT

Add the package and the broker details to `secrets.yaml`:

```yaml
packages:
  tdr:
    url: https://github.com/JakeTheRabbit/TDR-Sensor
    ref: main
    files:
      - esphome/packages/boards/atom-lite.yaml
      - esphome/packages/tdr_sdi12_core.yaml
      - esphome/packages/tdr_analytics.yaml
      - esphome/packages/wifi_extras.yaml
      - esphome/packages/tdr_mqtt.yaml
```

```yaml
# secrets.yaml
mqtt_broker: "192.168.1.10"
mqtt_username: "mqtt"
mqtt_password: "password"
```

### Lock down the Home Assistant connection

By default the API is open on your LAN. To require a key, generate one at [esphome.io/components/api](https://esphome.io/components/api) and add:

```yaml
api:
  encryption:
    key: "your-generated-key-here"
```

Home Assistant asks for it when it adopts the device.

## The full config, self-contained

If you want everything in one file, clone the repo and use the device files directly. They are the same packages, just included from disk instead of GitHub:

```
git clone https://github.com/JakeTheRabbit/TDR-Sensor.git
cd TDR-Sensor/esphome
cp secrets.yaml.example secrets.yaml
```

Edit `secrets.yaml`, then either just build it or build and flash over USB in one step:

```
esphome compile tdr-sensor-atom-lite.yaml   # build only, no board needed
esphome run tdr-sensor-atom-lite.yaml       # build and flash over USB
```

`compile` is what you want if you just want the `.bin` files, for example to flash later with the browser flasher or hand to someone else. It leaves them in `.esphome/build/tdr-sensor/build/`, as `firmware.factory.bin` (full image, for a first USB flash) and `firmware.ota.bin` (delta image, for updating a device already running ESPHome over WiFi).

The files are:

| Board | File |
|---|---|
| M5Stack Atom Lite | [tdr-sensor-atom-lite.yaml](../esphome/tdr-sensor-atom-lite.yaml) |
| M5Stack AtomS3 Lite | [tdr-sensor-atom-s3.yaml](../esphome/tdr-sensor-atom-s3.yaml) |
| M5Stack Atom PoE | [tdr-sensor-atom-poe.yaml](../esphome/tdr-sensor-atom-poe.yaml) |
| M5Stack Dial | [tdr-sensor-m5-dial.yaml](../esphome/tdr-sensor-m5-dial.yaml) |
| Generic ESP32 | [tdr-sensor-esp32-generic.yaml](../esphome/tdr-sensor-esp32-generic.yaml) |

If you want one single file with no `!include` at all, run `esphome config tdr-sensor-atom-lite.yaml`. That prints the fully resolved configuration with every package expanded, which you can save and use as a standalone file.

## One binary, many sensors, many sites

The `esphome/factory/*.yaml` files (the same ones CI builds into the Releases downloads) are built for exactly this: compile once, flash the same `.bin` onto as many boards as you want, then point each one at its own WiFi and MQTT from its own web page, no per-device secrets and no rebuild.

```
esphome compile factory/tdr-sensor-atom-lite-factory.yaml
```

What makes this work:

- **No WiFi baked in.** These ship with `wifi: ap: {}` and no ssid/password, plus `improv_serial:`. On first boot (or whenever it can't join a known network) the device raises its own hotspot. Provision it either with Improv (web.esphome.io, or the ESPHome mobile app, over the USB cable) or by joining the hotspot and using the captive portal's own WiFi page. Either way, ESPHome saves the network to flash itself, nothing to configure in YAML.
- **MQTT the same way.** [tdr_provisioning.yaml](../esphome/packages/tdr_provisioning.yaml) adds MQTT Broker / Username / Password / Port controls and an MQTT Enabled switch to the web page (Network section), all persisted to flash. Type in the broker, flip the switch, done, no reflash. Leave it off if you only want the native Home Assistant API.
- **`name_add_mac_suffix: true`** so multiple devices flashed from the same binary don't collide on the network with the same mDNS name.

Repurposing a board that was already provisioned somewhere else (it remembers its old WiFi network across a normal reflash, that's intentional so OTA updates don't lose your settings)? Wipe it first:

```
esptool --port COM5 erase-flash
```

then flash the factory `.bin` as usual.

### Giving a sensor a fixed local address

The device does not compile in a static IP, by design, since the same binary runs at any site. Two ways to stop its address moving around:

- **mDNS** — `http://<name>.local` always finds it without needing a fixed address at all. With `name_add_mac_suffix: true` that is `tdr-sensor-<macsuffix>.local`, the suffix is the last 3 bytes of the MAC, shown as the device name on its own web page and in the logs.
- **DHCP reservation on your router** — if you want a genuinely fixed IP, bind the device's MAC address to an address in your router's DHCP settings. Every router does this differently, but it is always in the DHCP or LAN client list, usually as "reserve", "static lease" or "always use this address". You need the device's MAC, shown on its web page and in `Settings > About` in Home Assistant if adopted, or in the boot log. This is the recommended way. It needs no firmware support and does not fight with the fleet model, since it is set up once per router, not per device.

## What the packages are

| Package | What is in it |
|---|---|
| [tdr_sdi12_core.yaml](../esphome/packages/tdr_sdi12_core.yaml) | SDI-12 bus, the reading pipeline, the whole calibration suite, web server |
| [tdr_analytics.yaml](../esphome/packages/tdr_analytics.yaml) | Dryback tracking, irrigation detection, steering detection |
| [tdr_mqtt.yaml](../esphome/packages/tdr_mqtt.yaml) | Optional MQTT with discovery, broker/credentials fixed at compile time from secrets.yaml. For one known device on one known network. |
| [tdr_provisioning.yaml](../esphome/packages/tdr_provisioning.yaml) | Optional MQTT with discovery, broker/credentials set at runtime from the web page. For the same binary deployed to many devices/sites. Used by the factory builds. |
| [wifi_extras.yaml](../esphome/packages/wifi_extras.yaml) | Fallback hotspot and WiFi diagnostics |
| [boards/](../esphome/packages/boards) | Per board pins, LED, display, ethernet |

The core and analytics packages carry no board or network config, so they work on any of the boards. Use either `tdr_mqtt.yaml` or `tdr_provisioning.yaml`, never both in the same device file, they both define an `mqtt:` component.

## The SDI-12 components

Both configs pull two external components:

```yaml
external_components:
  - source: github://ssieb/esphome@uarthalf
    components: [ uart ]
  - source: github://ssieb/esphome_components@sdi12
    components: [ sdi12 ]
```

The first is a fork of the ESPHome UART with half-duplex support, which mainline still does not have. The second is the SDI-12 component. Both are needed and both only work on an ESP32 with the esp-idf framework. This is the thing that catches people out, so it is worth repeating: on the Arduino framework or an ESP8266 the build succeeds and the probe never answers.

## Settings you can change without editing YAML

Almost everything worth tuning is a control on the web page and in Home Assistant, and it survives reboots. You do not reflash to calibrate. Substrate profile, VWC gain and offset, field capacity, EC gain and offset, the Hilhorst offset, the pore EC blend window, every analytics threshold and every steering anchor. See [CALIBRATION.md](CALIBRATION.md).

The YAML substitutions are only for things that are fixed at build time: the device name, the data pin, the SDI-12 address, the sample interval and the timezone.
