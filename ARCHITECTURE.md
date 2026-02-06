# System Architecture

This document describes how the components work together in the HA Matrix Barcode Scanner system.

## Component Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Physical Components                          │
│                                                                   │
│  ┌──────────────────┐         ┌─────────────────────────────┐   │
│  │  M5Stack Atom    │         │  M5Stack QR/Barcode         │   │
│  │  Matrix          │◄────────┤  Scanner Unit               │   │
│  │                  │ Grove   │                             │   │
│  │  - ESP32         │         │  - Barcode Reader           │   │
│  │  - 5x5 LEDs      │         │  - Red Aiming Light         │   │
│  │  - Button        │         │  - UART Interface           │   │
│  └──────────────────┘         └─────────────────────────────┘   │
│         │                                                         │
│         │ WiFi                                                    │
└─────────┼─────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Home Assistant Server                          │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ESPHome Integration                                      │   │
│  │  - sensor.atom_matrix_barcode_scanned_code               │   │
│  │  - button.atom_matrix_barcode_start_scan_session         │   │
│  │  - light.atom_matrix_barcode_matrix_leds                 │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                        │
│                          ▼                                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Barcode to Shopping List Automation                     │   │
│  │                                                           │   │
│  │  1. Receive barcode from sensor                          │   │
│  │  2. Lookup in product databases ──────────────►          │   │
│  │  3. Fallback to AI if not found  ──────────────►         │   │
│  │  4. Add to shopping list                                 │   │
│  │  5. Announce via TTS                                     │   │
│  │  6. Notify if unknown                                    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                          │                                        │
│         ┌────────────────┼────────────────┐                      │
│         ▼                ▼                ▼                      │
│  ┌──────────┐   ┌──────────────┐   ┌──────────────┐            │
│  │  Todo    │   │  TTS         │   │  Notify      │            │
│  │  List    │   │  Platform    │   │  Service     │            │
│  └──────────┘   └──────────────┘   └──────────────┘            │
└─────────────────────────────────────────────────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     External Services                            │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Open Food    │  │ Open Beauty  │  │ Open Products│          │
│  │ Facts API    │  │ Facts API    │  │ Facts API    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐                             │
│  │ UPC Item DB  │  │ OpenAI       │                             │
│  │ API          │  │ (AI Fallback)│                             │
│  └──────────────┘  └──────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

## Data Flow

### 1. Barcode Scanning Process

```
User Action                    Device Response
───────────                    ───────────────
Press button       ──►         LED matrix lights up
                               Scanner activates (aim + trigger)
                               Session timer starts (30s)

Point at barcode   ──►         Scanner reads barcode
                               
Barcode decoded    ──►         UART sends data to ESP32
                               
ESP32 receives     ──►         Parse & validate barcode
                               Check cooldown timer
                               Check duplicate
                               
Valid scan         ──►         GREEN LED flash
                               Publish to HA sensor
                               Extend session timer
                               Re-arm scanner

Duplicate/fast     ──►         RED LED flash
                               Re-arm scanner
                               Continue session

Timeout (30s)      ──►         LEDs turn off
                               Scanner deactivates
                               Session ends
```

### 2. Product Lookup Cascade

```
Barcode Received
     │
     ├─► Try Open Food Facts
     │        │
     │        ├─► Found? ──► Extract: product_name, generic_name, brands
     │        │                      ──► Continue to Add Item
     │        │
     │        └─► Not Found ▼
     │
     ├─► Try Open Beauty Facts
     │        │
     │        ├─► Found? ──► Extract: product_name, generic_name, brands
     │        │                      ──► Continue to Add Item
     │        │
     │        └─► Not Found ▼
     │
     ├─► Try Open Products Facts
     │        │
     │        ├─► Found? ──► Extract: product_name, generic_name, brands
     │        │                      ──► Continue to Add Item
     │        │
     │        └─► Not Found ▼
     │
     ├─► Try UPC Item DB
     │        │
     │        ├─► Found? ──► Extract: title, brand
     │        │                      ──► Continue to Add Item
     │        │
     │        └─► Not Found ▼
     │
     └─► AI Fallback (OpenAI)
              │
              ├─► Web search for product
              │
              ├─► Format: "type, brand, descriptor"
              │
              └─► Return result (or "unknown item")
```

### 3. Shopping List Integration

```
Product Identified
     │
     ├─► Format Item Name
     │    - Use product_name or generic_name
     │    - Or use AI-formatted name
     │    - Fallback: "Unknown item (barcode)"
     │
     ├─► Add Description
     │    - Include brand if available
     │    - Always include: "EAN: [barcode]"
     │
     ├─► Add to Todo List
     │    - Call todo.add_item service
     │    - Target: todo.shopping entity
     │
     ├─► Speak via TTS
     │    - Message: "[Product] added to shopping list"
     │    - Platform: Piper, Google, etc.
     │
     └─► Send Notification (if unknown)
          - Only if all databases failed
          - Only if AI couldn't identify
          - Message: "Could not identify [barcode]"
```

## Communication Protocols

### ESP Home ↔ Home Assistant

**Protocol:** Home Assistant API (encrypted)

**Data Sent to HA:**
- `sensor.scanned_code` - Text sensor with barcode value
- `button.start_scan_session` - Button press events
- `light.matrix_leds` - LED state

**Data Received from HA:**
- Button press commands
- LED control commands
- OTA firmware updates

### Scanner ↔ ESP32

**Protocol:** UART (115200 baud)

