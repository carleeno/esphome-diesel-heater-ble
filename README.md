## ESPHome Integration for Bluetooth Diesel Heater

![alt text](doc/display.png)


## Table of Contents

- [Part One: The fork](#part-one-the-fork)
- [Part Two: External Component Implementation](#part-two-external-component-implementation)
  - [What Has Been Done?](#what-has-been-done)
  - [Key Insights](#key-insights)
  - [Known Protocols](#known-protocols)
    - [IMPORTANT: Current Implementation](#important-current-implementation)
  - [Framework Selection](#framework-selection)
  - [Future Work / TODO](#future-work--todo)

> If you came here only for the juice, go ahead:
[doc/protocols.md](doc/protocols)

This repository contains an ESPHome external component that enables control of Bluetooth‐enabled diesel heaters.



# Part One: The Fork
I have started to maintain the main branch of this fork with my PRs for significant fixes and improvements that I submitted upstream. These are changes which I've found useful as I daily drive my own diesel heater using this component.

However it seems that the upstream fork is no longer maintained, it's been stale for a year with no response to the PRs or Issues, so I have started to merge my improvements here to `main` so it's easier for others to benefit.

# Part Two: External Component Implementation
## What Has Been Done?
- Created the diesel_heater_ble Component: Implements the BLE protocol to communicate with the heater controller board.
- Added Sensors: Supports all settings retrieved from the controller.
- Introduced Controls: Provides a climate entity, switches, buttons, and number inputs for key features, including:
  - Power switch
  - Level-up button
  - Level-down button
  - Temperature-up button
  - Temperature-down button
  - Level set (number input)
  - Temperature set (number input)
  - Climate entity:
    - Full control over the heater using the native climate UI. Temperature setting adjusts auto setpoint, or manually select one of the fan levels for manual control.
- Added support for auto climate control using an external temperature sensor provided by HA or the ESP32
  - Useful for models where the internal temperature control is not accurate, or when the control panel is mounted somewhere different from the area you want to control the temperature of
- Ensured Extensibility: The component is designed to support future versions of controllers.


## Key Insights
One major discovery is that no single protocol exists for communicating with all BLE-enabled controllers. Most existing documentation and repositories focus on one of at least four protocol versions—typically the oldest.

## Known Protocols
Protocols are identified by the first two bytes of the response:

- **0xAA 0x55:**  
  The most basic protocol, featuring a 20-byte response frame.
- **0xAA 0x66:**  
  Very similar to the previous protocol, but with a slightly different byte order in the response.
- **0xAA 0x55 (encrypted):**  
  A newer version that uses a 48-byte encrypted data frame.
- **0xAA 0x66 (encrypted):**  
  Another newer version with a 48-byte encrypted data frame and an altered byte order.

Each protocol employs slightly different formats for requests and responses. For incoming data (requests from the mobile app), the controller expects "plain text," whereas responses are encrypted only in the most recent protocol versions.

### IMPORTANT: Current Implementation
This code parses responses for all protocol versions but currently sends commands only using the **0xAA 0x55 (encrypted)** protocol. I own a heater that uses this third protocol version, so it is fully implemented in this PR. For the other protocols, I welcome assistance from owners of devices using those versions.

## Framework Selection
The BLE stack occupies a significant amount of flash memory. Combined with several sensors, buttons, and other components, this quickly exceeded the 4MB flash capacity of my ESP32 Lolin Lite. I found that the ESP-IDF framework produces a smaller binary compared to the Arduino framework. Therefore, unless you have an 8MB board, ESP-IDF is the only viable option.

## Future Work / TODO

- **Support for Additional Controllers:**  
  Implement `HeaterController_*` classes for other controller types.
  
- **Code Refactoring:**  
  Allow configuration without buttons, switches, or numbers. Currently, if none of these are defined in the YAML, the C++ compiler complains about missing header files (e.g., `esphome/components/button/button.h`).
