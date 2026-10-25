# Firmware validation

Normal-firmware validation completed on 2026-10-13 with ESP-IDF 6.0.2 and the locked dependencies.
The four builds passed. The local Connector workload used WS; saved-device reconnection was checked
separately against the backend over WSS.

| Check | T-Circle-S3 | StickS3 | Waveshare AMOLED | ESP32 DevKit |
| --- | --- | --- | --- | --- |
| HTTP interim responses, orderly close and failed reads | Passed | Passed | Passed | Passed |
| Text at the board cap and exact 1 MiB chunk transfer, three rounds | Passed | Passed | Passed | Passed |
| UDP binary echo, three rounds | Passed | Passed | Passed | Passed |
| Queued expiration, permission revocation, disconnect cancellation and recovery | Passed | Passed | Passed | Passed |
| LAN discovery, three rounds | Passed | Passed | Passed | Passed |
| TCP echo, pull and refused-port handling, three rounds | Passed | Passed | Passed | Not supported |
| User-confirmed microphone, playback and hang-up | Passed | Passed | No voice | No voice |

Discovery can return a bounded result with `truncated=true`; a pass does not mean that every host
on the subnet was found. StickS3 playback is set to -3 dB and passed the user's listening check.

Host regressions cover configuration persistence and failed saves, credential-generation races,
request cancellation/deadlines, HTTP parsing, and offline call input. Existing LAN, JSON, audio and
Python suites also passed. See [the development instructions](development.md#host-tests) to run them.

Still unverified on hardware: offline tap/button cancellation and credential replacement across
workspaces. An intermittent 45-second Connector silence observed earlier has not been diagnosed;
successful call checks do not establish a fix for it.

A one-off setup failure that succeeded on retry also has no confirmed cause. Later successful
pairing runs and fixes to other app behavior do not establish that this original failure is fixed.

## Waveshare AMOLED 1.43C

Added after the validation above; not yet part of the table. Checked on one board (ESP32-S3-PICO-1
revision v0.2) with ESP-IDF 6.0.2:

- The five board builds pass; the four existing boards' generated configs differ only by the new,
  unset board choice, and their app images are the same size as on `main`. All C host suites and the
  Python suite pass.
- Boot log after a full flash: 8 MB PSRAM found; ES8311 and ES7210 configured and the I2S port
  running at 24 kHz; ESP-SR models loaded; CST820 touch answers (chip ID 0xB7); CO5300 initialised
  with the framebuffer in PSRAM and the DMA stripe in internal RAM; the first frame is queued and
  Bluetooth setup advertises. The board idled for more than two hours without a reset.
- The setup card ("Hello / Open the Comma app / to set up / Comma-XXXX") is upright with the USB-C
  port at the bottom, centred, not clipped, with no stray column strip at the edges (user photo).

- Bluetooth setup from a Mac with `firmware/tools/ble_provision.py` (no Android phone was
  available): LE Secure Connections pairing with the on-screen passkey, Wi-Fi scan, Wi-Fi up, saved,
  Connector up within 13 s against the production backend; Bluetooth then turned off. Allowing device
  operations reached the board as a `device_access` request.
- Gateway, once each through the backend: LAN discovery (ARP, mDNS, SSDP and the UDP probes) and an
  HTTP GET to a host on the LAN.

Still unverified on hardware: taps, calls in both directions with the mic rate check, mic gain and
speaker level, screen sleep, BOOT holds, and the three-round gateway checks in the table above
(text at the cap, 1 MiB chunks, UDP, tunnels).
The ES7210's MIC3 echo reference is not used; the mic is gated during playback as on the other
boards.
