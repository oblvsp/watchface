# Project research notes

## Hardware and stock firmware

The clock's settings page identified the device as a Smart Weather Clock. The
stock firmware repository linked from that page is
[JUZIPi-tech/SD_PRO](https://github.com/JUZIPi-tech/SD_PRO).

That repository contains a short README, two prebuilt `.bin` images, and a
manual; it does not contain buildable source code or a GitHub release. The
firmware images contain the string `Firmware ONLY supports ESP8266!!!`. The
manual describes initial setup through the clock's `Smart Weather Clock`
wireless network and a local setup page.

The clock's USB connection did not enumerate as a device or serial port during
inspection. The replacement-firmware documentation reports that the unit's USB
connection may be power-only; UART recovery requires a USB-TTL adapter wired to
the board.

## Replacement firmware

[SmartClock-OSS](https://github.com/michaeldale/smartclock-oss) is an open
replacement for the SD Pro Smart Weather Clock. Its published documentation
lists SD Pro with stock firmware V1.0.6 as tested, and V1.0.4 as untested but
with a matching flash layout. Release 1.3.2 is available from the
[upstream release page](https://github.com/michaeldale/smartclock-oss/releases/tag/v1.3.2).
The official `SDPro_SmartClockOSS_v1.3.2.bin` release image was checked against
SHA-256 `aa3a232418a06bf189d9f7e72493688b4357f27876a1c782d7f827d71d4999c0`.

The firmware was installed through the stock Wi-Fi OTA endpoint and then
verified on the device as version 1.3.2. The firmware status endpoint reported
the SD Pro model, normal operation, and its LAN address. It uses Open-Meteo
for weather and does not require an OpenWeatherMap key.

Important migration constraints documented upstream:

- The replacement firmware does not play animated GIFs.
- Returning to stock firmware over Wi-Fi is unsupported; UART access is needed.
- The update should not be interrupted while the clock is powered.
- The clock's setup access point remains available for recovery.

## Current device configuration

The clock is configured for Orenburg with POSIX timezone rule `YEKT-5`, which
means UTC+05:00 year-round. No network name, password, MAC address, or private
LAN address is recorded here.

At the time of inspection, the device reported a weather-network error from
Open-Meteo. This is an observed transient state, not a confirmed firmware defect.

## Integrations API

The upstream
[Integrations API specification](https://github.com/michaeldale/smartclock-oss/blob/main/docs/specifications/integrations-api.md)
describes three optional integrations:

- **Custom faces:** JSON items include text, shapes, progress rings and bars,
  weather icons, and photos. Faces can use dynamic clock/weather placeholders
  or variables pushed through HTTP or MQTT. The API supports up to 12 faces,
  up to 32 items per face, and a 4 KB JSON limit.
- **Home Assistant dashboards (`geekmagic-hacs`):** uploads a rendered JPEG and
  can control photo rotation, brightness, and face selection. Since the
  integration cannot authenticate, its limited compatible endpoints are
  available to anyone on the network when enabled.
- **MQTT:** Home Assistant discovery and state/control topics for brightness,
  face selection, weather, and custom-face variables.

The firmware's fonts 6 and 8 are large digit fonts, not seven-segment glyphs.
The custom-face JSON API also has no animation or external image URL renderer.
To get true seven-segment digits, render a JPEG outside the clock and upload it
through the photo API.

The upstream documentation limits photo uploads to 200 KB and recommends
refresh intervals of at least 30 seconds to reduce flash writes. The renderer
should reuse one stable filename, rather than filling storage with a new image
on each refresh.

## Face design

The user supplied a reference image of a vintage Soviet electronic clock. The
face design takes inspiration from its black display and bright green digital
readout, adapted to a square 240x240 screen. The reference image itself is not
included in this repository.

The saved face is `assets/retro-orenburg-face.json`, named `retro_orenburg` on
the clock. It contains a dark display, bright mint-green time, date, city,
weather icon, temperature, and humidity. The face is selected on the device.

The user requested removing the red housing color, using a brighter green and
deeper black, and changing the digits to a true seven-segment style. The
firmware's custom-face renderer only offers its built-in text fonts, so true
segment glyphs need to be rendered outside the clock.