**Commands Sent (on boot):**
```
0x21 0x51 0xC2 0x00 0x02 0x0D 0x0A  → Set CRLF suffix
0x21 0x61 0x41 0x00                 → Trigger mode
0x21 0x61 0x44 0x01                 → Enable sleep
0x21 0x61 0x45 0x23                 → Sleep timeout (35s)
```

**Data Received:**
- Barcode string terminated by `\r\n`
- Printable ASCII characters (0x20-0x7E)

**Control Signals:**
- GPIO23 (TRIG) - Active low, enables scanning
- GPIO33 (AIM) - Active low, enables aiming light

## State Management

### Session States

```
IDLE State
├─ scanner_trigger: OFF
├─ scanner_aim: OFF
├─ session_active: false
└─ LED matrix: OFF

   ▼ (Button press)

ACTIVE State
├─ scanner_trigger: ON
├─ scanner_aim: ON
├─ session_active: true
├─ LED matrix: ON (during scans)
└─ session_last_ms: current time

   ▼ (Successful scan)

SCAN COMPLETE (within session)
├─ Green LED flash (120ms)
├─ Publish barcode to HA
├─ session_last_ms: updated
├─ Trigger OFF (1200ms)
└─ Trigger ON (re-arm)

   ▼ (30 seconds no scans)

TIMEOUT → Return to IDLE

   ▼ (60 seconds of inactivity in IDLE)

DEEP SLEEP State
├─ ESP32 in deep sleep mode
├─ Power consumption: ~0.05W (95% reduction)
├─ All peripherals powered down
└─ Wake on GPIO39 button press

   ▼ (Button press while in deep sleep)

WAKE UP → Boot sequence → Show connection status → Return to IDLE
```

### Deep Sleep Power Management

```
Activity Tracking
├─ last_activity_ms updated on:
│  ├─ Button press (physical or HA)
│  ├─ Successful scan
│  ├─ Connection status showing
│  └─ Session start/end
│
Deep Sleep Conditions
├─ No active scanning session (session_active = false)
├─ Not showing ready status (ready_status_shown = false)
├─ 60 seconds elapsed since last activity
└─ Checked every 5 seconds by interval component
│
Deep Sleep Exit
├─ Button press on GPIO39 triggers hardware wake
├─ ESP32 performs full boot sequence
├─ Initializes scanner and WiFi
└─ Shows red/yellow connection status, then green ready
```

### Cooldown Mechanism

```
Global Cooldown (2000ms default)
├─ Prevents scanning too quickly
├─ Applies to ALL scans
└─ last_accept_ms tracks last accepted scan

Duplicate Detection
├─ Compares with last_code
├─ Allows repeat after 2 seconds
└─ Shows RED flash if duplicate/too quick

Re-arm Delays
├─ Success: 1200ms delay
├─ Blocked/Duplicate: 1500ms delay
└─ Prevents immediate re-trigger
```

## Error Handling

### Scanner Errors
- **No barcode read:** Timeout after trigger, automatic re-arm
- **Invalid data:** Filtered out (non-ASCII), not published
- **UART errors:** Auto-recovery via continuous polling

### Network Errors
- **WiFi disconnect:** ESP32 auto-reconnects
- **HA API offline:** Queues data, sends when reconnected
- **OTA failure:** Falls back to running firmware

### Database Errors
- **API timeout:** Continue to next database (continue_on_error: true)
- **Invalid response:** Treated as not found, continue cascade
- **All databases fail:** Fallback to AI

### AI Fallback Errors
- **OpenAI offline:** Use "Unknown item (barcode)"
- **No web results:** Returns "unknown item"
- **API rate limit:** Graceful fallback to unknown

## Performance Characteristics

### Scan Speed
- Button to ready: <500ms
- Scan to HA update: <1 second
- HA to shopping list: 2-10 seconds (depends on database)
- Total time: 3-11 seconds per item

### Network Usage
- ESP32 → HA: ~100 bytes per scan
- HA → Databases: ~500 bytes per lookup
- AI fallback: ~2KB per request

### Power Consumption
- Deep sleep: ~0.05W (after 60 seconds of inactivity)
- Idle (awake): ~0.8W
- Scanning: ~1.0W
- LED flash: ~1.25W (brief)
- Power savings: 90-95% reduction when in deep sleep mode

### Memory Usage
- ESP32: ~40KB RAM for code + buffers
- Buffer size: 2048 bytes for UART
- String storage: Dynamic allocation

## Security Considerations

### Device Security
- WiFi credentials encrypted in flash
- API encryption via ESPHome native API
- OTA updates password protected
- No exposed network services

### Data Privacy
- Barcodes never stored on device
- Only transmitted to HA when scanned
- HA logs may contain barcode data
- API calls to external services via HA

### Network Security
- WPA2/WPA3 WiFi recommended
- mDNS for local discovery
- No cloud dependencies (except API lookups)
- All traffic encrypted (WiFi, API, HTTPS)

## Extensibility Points

### Hardware
- Add additional buttons/sensors to Grove ports
- Connect to external displays
- Add battery for portable use
- Integrate with other M5Stack modules

### Software
- Add more product databases to cascade
- Implement local database caching
- Add barcode history tracking
- Create mobile app interface
- Add categories/filtering logic
- Integrate with other shopping list apps

---

**See Also:**
- [README.md](README.md) - Full documentation
- [QUICK_START.md](QUICK_START.md) - Setup guide
- [HARDWARE_SETUP.md](HARDWARE_SETUP.md) - Hardware details
