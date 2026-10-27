# ESP32-C5 Marauder (JSON serial, passive)

> ### ⚖️ Legal / passive-only build — NO attack features
> This is a **receive-only scanner**. Every transmit / attack capability of
> upstream Marauder has been **removed**: no deauth, no beacon spam, no BLE spam,
> no Karma, no Evil Portal, and the firmware's raw-frame injection override is
> gone, so the board **cannot transmit attack frames** even from the console.
> It only *listens* for broadcasts any radio can legally receive. See
> [Legal](#legal) below.

ESP32 Marauder firmware **v1.19.2** built for the **Seeed Studio XIAO ESP32-C5**,
with a small machine-readable **JSON serial interface** (`@J` lines) added so a
host program — such as the companion Android scanner app — can parse device data
reliably instead of scraping the human-readable CLI.

This is a personal fork intended for **passive, receive-only scanning** (WiFi /
BLE sniffers, device lists, AirTag detection, GPS). The firmware's raw-frame
transmit override has been removed, so it does not inject frames.

## What's different from upstream

Based on [justcallmekoko/ESP32Marauder](https://github.com/justcallmekoko/ESP32Marauder)
v1.19.2. The JSON-serial feature is ported from
[0xsys/ESP32Marauder `feature/json-serial-interface`](https://github.com/0xsys/ESP32Marauder/tree/feature/json-serial-interface)
and rebased onto current upstream for the XIAO C5. Changes are tagged `[C5-PORT]`
in the source:

- **`JsonSerial.cpp` / `JsonSerial.h`** — the JSON command set (proto 1):
  `jsoninfo`, `jsonstatus`, `jsonlist <a|s|c|i|p|t>`, `jsonmode`, plus analyzer
  streaming. Every machine line is prefixed `@J `.
- **`configs.h`** — `XIAO_ESP32_C5` target enabled; `MARAUDER_JSON_PROTO 1`; the
  `capstream` capability is gated behind proto ≥ 2 (this build has no binary
  capture path, so it must not advertise it).
- **`CommandLine.cpp`** — runs the JSON dispatch first and suppresses the
  interactive `>` prompt while a JSON host is driving the link.
- **`WiFiScan.cpp`** — removed the `ieee80211_raw_frame_sanity_check` override
  (it only existed to enable raw-frame *transmission*; this is a passive build).
- **`GpsInterface.cpp`** — renamed the GPS UART object to `GpsSerial` to avoid a
  link collision with the core's own `Serial2` on recent ESP32 cores.
- **`WiFiScan.h`** — two header-defined globals made `inline` for GCC 14.

## Build

- **ESP32 Arduino core 3.3.4** (newer cores, e.g. 3.3.12, break the NimBLE link
  on the C5 — see below).
- Arduino IDE: Board → **XIAO_ESP32C5**, **USB CDC On Boot: Enabled**.
- JDK note for CLI builds: Gradle-based tooling needs JDK ≤ 23; use JDK 21 if your
  toolchain ships JDK 25.
- Libraries (pinned to what Marauder's CI uses for the C5): LinkedList 1.3.3,
  ArduinoJson 6.x, **NimBLE-Arduino 2.3.8**, Adafruit_NeoPixel 1.12.0, TFT_eSPI,
  lv_arduino 3.0.0, JPEGDecoder 1.8.0, SwitchLib, ESP32Ping 1.6, AsyncTCP 3.4.8,
  MicroNMEA 2.0.6, ESPAsyncWebServer 3.8.1, EspSoftwareSerial 8.1.0.

## Verify after flashing

Open a serial monitor at 115200 and type `jsoninfo`. Expect one line like:

```
@J {"t":"info","fw":"v1.19.2","proto":1,"board":"Seeed Studio XIAO ESP32-C5","caps":["wifi","bt","gps"]}
```

Then `scanall` starts a dual-band scan and `jsonlist a` streams the access
points as `@J {"t":"ap",...}` lines.

## Credits & license

- ESP32 Marauder firmware by [justcallmekoko](https://github.com/justcallmekoko/ESP32Marauder)
- JSON serial interface by [0xsys](https://github.com/0xsys)

MIT licensed — see [LICENSE](LICENSE). Original copyright retained.

## Legal

A passive scanner receives broadcasts any radio can hear. Transmitting /
interfering with networks you don't own (deauth, spoofing, BLE spam) is illegal
in the US (47 U.S.C. §333) and is intentionally not part of this build. Use it on
your own equipment and within your local laws.
