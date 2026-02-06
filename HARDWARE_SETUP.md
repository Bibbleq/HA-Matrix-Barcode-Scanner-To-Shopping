# Hardware Setup Guide

This guide provides detailed instructions for assembling and connecting the M5Stack Atom Matrix barcode scanner.

## Required Hardware

### Main Components
1. **M5Stack Atom Matrix** (~$15-20)
   - ESP32-PICO-D4 microcontroller
   - 5x5 RGB LED matrix (WS2812C)
   - Programmable button
   - Grove connector
   - USB-C port

2. **M5Stack QR/Barcode Scanner Unit** (~$25-30)
   - Compatible with EAN-8, EAN-13, UPC-A, UPC-E, Code 39, Code 93, Code 128, QR codes
   - Red LED aiming light
   - UART communication interface
   - Grove connector

3. **Grove Cable** (usually included with scanner unit)
   - 4-pin connector cable
   - Connects scanner to Atom Matrix

### Optional Accessories
- USB-C cable for programming and power
- Case or enclosure for protection
- Wall mount or stand for fixed installation

## Assembly Instructions

### Step 1: Physical Connection

1. **Locate the Grove connector** on the Atom Matrix (on the back/side)
2. **Connect the Grove cable:**
   - One end to the Atom Matrix Grove port
   - Other end to the Barcode Scanner Unit Grove port
3. **Verify connection** is secure and not loose

### Step 2: Power Connection

**Option A: USB Power (Recommended for setup)**
- Connect USB-C cable to Atom Matrix
- Connect other end to USB power adapter or computer
- Power LED should light up

**Option B: Battery Power (Portable use)**
- Use a USB power bank
- Connect via USB-C cable

### Step 3: Verify Hardware

Before flashing firmware:
1. **Check LED Matrix:** Should show default pattern or remain off
2. **Test Button:** Press the button on top of Atom Matrix
3. **Scanner Power:** Scanner should be receiving power through Grove cable

## Pin Mapping Reference

The Atom Matrix automatically maps the Grove connector pins:

| Grove Pin | Atom Matrix GPIO | Scanner Function |
|-----------|------------------|------------------|
| Pin 1     | GPIO 22          | RX (receives barcode data) |
| Pin 2     | GPIO 19          | TX (sends configuration) |
| Pin 3     | GPIO 23          | TRIG (trigger control) |
| Pin 4     | GPIO 33          | AIM (aimer LED control) |

**Additional Pins Used:**
- GPIO 27: LED Matrix data
- GPIO 39: Physical button (internal pull-up)

## Hardware Testing (After Firmware Flash)

### Test 1: Basic Connectivity
1. Flash the ESP Home firmware
2. Check ESPHome logs for successful boot
3. Verify device connects to WiFi
4. Confirm device appears in Home Assistant

### Test 2: LED Matrix
1. Press the physical button on Atom Matrix
2. LED matrix should light up (session started)
3. Scan a barcode
4. Observe LED feedback:
   - Green flash = Success
   - Red flash = Duplicate/too quick

### Test 3: Scanner Functionality
1. Start a scan session (button or HA)
2. Point scanner at a barcode
3. Red aiming light should be visible
4. Scanner beeps on successful read
5. Check Home Assistant sensor for barcode value

### Test 4: Button Control
1. Press button once → Session starts
2. Scanner stays active for 30 seconds
3. Each successful scan extends session
4. Session ends after timeout (LEDs turn off)

## Troubleshooting Hardware

### Scanner Not Reading Barcodes

**Check Distance:**
- Optimal range: 5-15cm (2-6 inches)
- Too close: Image unfocused
- Too far: Insufficient light reflected

**Check Lighting:**
- Ensure barcode is well-lit
- Avoid direct sunlight interference
- Clean barcode surface (no smudges)

**Check Scanner Orientation:**
- Hold scanner perpendicular to barcode
- Avoid extreme angles
- Keep scanner steady

### LED Matrix Not Working

**Check Power:**
- Verify USB-C connection is solid
- Try different power source/cable
- Check power supply provides at least 500mA

**Check Firmware:**
- Reflash ESP Home firmware
- Check GPIO27 pin configuration
- Verify WS2812B chipset setting

**Check Brightness:**
- Default brightness is 70%
- LED protection mode limits brightness
- Edit YAML to adjust if too dim

### Grove Connection Issues

**Check Cable:**
- Ensure Grove cable properly seated
- Try different Grove cable if available
- Check for bent pins

**Check Pins:**
- Verify no dust in connectors
- Gently clean with compressed air
- Check for physical damage

### Button Not Responding

**Check Physical Button:**
- Press firmly but don't force
- Button should "click"
- Check for debris around button

**Check Firmware:**
- GPIO39 configured correctly
- Debounce settings (20ms default)
- Review ESPHome logs for button events

## Mounting and Installation

### Desktop Use
- Place on flat surface
- Angle scanner toward user
- Keep USB cable accessible

### Wall Mount
- Use double-sided tape or mounting bracket
- Position at comfortable scanning height
- Ensure USB power reaches location

### Fixed Installation
- Mount near shopping area/pantry
- Keep away from water sources
- Ensure WiFi signal strength
- Consider cable management

## Power Consumption

**Normal Operation:**
- Idle (no scan): ~100mA @ 5V (0.5W)
- Active scan: ~200mA @ 5V (1.0W)
- LED flash: ~250mA @ 5V (1.25W) brief spike

**Power Saving Features:**
- Scanner auto-sleep after 35 seconds
- Session timeout after 30 seconds
- LEDs off when idle

**Recommended Power Supply:**
- Minimum: 500mA @ 5V (2.5W)
- Recommended: 1A @ 5V (5W)
- USB-C standard power adapter

## Environmental Specifications

**Operating Temperature:**
- Recommended: 15-30°C (59-86°F)
- Range: 0-40°C (32-104°F)

**Humidity:**
- Recommended: 20-80% RH non-condensing
- Avoid direct moisture

**Storage:**
- Temperature: -20 to 60°C (-4 to 140°F)
- Keep in dry location

## Safety Notes

⚠️ **Important Safety Information:**

- Do not expose to water or moisture
- Use only approved 5V USB power supplies
- Do not disassemble or modify hardware
- LED brightness limited to protect acrylic diffuser
- Avoid staring directly at red aiming laser
- Keep away from children under supervision

## Maintenance

**Regular Cleaning:**
- Wipe LED matrix with soft, dry cloth
- Clean scanner window with microfiber cloth
- Remove dust from Grove connectors monthly

**Firmware Updates:**
- Keep ESP Home firmware updated
- OTA updates recommended (no USB needed)
- Backup configuration before updates

**Battery Care (if using portable power):**
- Don't fully discharge power bank
- Recharge regularly
- Replace if degraded

## Next Steps

After hardware setup is complete:

1. ✅ **Hardware assembled** → Proceed to ESP Home firmware flash
2. 📝 **Refer to README.md** → Complete software setup
3. 🔧 **Configure Home Assistant** → Add automation
4. 🛒 **Start scanning** → Build your shopping list!

## Support Resources

- **M5Stack Docs:** https://docs.m5stack.com/en/core/atom_matrix
- **ESPHome:** https://esphome.io/
- **GitHub Issues:** Report problems or ask questions

---

**Questions?** Check the main README.md or create a GitHub issue with:
- Photos of your setup
- ESPHome logs
- Description of the problem
