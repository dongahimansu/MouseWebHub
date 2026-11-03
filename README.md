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
- Configuration — firmware option discovery and read/write.
- Battery — primary/backup live telemetry and decoded state labels.
- Firmware & DFU — firmware metadata and update workflow.
- LED stream — random and WAV-reactive LED streaming.
- Activity log — communication and operation messages.

## Battery report handling
The Battery tab listens on WebHID Input Report 8 (Usage Page `0xFF00`). It routes unsolicited battery frames with recipient `0xFF` and event ID `0xFF`, expects the leading battery identifier (1 primary, 2 backup), then decodes the 13- or 14-byte little-endian battery payload. Voltage is shown in mV, current in mA, and temperature in °C (raw signed value divided by 1000). A manual snapshot refresh is attempted through `bas/battery_main` and `bas/battery_backup` when the firmware exposes those options.

## Important
- The firmware must expose WebHID reports 7 (Output) and 8 (Input), each with 29 report-data bytes.
- Physical device testing is still required. Validate connection, read-only configuration and battery telemetry before writing settings or attempting firmware updates.
- The browser DFU implementation does not perform the cryptographic signature verification provided by the Python `imgtool` workflow. Only use trusted firmware packages and do not use browser DFU where signature verification is mandatory.

© 2026 G Mouse V1 Maintenance Tool. All rights reserved.
