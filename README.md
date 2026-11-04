# G Mouse V1 Maintenance Tool

A static, browser-only maintenance dashboard for G Mouse V1.

## Run
1. Extract the ZIP.
2. Open `index.html` in Chrome or Edge on Windows.
3. Connect the G Mouse V1 and click **Connect USB device**.
4. Grant WebHID access in the browser picker.

No Python, Node.js, local server, backend, or helper application is required. WebHID may require a secure context; if `file://` is blocked by the browser policy, host these static files on HTTPS.

## Tabs
- Overview — device identity and discovered modules.
- Settings — friendly names and descriptions for discovered settings, with readable value previews and save controls.
- Battery — primary/backup live telemetry and decoded state labels.
- Firmware & DFU — firmware metadata and update workflow.
- Activity log — communication and operation messages.

When a G Dongle V1 endpoint is selected, the Battery and Mouse Tester tabs are hidden. They reappear when a mouse endpoint is selected or after disconnecting.

Setting groups and options are shown with friendly names and plain-language descriptions instead of firmware IDs. Values include readable units and previews; scaled wireless-quality values are displayed as decimals while the firmware wire format remains unchanged. Battery snapshots on a mouse endpoint are decoded into a readable summary. Firmware-transfer internals are kept out of Settings because those have dedicated sections. Settings without a known safe format are labeled as advanced data, shown as hexadecimal, and kept read-only.

The dongle's Settings view also hides the BAS battery group, which is not supported on the dongle itself. A connected mouse endpoint continues to show its battery settings.

## HID configuration and battery reports
The page selects the configuration transport from the HID report descriptor. The mouse sends requests with Output Report 7 and receives responses on Input Report 8. The dongle sends requests with Feature Report 6 and reads its responses by polling Feature Report 6; its initial pending response may contain a zeroed recipient/event header. Both transports use the same 128-byte configuration frame (124 data bytes per request). Input Report 8 remains enabled for unsolicited battery updates.

The Battery tab routes unsolicited frames with recipient `0xFF` and event ID `0xFF`, expects the leading battery identifier (1 primary, 2 backup), then decodes the 13- or 14-byte little-endian battery payload. Voltage is shown in mV, current in mA, and temperature in °C (raw signed value divided by 1000). A manual snapshot refresh is attempted through `bas/battery_main` and `bas/battery_backup` when the firmware exposes those options.

## Important
- The mouse firmware must expose WebHID Output Report 7 and Input Report 8; dongle firmware must expose readable/writable Feature Report 6. Configuration frames contain 128 report-data bytes; both devices must run firmware built with this size.
- Physical device testing is still required. Validate connection, read-only configuration and battery telemetry before writing settings or attempting firmware updates.
- The browser DFU implementation does not perform the cryptographic signature verification provided by the Python `imgtool` workflow. Only use trusted firmware packages and do not use browser DFU where signature verification is mandatory.

© 2026 G Mouse V1 Maintenance Tool. All rights reserved.

## Profiles

The **Profiles** tab (G Mouse V1 only) edits the 5 profiles stored on the mouse: report rate, CPI, scroll speed, sensor angle, acceleration, straight-line assist, sleep and battery-saver timers, LED colours and light intensity (10�100 %, scales the firmware LED effects). Values travel as a 222-byte little-endian block with a CRC-16 (layout in `src/util/profile_core.h`), sent in two chunks (`core`, `core_tail`) through the `profile_mgr` configuration module (options `active`, `edit_slot`, `core`, `core_tail`, `valid`, `button_array`). The mouse is the source of truth; backups are JSON files. Button maps with Base, Shift and Shift + Shift layers are edited on the **Button Mapping** tab. The full protocol and user guide is in `docs/WebHub_Configuration_Guide.md`.

A short press of button 4 on the mouse switches to the next profile (round-robin). The RGB LED (PWM, P1.24/P1.22/P1.23, active low) blinks the profile colour once per profile number, scaled by the light intensity. Direct USB/BLE links push the change to this page; behind the dongle the page polls every 2 s while the Profiles tab is open. BLE peer selection by button is disabled; manage peers from Settings (long press still erases the peer).
