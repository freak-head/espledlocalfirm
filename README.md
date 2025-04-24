**LED Controller API Documentation**

**Version:** Based on firmware code `v1.0.0` (or as defined by `CURRENT_FIRMWARE_VERSION`)

**Base URL:** `http://<ESP_IP>/`
*   Replace `<ESP_IP>` with the actual IP address assigned to your ESP8266 on your local network (found via Serial Monitor on boot or network scanner).
*   If the device is in Access Point (AP) mode, the Base URL is `http://192.168.4.1/`.

**Authentication:**

*   Most control endpoints require authentication.
*   Authentication is performed by providing the API password (set by `API_PASSWORD` in the firmware code) as a query parameter named `password`.
*   **Example:** `http://<ESP_IP>/reboot?password=YOUR_STRONG_PASSWORD`
*   Endpoints that do **not** require authentication are explicitly marked.
*   Failure to provide the correct password will result in a `401 Unauthorized` error.

**Response Format:**

*   Successful API calls (excluding `/`) typically return `200 OK` with a JSON body representing the current status of the device (similar to the `/status` endpoint).
*   Error responses use standard HTTP status codes (4xx, 5xx) and return a JSON body with an `error` and optional `message` field. Example: `{"error": "Unauthorized", "message": "Missing or incorrect password parameter"}`
*   The root `/` endpoint returns `text/html`.

---

**Endpoints**

**1. Get HTML Control Page**

*   **Endpoint:** `GET /`
*   **Description:** Returns a basic HTML page. In STA mode, it provides links and status info. In AP mode, it displays the WiFi configuration form.
*   **Authentication:** Not Required.
*   **Parameters:** None.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `text/html`
    *   Body: HTML content of the page.
*   **Example Usage:** Open `http://<ESP_IP>/` in a web browser.

**2. Get Device Status**

*   **Endpoint:** `GET /status`
*   **Description:** Retrieves the current detailed status of the device, including network info, LED states, PIR status, and firmware update availability.
*   **Authentication:** Not Required.
*   **Parameters:** None.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: See **Status Object Details** section below.
*   **Example Usage:** `http://<ESP_IP>/status`

**3. Control LEDs**

*   **Endpoint:** `GET /led`
*   **Description:** Sets the state (on/off) or brightness for one or more LEDs. **Using this endpoint will disable any active effect** (sets effect mode to `none`).
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
    *   *Target Selection (Choose ONE):*
        *   `strip` (integer, required): The 1-based index of the single strip to control (e.g., `1`, `2`, `3`, `4`).
        *   `strips` (string, required): A comma-separated list of 1-based strip indices (e.g., `1,3`, `2,4`).
        *   `all` (boolean, required): Set to `true` to target all strips.
    *   *Action (Choose ONE):*
        *   `state` (string, required): Set to `on` or `off`. `on` sets the LED(s) to the current `master_brightness`. `off` sets brightness to 0.
        *   `brightness` (integer, required): Set brightness directly (0-1023). A value > 0 implies `on`.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: Current device status JSON object.
*   **Error Responses:**
    *   `400 Bad Request`: Missing required parameters (target or action), invalid parameter values.
    *   `401 Unauthorized`: Missing or incorrect password.
*   **Example Usage:**
    *   Turn strip 1 ON: `http://<ESP_IP>/led?strip=1&state=on&password=YOUR_PASSWORD`
    *   Turn strip 2 OFF: `http://<ESP_IP>/led?strip=2&state=off&password=YOUR_PASSWORD`
    *   Set strip 3 brightness: `http://<ESP_IP>/led?strip=3&brightness=500&password=YOUR_PASSWORD`
    *   Set strips 1 and 4 brightness: `http://<ESP_IP>/led?strips=1,4&brightness=850&password=YOUR_PASSWORD`
    *   Turn all strips OFF: `http://<ESP_IP>/led?all=true&state=off&password=YOUR_PASSWORD`

**4. Control Effects & Master Brightness**

*   **Endpoint:** `GET /effect`
*   **Description:** Sets the active lighting effect or adjusts the global master brightness level.
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
    *   *Action (Provide at least ONE):*
        *   `mode` (string, optional): The name of the effect to activate. Use `none` to disable effects and revert to static LED brightness. See **Effect Names** section below.
        *   `master_brightness` (integer, optional): Sets the global brightness ceiling (0-1023). Affects `state=on` commands and scales effect brightness.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: Current device status JSON object.
*   **Error Responses:**
    *   `400 Bad Request`: Missing `mode` or `master_brightness`, or invalid `mode` name.
    *   `401 Unauthorized`: Missing or incorrect password.
*   **Example Usage:**
    *   Activate Breathe effect: `http://<ESP_IP>/effect?mode=breathe&password=YOUR_PASSWORD`
    *   Set master brightness: `http://<ESP_IP>/effect?master_brightness=750&password=YOUR_PASSWORD`
    *   Stop effects: `http://<ESP_IP>/effect?mode=none&password=YOUR_PASSWORD`

**5. Control PIR Sensor**

*   **Endpoint:** `GET /pir`
*   **Description:** Enables or disables the PIR motion sensor functionality.
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
    *   `enabled` (boolean, required): Set to `true` to enable the sensor, `false` to disable it.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: Current device status JSON object.
*   **Error Responses:**
    *   `400 Bad Request`: Missing `enabled` parameter or invalid value.
    *   `401 Unauthorized`: Missing or incorrect password.
