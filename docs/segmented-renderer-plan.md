# Seven-segment renderer findings

## Goal

Create a true seven-segment clock face with a deep-black background and vivid
green digits, following the user's supplied vintage-clock reference. The
reference image is intentionally not copied into this repository.

## Why an external renderer

SmartClock-OSS custom faces support text, lines, shapes, rings, bars, weather
icons, and stored JPEG photos. Its large fonts are digit-only but are not
seven-segment glyphs. The custom-face API cannot run animation or fetch a
remote image URL, so dynamic seven-segment digits need to be rasterized outside
the clock.

## Proposed approach

Run a small local service on an Orange Pi Zero 3:

1. Read local time and date using the Orenburg time zone (`YEKT-5`).
2. Optionally read weather and humidity from a local Home Assistant instance.
3. Draw a 240x240 black-and-green seven-segment JPEG.
4. Upload it to the clock as a JPEG through the Home Assistant-compatible photo
   API, reusing one stable filename.

The current device timezone is already `YEKT-5`. The project has not yet
implemented or installed this renderer.

## Clock-side requirements and limitations

- Enable **Home Assistant dashboards** in the clock settings before using the
  compatible photo API.
- With this integration enabled, the compatible API does not require
  authentication and is reachable by other devices on the LAN. Restrict access
  to the Orange Pi or use an isolated trusted network.
- Photo uploads are limited to 200 KB.
- Re-uploading writes flash storage. The upstream documentation recommends
  refresh intervals of 30 seconds or longer; updating once per minute is a
  reasonable starting point.
- Keep replacing a single named JPEG instead of creating a file for every
  update.
- A failed Open-Meteo connection was observed on the clock. The renderer should
  still display time and date when weather data is unavailable.

## Open implementation decisions

- Which host and port the Orange Pi service will use.
- Whether weather comes from Home Assistant or directly from Open-Meteo.
- Which local service manager and deployment method to use.
- Whether to enable the unauthenticated compatible API on the clock.
