# Eliana_Lab_ESP32 - AskDuino

An interactive, AI-powered embedded ecosystem hosted on the **ESP32-S3** microcontroller. **AskDuino** bridges local IoT hardware control (RGB LED strip, BLE display) with cloud-based Large Language Models (**OpenAI GPT API**) via real-time WebSockets and an asynchronous web interface.
---

## Key Features

- **Embedded Web Server & WebSockets:** Serves an interactive HTML5/CSS3/JS Web UI directly from the ESP32-S3 over local Wi-Fi, featuring real-time bi-directional WebSocket communication (`/ws`) and REST fallback (`/chat`).
- **Cloud AI Integration:** Connects securely to the **OpenAI API** over TLS (`esp_crt_bundle_attach`) to process natural language queries with conversational memory management (up to 8 rolling context turns).
- **Edge Natural Language Processing (NLP):** Automatically routes and executes hardware commands locally before reaching cloud APIs:
  - **Local RGB Control:** Parses Romanian/English natural language commands (e.g., *"aprinde ledul roșu"*) and Hex codes (e.g., `led #00ffaa`) to drive WS2812 NeoPixel LEDs with smart brightness scaling.
  - **BLE Multi-Device Orchestration:** Communicates with a secondary **ESP32-C3** via Bluetooth Low Energy (BLE GATT) to display custom icons, shapes (triangle, circle, square, star), and emotions (*smile*, *sad*) on a 0.42" OLED display.
- **Fail-Safe Networking:** Built-in Access Point (AP) captive portal fallback for Wi-Fi provisioning if pre-configured credentials fail.
- **Visual Diagnostics:** Multi-color RGB LED state machine showing boot phase, Wi-Fi connectivity, and server status.

---

## System Architecture

```
                       +-------------------------+
                       |    Web Browser / UI     |
                       |  (HTML / CSS / JS App)  |
                       +------------+------------+
                                    |
                          HTTP / WebSockets
                                    v
+-----------------------------------------------------------------------+
|  ESP32-S3 Microcontroller (Main Node)                                 |
|                                                                       |
|  +--------------------+     +-------------------+    +-------------+  |
|  | WS2812 RGB Driver  |     | Command Parser    |    | Local State |  |
|  |  (Status & Colors) |     |  (Edge NLP Logic) |    |  & NVS      |  |
|  +--------------------+     +---------+---------+    +-------------+  |
|                                       |                               |
+---------------------------------------|-------------------------------+
                    |                   |                   |
            Direct  |         BLE GATT  |           HTTPS   |
            Hardware|         Protocol  |        (TLS Sync) |
                    v                   v                   v
      +------------------+    +------------------+    +-------------------+
      | WS2812 LED Strip |    | ESP32-C3 Node    |    |  OpenAI API       |
      | (NeoPixel RGB)   |    | (0.42" OLED UI)  |    |  (GPT Models)     |
      +------------------+    +------------------+    +-------------------+
```

---

## Tech Stack & Engineering Skills

| Category | Technologies & Tools |
| :--- | :--- |
| **Microcontroller Development** | C++, ESP-IDF v5.1.6, FreeRTOS, NVS Flash Storage |
| **Networking & Protocols** | HTTP Server, WebSockets, REST APIs, TLS/HTTPS (`esp_http_client`), Wi-Fi AP/STA |
| **Wireless & IoT Protocols** | Bluetooth Low Energy (BLE / GATT Client & Server) |
| **Hardware Peripherals** | ESP32-S3-WROOM-1, ESP32-C3 SuperMini, WS2812 NeoPixel RGB, 0.42" OLED Display |
| **Frontend & Design** | Modern Responsive UI (HTML5, CSS3 Glassmorphism, Asynchronous JavaScript) |
| **Cloud & Data Processing** | OpenAI Chat Completions API, `cJSON` Parsing, Circular Buffer Memory Management |

---

## Repository Structure