*   **Example Usage:**
    *   Enable PIR: `http://<ESP_IP>/pir?enabled=true&password=YOUR_PASSWORD`
    *   Disable PIR: `http://<ESP_IP>/pir?enabled=false&password=YOUR_PASSWORD`

**6. Save WiFi Credentials**

*   **Endpoint:** `POST /wifi`
*   **Description:** Saves new WiFi credentials (SSID and Password) to the device's persistent storage (EEPROM). The device will then reboot and attempt to connect using these new credentials. This endpoint is primarily used by the HTML form in AP mode but can be called programmatically.
*   **Authentication:** **Required** (within the form data/body).
*   **Parameters (Form Data):**
    *   `ssid` (string, required): The WiFi network name (SSID).
    *   `pass` (string, optional): The WiFi password. Can be empty for open networks.
    *   `password` (string, required): Your **API password** (for authentication), *not* the WiFi password.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `text/html`
    *   Body: HTML confirmation page indicating reboot.
*   **Error Responses:**
    *   `400 Bad Request`: Missing `ssid` parameter.
    *   `401 Unauthorized`: Missing or incorrect API `password` in form data.
*   **Example Usage (using curl):**
    `curl -X POST -d "ssid=MyHomeWiFi&pass=MyWiFiPassword&password=YOUR_API_PASSWORD" http://<ESP_IP>/wifi`

**7. Enter WiFi Setup Mode**

*   **Endpoint:** `GET /wifi_setup`
*   **Description:** Instructs the device to clear its currently saved WiFi credentials from EEPROM and reboot into Access Point (AP) mode. This allows reconfiguration if the device can no longer connect to the saved network.
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `text/html`
    *   Body: HTML confirmation page indicating reboot into AP mode.
*   **Error Responses:**
    *   `401 Unauthorized`: Missing or incorrect password.
*   **Example Usage:** `http://<ESP_IP>/wifi_setup?password=YOUR_PASSWORD`

**8. Check for Firmware Updates**

*   **Endpoint:** `GET /update/check`
*   **Description:** Forces the device to query the configured GitHub repository for the latest release and check if its version tag is newer than the currently running firmware. Updates the `update_available` and `latest_version` fields in the status object.
*   **Authentication:** Not Required.
*   **Parameters:** None.
*   **Success Response:**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: Current device status JSON object (reflecting the results of the check).
*   **Error Responses:**
    *   `503 Service Unavailable`: Returned if an update is already in progress.
*   **Example Usage:** `http://<ESP_IP>/update/check`

**9. Trigger Firmware Update**

*   **Endpoint:** `GET /update/trigger`
*   **Description:** If a newer firmware version is available (check `/status` or run `/update/check` first), this endpoint initiates the Over-The-Air (OTA) update process. The device will download the `firmware.bin` from GitHub and attempt to flash it. The device will become unresponsive during the update and reboot automatically upon success.
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
*   **Success Response (Initiation):**
    *   Code: `202 Accepted`
    *   Content-Type: `application/json`
    *   Body: `{"message":"Update started. Device will reboot if successful.", "url":"<firmware_download_url>"}` (The device may reboot before the client fully receives this).
*   **Error Responses:**
    *   `400 Bad Request`: No update is available, or the firmware URL could not be found.
    *   `401 Unauthorized`: Missing or incorrect password.
    *   `503 Service Unavailable`: An update is already in progress.
*   **Example Usage:** `http://<ESP_IP>/update/trigger?password=YOUR_PASSWORD`

**10. Reboot Device**

*   **Endpoint:** `GET /reboot`
*   **Description:** Instructs the device to perform a software reboot.
*   **Authentication:** **Required**.
*   **Parameters:**
    *   `password` (string, required): Your API password.
*   **Success Response (Initiation):**
    *   Code: `200 OK`
    *   Content-Type: `application/json`
    *   Body: `{"message":"Rebooting device..."}` (The device will reboot shortly after sending this).
*   **Error Responses:**
    *   `401 Unauthorized`: Missing or incorrect password.
*   **Example Usage:** `http://<ESP_IP>/reboot?password=YOUR_PASSWORD`

---

**Status Object Details (`GET /status`)**

The JSON object returned by `/status` and successful control calls:

```json
{
  "firmware_version": "v1.0.0",
  "update_available": false,          
  "latest_version": "v1.0.0",        
  "update_in_progress": false,       
  "wifi_mode": "STA",                
  "ip_address": "192.168.1.123",     
  "ap_ssid": "",                     
  "sta_ssid": "MyHomeWiFi",         
  "configured_sta_ssid": "MyHomeWiFi",
  "using_hardcoded_wifi": false,     
  "wifi_connected": true,           
  "master_brightness": 1023,       
  "active_effect": "none",           
  "pir": {                           
    "enabled": true,                
    "motion_detected": false       
  },
  "leds": [                        
    {
      "strip": 1,     
      "pin": 5,                    
      "brightness": 0,               
      "state": "off",              
      "base_brightness": 0           
    },
    {
      "strip": 2,
      "pin": 4,
      "brightness": 750,
      "state": "on",
      "base_brightness": 750
    }
  ]
}
```

---

**Effect Names**

Valid values for the `mode` parameter in `GET /effect`:

*   `none` (Disables effects)
*   `breathe`
*   `sequence_on`
*   `sequence_off`
*   `chase`
*   `swell`
*   `random_soft`
*   `cycle`
*   `candle`
*   `wave`
*   `ramp_up`
*   `ramp_down`

---
