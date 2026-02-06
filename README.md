# HA Matrix Barcode Scanner to Shopping List

A complete Home Assistant integration for scanning barcodes and automatically adding items to your shopping list. This project consists of two main components:

1. **ESP Home firmware** for M5Stack Atom Matrix + Barcode Scanner module
2. **Home Assistant automation** with intelligent product lookup and AI fallback

## 🎯 Features

- **Physical barcode scanner** built with M5Stack Atom Matrix and barcode scanner module
- **Connection status indicator** - Visual feedback on LED matrix:
  - Red/yellow blinking: Waiting for Home Assistant connection
  - Solid green (30s): Connected and ready to scan
  - Green flash: Successful barcode scan
  - Red flash: Duplicate/too quick scan
- **Session-based scanning** with auto-timeout (press button once, scan multiple items)
- **Intelligent product lookup** cascading through multiple databases:
  - Open Food Facts (food products)
  - Open Beauty Facts (beauty/cosmetic products)
  - Open Products Facts (general products)
  - UPC Item DB (alternative database)
- **AI-powered fallback** using OpenAI to identify unknown products
- **Voice feedback** via TTS when items are added
- **Duplicate detection** with configurable cooldown periods
- **Low power consumption** with auto-sleep mode

## 📋 Hardware Requirements

### Required Components
- **M5Stack Atom Matrix** - ESP32-based controller with 5x5 LED matrix
  - https://www.aliexpress.com/item/1005003299281299.html
- **M5Stack QR/Barcode Scanner Unit** - Compatible barcode scanner module
  - https://www.aliexpress.com/item/1005007316111365.html

### Pin Connections
The barcode scanner connects to the Atom Matrix via Grove port:
- GPIO22 - RX (receives data from scanner)
- GPIO19 - TX (sends configuration to scanner)
- GPIO23 - Trigger control (active low)
- GPIO33 - Aim control (active low)
- GPIO39 - Physical button on Atom Matrix

## 🚀 Quick Start

### 1. ESP Home Setup

#### Prerequisites
- ESPHome installed (via Home Assistant add-on or standalone)
- USB-C cable to flash the Atom Matrix
- WiFi credentials

#### Installation Steps

1. **Copy the secrets template:**
   ```bash
   cd esphome
   cp secrets.yaml.example secrets.yaml
   ```

2. **Edit `secrets.yaml` with your information:**
   ```yaml
   wifi_ssid: "YourWiFiNetwork"
   wifi_password: "YourWiFiPassword"
   esphome_ota_password: "YourOTAPassword"
   ```

3. **Flash the device:**
   
   First time (via USB):
   ```bash
   esphome run atom-matrix-barcode.yaml
   ```
   
   Subsequent updates (OTA):
   ```bash
   esphome run atom-matrix-barcode.yaml --device atom-matrix-barcode.local
   ```

4. **Add to Home Assistant:**
   - The device should be auto-discovered
   - Or manually add via Settings → Devices & Services → ESPHome

### 2. Home Assistant Setup

#### Prerequisites
- Home Assistant installed
- OpenAI integration configured (for AI fallback)
- A shopping list (todo entity)
- TTS platform configured (optional, for voice feedback)
- Mobile notification service configured (optional)

#### Installation Steps

1. **Add REST commands** to your `configuration.yaml`:
   ```yaml
   rest_command: !include rest_commands.yaml
   ```
   
   Or copy the content from `home-assistant/rest_commands.yaml` into your configuration.

2. **Import the automation:**
   - Copy `home-assistant/barcode_to_shopping_list.yaml`
   - In Home Assistant: Settings → Automations & Scenes → Import Automation
   - Or add to your automations.yaml file

3. **Customize the automation** (edit these settings as needed):
   - Line 2: Change `sensor.atom_matrix_barcode_scanned_code` if your sensor has a different name
   - Line 216: Change `entity_id: todo.shopping` to match your shopping list entity
   - Line 221: Update `media_player.kitchen_echo` to your TTS device
   - Line 223: Update `tts.piper` to your TTS platform
   - Line 233: Update `notify.notify_ben_mobile` to your notification service

4. **Configure OpenAI (if using AI fallback):**
   - Settings → Devices & Services → Add Integration → OpenAI Conversation
   - Configure with your OpenAI API key
   - The automation expects an agent named `conversation.openai_conversation`

## 📖 Usage

### Device Startup and Connection

When the device powers on or is unplugged and reconnected:

1. **Blinking Red/Yellow** - Device is booting or waiting for Home Assistant connection
   - Red blink (500ms) → Yellow blink (500ms) → repeat
   - Continues until successfully connected to Home Assistant

2. **Solid Green (30 seconds)** - Connected and ready to scan
   - Shows for 30 seconds after connection established
   - Automatically turns off after timeout
   - Press button to start scanning (cancels ready timeout)

3. **Idle (LEDs off)** - Normal standby state
   - Ready to start a scan session when button is pressed

### Starting a Scan Session

**Method 1: Physical Button**
- Press the button on the Atom Matrix device
- LED matrix lights up, indicating session started
- Scanner enters continuous mode

**Method 2: Home Assistant Button**
- Find the "Start Scan Session" button in Home Assistant
- Tap to start a session remotely

### Scanning Items

