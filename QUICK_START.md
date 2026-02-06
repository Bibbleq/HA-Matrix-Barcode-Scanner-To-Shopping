# Quick Start Guide

Get your barcode scanner running in 15 minutes!

## Prerequisites Checklist

- [ ] M5Stack Atom Matrix + Barcode Scanner assembled
- [ ] USB-C cable
- [ ] WiFi network (2.4GHz)
- [ ] Home Assistant installed
- [ ] ESPHome add-on installed in HA

## 5-Minute Hardware Setup

1. **Connect the scanner to Atom Matrix** using Grove cable
2. **Plug in USB-C** to power the device
3. **Verify LED matrix** lights up (even briefly)

✅ Hardware ready!

## 10-Minute Software Setup

### Part 1: Flash ESP Home (5 minutes)

1. **Copy secrets file:**
   ```bash
   cd esphome
   cp secrets.yaml.example secrets.yaml
   ```

2. **Edit secrets.yaml:**
   - Add your WiFi name and password
   - Create an OTA password

3. **Flash the device** (first time requires USB):
   - Open ESPHome dashboard
   - Click "New Device" → "Continue"
   - Select "atom-matrix-barcode.yaml"
   - Click "Install" → "Plug into this computer"
   - Wait for compilation and flash

4. **Add to Home Assistant:**
   - Device auto-discovered
   - Or: Settings → Integrations → ESPHome → Add device

### Part 2: Configure Home Assistant (5 minutes)

1. **Add REST commands** to `configuration.yaml`:
   ```yaml
   rest_command: !include rest_commands.yaml
   ```
   Then copy `home-assistant/rest_commands.yaml` to your HA config folder.

2. **Import automation:**
   - Settings → Automations & Scenes
   - Click "+" → "Create new automation"
   - Click "⋮" (three dots) → "Edit in YAML"
   - Paste contents of `home-assistant/barcode_to_shopping_list.yaml`
   - Save

3. **Update settings** in automation:
   - Change `todo.shopping` to your shopping list entity (line 216)
   - Optional: Update TTS device (line 221, 223)
   - Optional: Update notification service (line 233)

4. **Restart Home Assistant** to load REST commands

## First Scan Test

1. **Press the button** on Atom Matrix
   - LEDs should light up
   - Scanner aiming light turns on

2. **Scan a product barcode:**
   - Point at barcode (5-15cm away)
   - Wait for beep
   - Green LED flash = success!

3. **Check your shopping list:**
   - Open Home Assistant
   - Navigate to your todo list
   - Item should appear with product name

🎉 **Success!** You're now scanning barcodes!

## Common First-Time Issues

### Device won't flash
**Solution:** Hold button while plugging in USB cable

### Can't find device in HA
**Solution:** Check ESPHome logs, verify WiFi credentials

### Barcode doesn't scan
**Solution:** Press button first to start session, then scan

### Item not added to list
**Solution:** Check automation entity IDs match your setup

## Next Steps

- 📖 Read the full [README.md](README.md) for advanced features
- 🔧 Review [HARDWARE_SETUP.md](HARDWARE_SETUP.md) for troubleshooting
- ⚙️ Customize timing in `esphome/atom-matrix-barcode.yaml`
- 🤖 Configure OpenAI for AI fallback on unknown products

## Tips for Best Results

- **Start session first** - Press button before scanning
- **Hold steady** - Keep scanner perpendicular to barcode
- **Optimal distance** - 5-15cm (2-6 inches) from barcode
- **Clean barcodes** - Works best on clean, flat surfaces
- **Good lighting** - Ensure barcode is well-lit

## Support

Issues? Check:
1. ESPHome logs (debug level)
2. HA automation traces
3. GitHub Issues for similar problems

**Enjoy your automated shopping list!** 🛒