```text
├── main/
│   ├── main.cpp          # System entry point (app_main), NVS, Wi-Fi, HTTP/WS server tasks
│   ├── chat_client.cpp   # OpenAI API client, JSON payload builder, HTTP/TLS handler
│   ├── led_ctrl.cpp      # WS2812 RGB driver, hex/name color parsing, brightness scaling
│   ├── c3_client.cpp     # BLE GATT client logic for ESP32-C3 OLED communication
│   ├── display.c         # Monochromatic OLED drawing utilities and 16x16 icon bitmaps
│   ├── ui_html.cpp       # Embedded HTML/CSS/JS frontend served directly from flash
│   └── idf_component.yml # Managed dependencies (e.g., espressif/led_strip)
├── sdkconfig.defaults    # Pre-configured build flags (16MB Flash, HTTPD, Cert Bundles)
├── source_esp_idf.sh     # IDF environment helper script
└── README.md             # Project documentation
```

---

## Getting Started

### Prerequisites

1. **Hardware:**
   - ESP32-S3 Board (e.g., ESP32-S3-WROOM-1 N16R8)
   - ESP32-C3 Board with 0.42" OLED (optional for BLE display features)
   - WS2812 RGB LED (addressable or fallback GPIO pin)
   - 2x USB Type-C Cables

2. **Software Toolchain:**
   - Visual Studio Code
   - [ESP-IDF v5.1.6 Framework](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/)
   - Valid OpenAI API Key

### Build & Flash Instructions

1. **Set up the ESP-IDF Environment:**
   ```bash
   # Export ESP-IDF tools
   .\esp-idf\export.bat   # Windows
   # or
   source source_esp_idf.sh # Linux/macOS
   ```

2. **Target Setup & Configuration:**
   ```bash
   idf.py set-target esp32s3
   idf.py menuconfig
   ```
   *In `menuconfig`, navigate to `App config` and set:*
   - **Wi-Fi SSID / Password** (optional; enters AP mode if omitted)
   - **OpenAI API Key** & **Model** (e.g., `gpt-4o-mini` or `gpt-3.5-turbo`)
   - **WS2812 LED GPIO Pin**

3. **Build, Flash, and Monitor:**
   ```bash
   idf.py build
   idf.py -p <COM_PORT> flash monitor
   ```

4. **Accessing the Dashboard:**
   - Once connected to Wi-Fi, the serial monitor will output the local IP address:
     `Chat ready: http://192.168.x.x/`
   - Open any browser on the same network and navigate to the IP address to launch **Eliana's Laboratory UI**.

---

## Usage & Command Reference

The system processes commands in Romanian or English. If a command matches local hardware capability, it executes instantly on the device; otherwise, it is delegated to OpenAI GPT.

| Action | Example Command | Output / Behavior |
| :--- | :--- | :--- |
| **Set LED by Name** | `"led albastru"`, `"aprinde ledul rosu"` | Changes RGB LED color locally |
| **Set LED by Hex Code** | `"led #00FFAA"`, `"led #FFC800"` | Parses Hex and updates RGB pixel |
| **Turn Off LED** | `"led stinge"`, `"opreste lumina"` | Clears WS2812 pixel output |
| **Draw Shapes on OLED** | `"patrat"`, `"triunghi"`, `"cerc"`, `"stea"` | Sends BLE GATT packet to C3 to draw shape |
| **Display Emotion** | `"sunt vesel"`, `"trist"` | Displays corresponding emoticon on OLED |
| **General Knowledge** | `"Ce este un microcontroler?"` | Queries OpenAI API and returns conversational response |

---

## Key Technical Highlights

### 1. Memory-Efficient History Management
To run on constrained microcontrollers without exhausting RAM, `chat_client` implements a circular history buffer using `memmove()` to keep only the last **8 context entries**, appending a system prompt (`PROMPT_SISTEM`) on every query.

### 2. Embedded Dynamic Server
The HTTP/WebSocket server in `main.cpp` streams the web interface using C-string variables stored directly in flash memory, eliminating the need for an external SD card or SPIFFS filesystem.

### 3. Graceful Hardware Fallbacks
If the BLE connection to the ESP32-C3 board fails at boot, the main ESP32-S3 application logs a warning (`ESP_LOGW`) and continues normal operation for Wi-Fi, Web UI, RGB LED, and AI processing without crashing.

---
