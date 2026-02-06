# Home Assistant Configuration Files

This directory contains the Home Assistant automation and REST command configurations for the barcode scanner integration.

## Files

### `barcode_to_shopping_list.yaml`
Main automation that processes scanned barcodes. Features:
- Cascading product lookup across multiple databases
- AI-powered fallback for unknown products
- Automatic shopping list management
- TTS voice feedback
- Mobile notifications for unknown items

### `rest_commands.yaml`
REST API endpoint configurations for product databases:
- **Open Food Facts** - Food products database
- **Open Beauty Facts** - Beauty and cosmetic products
- **Open Products Facts** - General products database
- **UPC Item DB** - Alternative barcode lookup service

## Quick Setup

### 1. Add REST Commands

Option A - Include method (recommended):
```yaml
# In configuration.yaml
rest_command: !include rest_commands.yaml
```

Option B - Direct paste:
Copy the contents of `rest_commands.yaml` directly into your `configuration.yaml` under the `rest_command:` section.

### 2. Import the Automation

**Via UI (easiest):**
1. In Home Assistant: Settings → Automations & Scenes
2. Click "+" → "Create new automation"
3. Click "⋮" (three dots) → "Edit in YAML"
4. Copy/paste contents of `barcode_to_shopping_list.yaml`
5. Save with name: "Barcode to Shopping List"

**Via YAML:**
Add to your `automations.yaml`:
```yaml
- !include barcode_to_shopping_list.yaml
```

### 3. Customize for Your Setup

Edit these lines in `barcode_to_shopping_list.yaml`:

**Line 2:** Trigger entity name
```yaml
entity_id: sensor.atom_matrix_barcode_scanned_code  # Update if different
```

**Line 216:** Shopping list entity
```yaml
entity_id: todo.shopping  # Change to your todo list
```

**Line 221:** TTS media player
```yaml
media_player_entity_id: media_player.kitchen_echo  # Your speaker
```

**Line 223:** TTS platform
```yaml
entity_id:
  - tts.piper  # Your TTS service (piper, google_translate, etc.)
```

**Line 233:** Notification service
```yaml
action: notify.notify_ben_mobile  # Your notification service
```

### 4. Restart Home Assistant

Restart is required to load the REST commands.

## How It Works

### Flow Diagram

```
Barcode Scanned
     ↓
Try Open Food Facts
     ↓
   Found? → YES → Extract product info
     ↓ NO
Try Open Beauty Facts
     ↓
   Found? → YES → Extract product info
     ↓ NO
Try Open Products Facts
     ↓
   Found? → YES → Extract product info
     ↓ NO
Try UPC Item DB
     ↓
   Found? → YES → Extract product info
     ↓ NO
Ask OpenAI (AI fallback)
     ↓
Format product name
     ↓
Add to shopping list
     ↓
Speak via TTS
     ↓
Send notification (if unknown)
```

### Product Lookup Priority

1. **Open Food Facts** - Best for food items (especially European products)
2. **Open Beauty Facts** - Beauty, cosmetics, personal care
3. **Open Products Facts** - General household products
4. **UPC Item DB** - Broad database (mainly US products)
5. **OpenAI Conversation** - Web search fallback when all else fails

### AI Fallback

When all databases fail, the automation:
1. Sends barcode to OpenAI with web search enabled
2. AI searches the internet for product information
3. Returns formatted product name: "type, brand, descriptor"
4. Examples: "milk, Yeo Valley, semi-skimmed" or "toothpaste, Oral-B, fresh"

## Required Home Assistant Integrations

### Essential
- ✅ **ESPHome** - For barcode scanner device
- ✅ **Todo List** - For shopping list

### Optional (but recommended)
- 🤖 **OpenAI Conversation** - For AI product identification
- 🔊 **TTS (Text-to-Speech)** - For voice feedback (Piper, Google, Amazon Echo, etc.)
- 📱 **Mobile App / Notify** - For unknown item notifications

## Configuration Examples

### Using Different Shopping List

Replace `todo.shopping` with your list entity:

```yaml
target:
  entity_id: todo.grocery_list  # or todo.costco_list, etc.
```

### Using Multiple TTS Platforms

Speak on multiple devices:

