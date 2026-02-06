# ESPHome Configuration Files

This directory contains the firmware configuration for the M5Stack Atom Matrix barcode scanner.

## Files

### `atom-matrix-barcode.yaml`
Main ESP Home configuration file for the device. This file contains:
- WiFi and API configuration with connection status indicators
- LED matrix control (5x5 RGB matrix) with visual status feedback
- Barcode scanner UART communication
- Session management and button control
- Scanner configuration commands
- Duplicate detection and cooldown logic
- Connection status indicator (red/yellow blink → solid green)

### `secrets.yaml.example`
Template for storing sensitive credentials. Copy this to `secrets.yaml` and fill in your:
- WiFi SSID and password
- OTA update password

**Important:** Never commit `secrets.yaml` to version control!

## Quick Setup

1. Copy the secrets template:
   ```bash
   cp secrets.yaml.example secrets.yaml
   ```

2. Edit `secrets.yaml` with your credentials:
   ```yaml
   wifi_ssid: "YourNetworkName"
   wifi_password: "YourNetworkPassword"
   esphome_ota_password: "SecurePassword123"
   ```

3. Flash the device:
   ```bash
   esphome run atom-matrix-barcode.yaml
   ```

## Configuration Options

### LED Status Indicators

The device shows visual connection status on the LED matrix:

**Connection Status:**
- **Red/Yellow Blinking** - Waiting for Home Assistant connection
  - Alternates red (500ms) and yellow (500ms)
  - Continues until successfully connected
- **Solid Green (30s)** - Connected and ready to scan
  - Shows after successful connection
  - Auto-turns off after 30 seconds
  - Press button to start scanning (cancels timeout)
- **Green Flash (120ms)** - Successful barcode scan
- **Red Flash (80ms)** - Duplicate or cooldown block

### Timing Adjustments

Edit the `substitutions` section in `atom-matrix-barcode.yaml`:

```yaml
substitutions:
  cooldown_ms: "2000"         # Time between scans (default: 2 seconds)
  rearm_success_ms: "1200"    # Delay after successful scan
  rearm_block_ms: "1500"      # Delay after duplicate/blocked
  session_timeout_ms: "30000" # Session auto-end timeout (30 seconds)
  ready_timeout_ms: "30000"   # Ready status display timeout (30 seconds)
```

### LED Brightness

Adjust LED brightness to protect the acrylic diffuser (default: 70%):

```yaml
- light.turn_on:
    id: matrix_leds
    brightness: 50%  # Change this value (20-100%)
```

### Debug Mode

Enable detailed logging for troubleshooting:

```yaml
logger:
  level: DEBUG  # Change from INFO to DEBUG
```

## Hardware Pin Mapping

| Function | GPIO | Description |
|----------|------|-------------|
| LED Matrix | GPIO27 | WS2812B data line (25 LEDs) |
| Scanner RX | GPIO22 | Receives barcode data from scanner |
| Scanner TX | GPIO19 | Sends configuration to scanner |
| Trigger | GPIO23 | Scanner trigger control (active low) |
| Aim | GPIO33 | Aimer LED control (active low) |
| Button | GPIO39 | Physical button on Atom Matrix |

## Entities Exposed to Home Assistant

After adding the device to Home Assistant, you'll see:

- **Sensor:** `sensor.atom_matrix_barcode_scanned_code` - Last scanned barcode
- **Button:** `button.atom_matrix_barcode_start_scan_session` - Start scanning
- **Light:** `light.atom_matrix_barcode_matrix_leds` - Control LED matrix

## Scanner Configuration Commands

The device automatically configures the scanner module on boot:

1. **Set line endings:** CRLF (`\r\n`) suffix for each scan
2. **Enable trigger mode:** On-demand scanning (saves power)
3. **Enable auto-sleep:** Scanner sleeps after 35 seconds idle
4. **Initialize control pins:** Set trigger and aim to idle state

## Troubleshooting

### LEDs continuously blinking red/yellow
- Device cannot connect to Home Assistant
- Verify `secrets.yaml` credentials are correct
- Ensure 2.4GHz WiFi network (ESP32 doesn't support 5GHz)
- Check WiFi signal strength at device location
- Verify Home Assistant is running and accessible
- Check ESPHome API is enabled in Home Assistant

### Device not connecting to WiFi
- Verify `secrets.yaml` credentials
- Ensure 2.4GHz WiFi network
- Check WiFi signal strength at device location

### Scanner not reading barcodes
- Press button to start session first
- Check UART connections (GPIO22, GPIO19)
- Enable DEBUG logging to see scanner communication
- Verify scanner has power via Grove cable

### LED matrix not working
- Check GPIO27 connection
- Verify WS2812B configuration
- Reduce brightness if LEDs appear dim
- Check power supply provides adequate current

### Excessive duplicate rejections
- Increase `cooldown_ms` value
- Scan different items more slowly
- Check for double-trigger issues

## OTA Updates

After initial flash via USB, updates can be done wirelessly:

```bash
esphome run atom-matrix-barcode.yaml --device atom-matrix-barcode.local
```

Or use the ESPHome dashboard in Home Assistant.

## Learn More

- [ESPHome Documentation](https://esphome.io/)
- [M5Stack Atom Matrix Docs](https://docs.m5stack.com/en/core/atom_matrix)
- [Main README](../README.md)
- [Hardware Setup Guide](../HARDWARE_SETUP.md)
