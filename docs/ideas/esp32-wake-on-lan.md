# Idea: Wake Mac on Telegram Message (ESP32)

## Problem
When the MacBook sleeps, NanoClaw stops polling Telegram. Messages are missed until the Mac wakes.

## Solution
An ESP32 (~$10) plugged into any USB charger on the home network:
- Runs a Telegram bot 24/7 (~1W)
- When a message arrives → sends a WOL magic packet to the Mac → Mac wakes
- NanoClaw picks up and responds

## Why ESP32
| Option | Cost | Power | Notes |
|--------|------|-------|-------|
| Always-on MacBook | $10–25/yr | ~15W | Wasteful |
| ESP32 | ~$10 one-time | ~1W | Purpose-built for this |
| Raspberry Pi Zero W | ~$20 one-time | ~1–2W | More flexible but overkill |

Apple TV can't run custom scripts, so it's not usable as a relay.

## Hardware
- Any ESP32 board, e.g. [ESP32-WROOM DevKit](https://www.amazon.com/s?k=esp32+devkit) ~$8–12
- USB charger + cable (likely already have one)

## Existing Firmware
https://github.com/danilofuchs/esp32-telegram-wake-on-lan — does exactly this.

## Setup Steps (when ready)
1. Buy ESP32 board
2. Install Arduino IDE + ESP32 board support
3. Clone the firmware repo above
4. Configure `config.h` with:
   - WiFi SSID + password
   - Telegram bot token (create a new dedicated bot via @BotFather)
   - Mac's MAC address (`ifconfig en0 | grep ether`)
5. Flash to board, plug into USB charger
6. Enable Wake for Network Access on the Mac:
   `System Settings → Battery → Options → Enable Wake for Network Access`
7. Test: put Mac to sleep, send a message to the ESP32 bot

## Notes
- The ESP32 bot is separate from NanoClaw's bot — it's just a wake trigger
- NanoClaw's Telegram bot handles the actual conversation once the Mac is awake
- May need a static DHCP reservation for the Mac on the router so its IP doesn't change
- WOL only works on the local network — no port forwarding needed
