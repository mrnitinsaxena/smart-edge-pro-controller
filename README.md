<p>
  <a href="https://resonatetoinnovate.com/"><img src="static/images/resonate-to-innovate-logo.svg" alt="Resonate to Innovate" width="190" align="left"></a>
  <a href="https://resonatetoinnovate.com/"><img src="static/images/smart-edge-logo.svg" alt="Smart Edge" width="300" align="right"></a>
</p>
<br clear="both">

<h1 align="center">Pro Controller</h1>

<p align="center">
  <a href="https://mrnitinsaxena.github.io/smart-edge-pro-controller/"><b>Install firmware</b></a> ·
  <a href="https://github.com/mrnitinsaxena/smart-edge-pro-controller/releases">Releases</a>
</p>

ESPHome firmware for the **Smart Edge Pro Controller** by [Resonate to Innovate](https://resonatetoinnovate.com/): an ESP8266-based 4-channel relay controller with 4 wall-switch inputs. Built for [Home Assistant](https://www.home-assistant.io/).

## Features

- **4 relays, 4 switch inputs:** each wall switch controls its relay directly; Home Assistant can switch every relay too
- **Works without Home Assistant:** the wall switches keep working when Home Assistant or Wi-Fi is down
- **Power cuts:** relays come back in their last state
- **Your names and icons:** set a label and icon per channel, and invert a relay if it switches the wrong way
- **Easy setup:** Wi-Fi through the controller's own hotspot (works on any phone); adopt in Home Assistant and the ESPHome dashboard
- **Automatic updates:** a *Firmware update* entity in Home Assistant installs new releases from GitHub
- **Secure by default:** no shared passwords or keys; Home Assistant sets the API encryption key when it adopts the device

## Models

Every release is built for each model. A controller only ever receives updates for its own model.

| Model | Variant | Firmware file |
|---|---|---|
| Pro Controller | P4.S4: 4 relays, 4 switch inputs | `pro_controller/pro-controller.yaml` |

## Repository layout

```
pro_controller/
├── pro-controller.yaml   # one file per model - the firmware the release workflow builds
├── hardware/             # pins per board revision (hw-2025.08.19.yaml)
├── packages/             # shared code: core.yaml, switches/relays and optional features
└── examples/             # all-options.example.yaml - every setting with its default
static/                   # web installer page and logos
.github/workflows/        # checks every model on each push, builds them on each release
```

## Getting started

1. **Install the firmware** (skip if your controller came pre-installed): open the [web installer](https://mrnitinsaxena.github.io/smart-edge-pro-controller/) in Chrome or Edge, connect the controller by USB and click **Install firmware**.
2. **Connect it to Wi-Fi:** on your phone, join the `pro-controller-xxxxxx` Wi-Fi network. The setup page opens automatically (or go to `192.168.4.1`); pick your network and enter the password.
3. **Add it to Home Assistant.** It's discovered automatically under *Settings → Devices & services*. Click **Add**.
4. **Name your channels:** rename *Plug 1–4* and *Switch 1–4* in Home Assistant to what they control.

### Customize the firmware (optional)

In the ESPHome dashboard the device shows up as **Discovered**. Click **Adopt** to take over its configuration, then add any of the options below.

You can also start from scratch:

```yaml
substitutions:
  label_1: "Ceiling light"
  icon_1: "ceiling-light"

packages:
  smart_edge: github://mrnitinsaxena/smart-edge-pro-controller/pro_controller/pro-controller.yaml@v2026.11.25

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

## Configuration options

Set any of these under `substitutions:` in your device config. See [`all-options.example.yaml`](pro_controller/examples/all-options.example.yaml) for all of them.

### Channels (N = 1 to 4)

| Option | Default | Description |
|---|---|---|
| `label_N` | `Switch N` / `Plug N` | Name of switch input N and relay N |
| `icon_N` | `light-switch` / `power-plug` | [Material Design icon](https://pictogrammers.com/library/mdi/) name, without `mdi:` |
| `gpio_invert_N` | `false` | `true` if relay N switches the wrong way |

### Device and network

| Option | Default | Description |
|---|---|---|
| `custom_device_name` | `pro-controller-xxxxxx` | Hostname |
| `custom_friendly_name` | `Smart Edge Pro Controller` | Name shown in Home Assistant |
| `custom_area` | — | Room suggested to Home Assistant when the device is first added (e.g. `Living Room`) |
| `wifi_ssid` / `wifi_password` | — | Hard-code Wi-Fi instead of using the hotspot |
| `api_key` | provisioned by HA | Fix the API encryption key (also encrypts OTA) |
| `web_username` / `web_password` | off | Enable the built-in web page (both required) |
| `wifi_power_mode` | `NONE` | `NONE`, `LIGHT` or `HIGH` power saving |
| `logger_level` | `WARN` | `DEBUG`, `INFO`, `WARN` or `ERROR` |
| `usb_logs` | `false` | Logs over the USB cable (115200 baud) |

## How the switches work

- **Wall switch = relay:** while switch input N is on, relay N is on; turn the switch off and the relay turns off. Use normal (latching) wall switches.
- **Home Assistant** can switch any relay at any time. The next change of the wall switch takes over again.
- **Status LED** (on-board): blinks when Wi-Fi or Home Assistant isn't connected; off when everything is fine.

## Factory reset

- **With a wall switch:** switch the controller's power off and on (e.g. at the breaker). Then, **within 2 minutes**, flick any switch on and off **5 times quickly** (each on and off within half a second). The controller erases its Wi-Fi and API key and restarts in setup mode. After those 2 minutes the pattern is ignored, so it can't happen by accident in daily use.
- **From Home Assistant:** turn on the **Factory reset** switch under the device's configuration entities.

## Hardware

Board revision **2025.08.19**, variant **P4.S4** (shown as *Hardware version* and *Variant* in Home Assistant). MCU: ESP8266 (Wemos D1 mini).

| Function | Pin |
|---|---|
| Relay 1 / 2 / 3 / 4 | D3 / D8 / D5 / D0 |
| Switch input 1 / 2 / 3 / 4 (to GND) | D1 / D2 / D6 / D7 |
| Status LED (on-board) | D4 |

D3 (GPIO0) and D8 (GPIO15) are ESP8266 boot pins: the relay drivers on them must keep D3 high and D8 low while the board powers up.

Each board revision has its own pin file in `pro_controller/hardware/` and its own model files and update channel, so a device only ever receives firmware built for its own board.

## Good to know

- **Time zone:** taken automatically from your Home Assistant.
- **Diagnostics:** uptime and the Home Assistant connection sensor are hidden by default; enable them under the device's entities if you need them.
- **Security:** like most ESPHome devices, the controller accepts firmware updates from your local network. Keep it on a trusted network (or an IoT VLAN), and don't expose it to the internet.

## ⚠️ Safety notice

The controller switches mains-powered loads. Installation must be done by a qualified electrician, following local wiring rules, with the relays rated for the load.

## License

MIT. See [LICENSE](LICENSE).

---

<p align="center">
  Designed and built by<br><br>
  <a href="https://resonatetoinnovate.com/">
    <picture>
      <img src="static/images/resonate-to-innovate-logo.svg" alt="Resonate to Innovate" width="240">
    </picture>
  </a>
  <br>
  <a href="https://resonatetoinnovate.com/">resonatetoinnovate.com</a>
</p>
