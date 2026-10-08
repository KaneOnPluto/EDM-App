# EDM-App

React app for the Environmental Divergence Meter, includes simulation, documentation, live data fetch through json fetch, straight from esp32, search options, API fetches, etc. Clean terminal style theme for that retro feel.

## Contributors

- UI/UX and App development (Major Developer) - https://github.com/YaxYiran
- Divergence model, ESP32 backend (soon) - Me!

---

# Running EDM-App with Expo Go

This guide explains how to run the EDM-App on an Android phone using **Expo Snack + Expo Go**.

You do **not** need to install the project locally for this method.

---

## 1. Requirements

You need:

- An Android phone
- **Expo Go** installed on the Android phone
- An internet connection
- The project's `App.js`
- Access to a Supabase project if you want to use the Dashboard
- The ESP32 and its Wi-Fi network if you want to use **LIVE DATA**
- Read the Supabase schema for more info

Install Expo Go from the Google Play Store.

---

# 2. Open Expo Snack

Go to:

https://snack.expo.dev/

Create a new Snack.

You should see the Expo Snack editor with an `App.js` file.

---

# 3. Upload `App.js`

Open the provided EDM-App `App.js`.

Copy the **entire contents** of `App.js`.

Replace the default `App.js` in Expo Snack with the EDM-App `App.js`.

After uploading/pasting `App.js`, Expo Snack should detect the packages used by the application and prompt you to **automatically install the required dependencies**.

Accept the dependency installation prompt.

The application uses packages including:

```text
@supabase/supabase-js
react-native-svg
react-native-chart-kit
```

You should **not need to manually add these packages** if Snack correctly detects them from `App.js`.

---

# 4. Add the Supabase URL and Key

The Dashboard section of the application connects to Supabase.

In `App.js`, near the top of the file, find:

```javascript
const supabase = createClient(
  'SUPA_URL',
  'API SECERET'
);
```

Replace the two placeholder values with your Supabase project information.

For example:

```javascript
const supabase = createClient(
  'https://your-project.supabase.co',
  'your-supabase-anon-key'
);
```

### Where to get these values

From your Supabase project, obtain:

- **Project URL**
- **Anon / public API key**

Do not put a Supabase `service_role` key into `App.js`.

The app only needs the client-side key intended for public applications.

After replacing the values, save `App.js`.

---

# 5. Configure the ESP32 Wi-Fi

The Wi-Fi credentials are **not entered into `App.js`**.

They are configured inside the ESP32 Arduino code, `EDM.ino`.

Open:

```text
EDM.ino
```

Find:

```cpp
const char* ssid = "WIFI_ID";
const char* password = "WIFI_PASSWORD";
```

Replace them with the Wi-Fi network that the ESP32 should connect to:

```cpp
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";
```

For example:

```cpp
const char* ssid = "MyWiFi";
const char* password = "MyPassword123";
```

Upload the modified `EDM.ino` to the ESP32.

The ESP32 will connect to this Wi-Fi during startup.

---

# 6. Find the ESP32 IP Address

After uploading `EDM.ino`, open the Arduino Serial Monitor.

Use:

```text
115200 baud
```

After the ESP32 connects to Wi-Fi, it prints:

```text
WiFi connected
IP: xxx.xxx.xxx.xxx
```

For example:

```text
WiFi connected
IP: 192.168.0.202
```

Copy this IP address.

The IP can change if the ESP32 reconnects to the network, so check the Serial Monitor if the app stops connecting.

---

# 7. Configure the ESP32 IP in `App.js`

Open `App.js`.

Inside `LiveDataMode()`, find:

```javascript
const res = await fetch('http://192.168.0.202/data');
```

Replace:

```text
192.168.0.202
```

with the current IP address printed by the ESP32.

For example, if the ESP32 prints:

```text
IP: 192.168.1.15
```

change it to:

```javascript
const res = await fetch('http://192.168.1.15/data');
```

The `/data` endpoint must remain:

```text
/data
```

The ESP32 provides this endpoint through its web server.

---

# 8. Important Wi-Fi Requirement for LIVE DATA

For the Android phone running Expo Go to communicate directly with the ESP32:

```text
Android Phone
      |
      | Wi-Fi
      |
   Router
      |
      | Wi-Fi
      |
    ESP32
```

The phone and ESP32 should be connected to the **same local network**.

For example:

```text
Phone  → MyWiFi
ESP32  → MyWiFi
```

If the phone is using mobile data while the ESP32 is connected to your Wi-Fi, the phone may not be able to access the ESP32's local IP address.

---

# 9. Start the Expo Snack

Once:

- `App.js` has been uploaded
- Dependencies have been installed
- Supabase URL/key have been configured
- ESP32 IP has been configured if LIVE DATA is required

start the Snack.

Expo Snack will generate a QR code.

---

# 10. Open the App with Expo Go

On the Android phone:

1. Open **Expo Go**.
2. Scan the QR code shown by Expo Snack.
3. Wait for the project to load.
4. The EDM-App should open.

You can now navigate between:

```text
[LIVE DATA]
[SIMULATION]
[DOCS]
[DASHBOARD]
```