```yaml
target:
  entity_id:
    - tts.piper
    - tts.google_translate_say
```

### Disabling AI Fallback

If you don't want AI lookups, set `enabled: false`:

```yaml
- response_variable: ai_resp
  data:
    agent_id: conversation.openai_conversation
    text: >-
      ...
  action: conversation.process
  enabled: false  # ← Disable this
```

### Custom AI Prompt

Modify the AI instructions to change response format:

```yaml
text: >-
  You are a UK shopping assistant.
  Identify this barcode: {{ scanned_code }}
  
  Return format: "ProductName by Brand"
  Example: "Semi-Skimmed Milk by Yeo Valley"
  
  If unsure, return: "Unknown Product"
```

## Product Database APIs

### Open Food Facts
- **URL:** `https://world.openfoodfacts.org/`
- **Coverage:** Worldwide food products
- **No API key required**
- **Rate limit:** Reasonable for personal use

### Open Beauty Facts
- **URL:** `https://world.openbeautyfacts.org/`
- **Coverage:** Beauty and cosmetic products
- **No API key required**
- **Rate limit:** Reasonable for personal use

### Open Products Facts
- **URL:** `https://world.openproductsfacts.org/`
- **Coverage:** General products
- **No API key required**
- **Rate limit:** Reasonable for personal use

### UPC Item DB
- **URL:** `https://www.upcitemdb.com/`
- **Coverage:** Broad product database
- **Free tier:** 100 requests/day (trial mode)
- **Paid tiers available** for higher limits
- **Register for API key** (optional, improves reliability)

## Troubleshooting

### Automation Not Triggering

Check sensor entity name:
```bash
# In Developer Tools → States, search for:
sensor.atom_matrix_barcode_scanned_code
```

If different, update the `entity_id` in the trigger section.

### REST Commands Not Working

**Verify they're loaded:**
1. Developer Tools → Services
2. Search for: `rest_command.off_lookup_product`
3. If not found, check configuration.yaml includes rest_commands.yaml

**Test manually:**
1. Developer Tools → Services
2. Select: `rest_command.off_lookup_product`
3. Data: `{"barcode": "3017620422003"}`
4. Call service and check response

### Items Not Added to List

**Check todo entity exists:**
1. Settings → Devices & Services → Todo
2. Find your shopping list entity
3. Update line 216 with correct entity_id

### AI Fallback Not Working

**Check OpenAI integration:**
1. Settings → Devices & Services → OpenAI Conversation
2. Verify API key is valid and has credits
3. Ensure agent_id matches: `conversation.openai_conversation`
4. Enable web search in OpenAI settings

### TTS Not Speaking

**Verify TTS platform:**
1. Test TTS in Developer Tools → Services
2. Select your TTS service (e.g., `tts.piper`)
3. Confirm media_player entity is correct
4. Check device volume and network connection

## Automation Traces

Debug using automation traces:
1. Settings → Automations & Scenes
2. Click on "Barcode → AnyList" automation
3. Click "Traces" tab
4. Review last run to see which database succeeded/failed

## Advanced Customization

### Add Custom Product Database

Add another REST command:

```yaml
rest_command:
  custom_api_lookup:
    url: "https://api.example.com/product/{{ barcode }}"
    method: GET
    headers:
      Authorization: "Bearer YOUR_API_KEY"
```

Then add to automation cascade before AI fallback.

### Filter Products by Category

Add condition to only add specific types:

```yaml
- condition: template
  value_template: >-
    {{ 'food' in (product.get('categories', '') | lower) }}
```

### Add to Multiple Lists

Based on product category:

```yaml
- choose:
    - conditions:
        - condition: template
          value_template: "{{ 'beverage' in final_name | lower }}"
      sequence:
        - action: todo.add_item
          target:
            entity_id: todo.drinks_list
  default:
    - action: todo.add_item
      target:
        entity_id: todo.shopping
```

## Learn More

- [Home Assistant Automations](https://www.home-assistant.io/docs/automation/)
- [REST Command Integration](https://www.home-assistant.io/integrations/rest_command/)
- [Todo Integration](https://www.home-assistant.io/integrations/todo/)
- [Main README](../README.md)