1. Point the scanner at a barcode
2. Wait for the beep and LED feedback:
   - **Green flash** = Successfully scanned and added
   - **Red flash** = Duplicate or too quick (within cooldown)
3. Continue scanning items - session auto-extends with each successful scan
4. Session ends automatically after 30 seconds of no scans

### What Happens After Scanning

1. **Product Lookup:**
   - Searches Open Food Facts
   - If not found, searches Open Beauty Facts
   - If not found, searches Open Products Facts
   - If not found, searches UPC Item DB

2. **AI Fallback (if all databases fail):**
   - Sends barcode to OpenAI with web search
   - AI identifies product and returns formatted name
   - Format: "type, brand, descriptor" (e.g., "milk, Yeo Valley, semi-skimmed")

3. **Add to Shopping List:**
   - Item added with product name
   - Description includes brand and barcode

4. **Voice Feedback:**
   - TTS announces: "[Product name] added to shopping list"

5. **Notification (if unidentified):**
   - Mobile notification sent if product couldn't be identified
   - Placeholder added as "Unknown item (barcode)"

## ⚙️ Configuration

### Configuration Tuning

Edit `esphome/atom-matrix-barcode.yaml` substitutions:

```yaml
substitutions:
  cooldown_ms: "2000"         # Wait time between scans (ms)
  rearm_success_ms: "1200"    # Re-arm delay after successful scan
  rearm_block_ms: "1500"      # Re-arm delay after duplicate/blocked
  session_timeout_ms: "30000" # Session timeout (30 seconds)
  ready_timeout_ms: "30000"   # Ready indicator timeout (30 seconds)
```

### Scanner Hardware Configuration

The device automatically configures the scanner on boot:
- Sets CRLF line endings (`\r\n`)
- Enables trigger mode (on-demand scanning)
- Enables auto-sleep (35 second timeout)
- Configures control pin states

### Home Assistant Automation Customization

**Changing Product Lookup Priority:**
Edit the `choose` blocks in the automation to reorder or remove databases.

**Adjusting AI Prompt:**
Modify the `conversation.process` action to customize how AI interprets barcodes.

**Changing Shopping List:**
Update line 216: `entity_id: todo.shopping` to your list entity.

## 🔧 Troubleshooting

### ESPHome Device Issues

**Device won't flash:**
- Hold the button while plugging in USB to enter flash mode
- Try different USB cable or port
- Ensure drivers are installed for ESP32

**Scanner not working:**
- Check if LEDs are showing status (red/yellow blinking = not connected)
- Green LED means connected - press button to start scanning
- Check UART connections (GPIO22 RX, GPIO19 TX)
- Review logs in ESPHome: set `logger: level: DEBUG`
- Verify scanner power (check Grove connector)

**WiFi connection fails:**
- Double-check credentials in `secrets.yaml`
- Ensure 2.4GHz WiFi (ESP32 doesn't support 5GHz)
- Check signal strength at device location

### Home Assistant Automation Issues

**Automation not triggering:**
- Verify sensor name matches: `sensor.atom_matrix_barcode_scanned_code`
- Check that sensor exists and updates in Developer Tools → States
- Review automation traces in Settings → Automations

**REST commands failing:**
- Check internet connectivity
- Verify API endpoints are accessible
- Some APIs may have rate limits

**AI fallback not working:**
- Ensure OpenAI integration is configured
- Check agent_id matches: `conversation.openai_conversation`
- Verify OpenAI API key has credits
- Enable web search in OpenAI settings

**Items not added to list:**
- Verify todo entity exists: `todo.shopping`
- Check entity permissions
- Review automation traces for errors

### LED Feedback Issues

**LEDs continuously blinking red/yellow:**
- Device cannot connect to Home Assistant
- Check WiFi credentials in `secrets.yaml`
- Verify Home Assistant API is accessible
- Check ESPHome logs for connection errors

**LEDs too bright:**
Edit brightness in the YAML (default is 70%):
```yaml
brightness: 50%  # Reduce to 50% or lower
```

**LEDs not showing:**
- Check GPIO27 connection
- Verify LED strip is WS2812B compatible
- Ensure correct num_leds: 25 for Atom Matrix

## 📚 File Structure

```
├── esphome/
│   ├── atom-matrix-barcode.yaml    # Main ESP Home configuration
│   └── secrets.yaml.example        # Template for WiFi/passwords
├── home-assistant/
│   ├── barcode_to_shopping_list.yaml   # Main automation
│   └── rest_commands.yaml          # REST API configurations
├── .gitignore                      # Git ignore rules
├── LICENSE                         # MIT License
└── README.md                       # This file
```

## 🔐 Security Notes

- **Never commit `secrets.yaml`** - It contains passwords and API keys
- The `.gitignore` file protects secrets by default
- Use strong passwords for WiFi and OTA updates
- Consider using ESPHome encryption for API communication

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Report bugs via GitHub Issues
- Suggest features or improvements
- Submit pull requests
- Share your setup and customizations

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- M5Stack for the excellent Atom Matrix hardware
- ESPHome team for the powerful ESP32 framework
- Home Assistant community for integration support
- Open Food Facts and related projects for product databases

## 📞 Support

For issues and questions:
- Check the [Troubleshooting](#troubleshooting) section
- Search existing GitHub Issues
- Create a new issue with details about your setup

---

**Happy Scanning! 🛒📱**