---

# 11. Test Simulation Mode

You can test the application without the ESP32.

Open:

```text
SIMULATION
```

There are two simulation modes:

### Manual Input

Enter:

- Temperature
- Humidity
- IAQ
- Luminosity
- Sound

The application calculates the divergence value locally.

### API Simulation

Use the location search to select a city.

The application retrieves:

- Location data
- Weather data
- Air-quality data

and uses those values to simulate an indoor environment.

No ESP32 connection is required for Simulation Mode.

---

# 12. Test LIVE DATA

Make sure the ESP32 is:

- Powered on
- Connected to Wi-Fi
- Running the EDM firmware
- Showing a valid IP address in Serial Monitor

Then open:

```text
LIVE DATA
```

The application requests:

```text
http://ESP32_IP/data
```

The ESP32 returns JSON containing:

```json
{
  "temperature": 23.5,
  "humidity": 45,
  "iaq": 35,
  "lux": 400,
  "sound": 35,
  "divergence": 0.823456
}
```

The app refreshes the live data automatically every **3 seconds**.

You can also press:

```text
[ REFRESH DATA ]
```

to request the data manually.

---

# 13. Test the Dashboard

Open:

```text
DASHBOARD
```

The Dashboard reads historical sensor data from the Supabase `sensor_data` table.

The application currently queries:

```javascript
supabase
  .from('sensor_data')
  .select('*')
```

The Dashboard uses this data for:

- Current divergence
- Sensor snapshot
- Historical graph
- Highest/lowest divergence
- Distribution
- Trend detection
- Alerts
- Daily summary
- Recommendations

If the Supabase URL/key is incorrect, or the required table/data is unavailable, the Dashboard will not be able to retrieve the stored data.

---

# 14. Troubleshooting

## App does not start in Snack

Check whether Expo Snack reported a dependency installation error.

If Snack asks to install detected dependencies, accept the installation.

Then wait for the project to rebuild.

---

## Dashboard is empty

Check the Supabase configuration in `App.js`:

```javascript
const supabase = createClient(
  'SUPA_URL',
  'API SECERET'
);
```

Make sure the values have been replaced with the correct Supabase project URL and public/anon key.

Also make sure the Supabase project contains the expected `sensor_data` table and records.

---

## LIVE DATA says `DISCONNECTED`

Check:

1. ESP32 is powered on.
2. ESP32 successfully connected to Wi-Fi.
3. Phone and ESP32 are on the same network.
4. The ESP32 IP address is correct in `App.js`.
5. The ESP32 web server is running.
6. The endpoint is:

```text
http://ESP32_IP/data
```

7. Try opening the ESP32 endpoint from a device on the same network.

---

## ESP32 IP changed

Check the Arduino Serial Monitor again.

You may see a different:

```text
IP: ...
```

Update this line in `App.js`:

```javascript
const res = await fetch('http://NEW_ESP32_IP/data');
```

Reload the Expo Snack application.

---

## Supabase works but ESP32 does not

These are separate connections.

The application communicates with:

```text
Supabase
    ↑
    |
  App.js
```

and:

```text
ESP32
  ↑
  |
App.js
```

A working Supabase connection does not mean the ESP32 connection is working.

---

# 15. Final Setup Checklist

Before testing the complete system, verify:

### Expo

- [ ] Expo Snack is open
- [ ] `App.js` has been uploaded
- [ ] Expo Snack installed the detected dependencies
- [ ] Expo Go is installed on Android
- [ ] QR code opens the app

### Supabase

- [ ] Supabase URL added to `App.js`
- [ ] Supabase anon/public key added to `App.js`
- [ ] `sensor_data` table exists
- [ ] Supabase contains sensor data if Dashboard testing is required

### ESP32

- [ ] Wi-Fi SSID added to `EDM.ino`
- [ ] Wi-Fi password added to `EDM.ino`
- [ ] `EDM.ino` uploaded to ESP32
- [ ] ESP32 connects successfully
- [ ] ESP32 IP obtained from Serial Monitor
- [ ] ESP32 IP added to `App.js`
- [ ] Phone and ESP32 are on the same Wi-Fi network

### App

- [ ] LIVE DATA tested
- [ ] SIMULATION tested
- [ ] DOCS tested
- [ ] DASHBOARD tested

---

# Quick Setup Summary

If you only want the shortest version:

```text
1. Open Expo Snack
        ↓
2. Upload/paste App.js
        ↓
3. Accept automatic dependency installation
        ↓
4. Add Supabase URL + anon/public key to App.js
        ↓
5. Put Wi-Fi SSID + password in EDM.ino
        ↓
6. Upload EDM.ino to ESP32
        ↓
7. Get ESP32 IP from Serial Monitor
        ↓
8. Put ESP32 IP into App.js
        ↓
9. Start Snack
        ↓
10. Scan QR code using Expo Go
        ↓
11. Test the app
```

The **ESP32, Wi-Fi, and Supabase configuration are only required for the corresponding live/database functionality**. Simulation and documentation can be tested without the ESP32.
