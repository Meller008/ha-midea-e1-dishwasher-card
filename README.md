# 🍽️ Midea E1 Dishwasher Card for Home Assistant (`ha-midea-e1-dishwasher-card`)

A modern, animated, turnkey Lovelace card and package for **Midea E1 Dishwashers** (Model `760001FA`, Protocol Version 3) using local LAN control via [`midea_ac_lan`](https://github.com/wuwentao/midea_ac_lan).

---

> [!NOTE]
> **AI Collaboration & Translation Disclaimer:**
> This project, including the reverse-engineering of the M-Smart V3 byte frames, Home Assistant package scripts, custom UI animations, and this English documentation, was researched, developed, tested, and translated with the active assistance of **Artificial Intelligence (Google DeepMind Antigravity AI)** by a Russian-speaking community member.

---

<img width="521" height="138" alt="image" src="https://github.com/user-attachments/assets/aa2e1f19-5490-4701-a13f-c864874687ae" /><img width="535" height="151" alt="image" src="https://github.com/user-attachments/assets/4810189d-2cb6-4ddf-88ad-7a257462d226" />
<img width="531" height="145" alt="image" src="https://github.com/user-attachments/assets/63e6226f-b149-4102-bca9-0fe983e4580e" />
<img width="549" height="808" alt="image" src="https://github.com/user-attachments/assets/a925d2c1-3b57-46b6-acf2-c8d3313bd46a" /><img width="558" height="800" alt="image" src="https://github.com/user-attachments/assets/cd415dd2-35f2-4f24-9d49-ebb443759dcb" /><img width="553" height="805" alt="image" src="https://github.com/user-attachments/assets/a3ff7029-c7e5-4560-8907-5b5ec05bbaeb" />





## ✨ Features

- 🌊 **Animated Water Wave & Vibration:**
  - Dynamic water-filling simulation with smooth sine wave animation (`@keyframes wave`) inside the icon.
  - Gentle appliance vibration effect (`@keyframes shake`) while the wash cycle is running.
  - Amber pause indication with dedicated symbol (`mdi:pause-circle`).
- 💡 **Bidirectional Neon Progress Bar:**
  - **Delayed Start mode:** Progress bar counts **UP from 0% to 100%** (Cyan `#00E5FF`) as the start time approaches. Automatically captures and synchronizes with the timer even if delay is set physically on the dishwasher panel!
  - **Washing Cycle:** Progress bar counts **DOWN from 100% to 0%** (Emerald `#00E676`) showing remaining program time.
  - **Pause mode:** Turns Amber (`#FFB300`) and freezes until resumed.
  - **No bleed-through:** Isolated CSS stacking context (`isolation: isolate; z-index: 0`) ensures the progress bar never renders on top of popups.
- 🚪 **Hardware Auto-Door Opening (`door_auto_open`):**
  - Native command injection sending byte `01` (enable) or `02` (disable) to pop the door open automatically after drying.
- ⏳ **Hardware Delayed Start (`delay` / `order`):**
  - Native 38-byte order frame sending hours and minutes to the dishwasher with the physical display countdown on the machine.
  - Stepper with quick presets (`Reset (0h)`, `+1h`, `+2h`, `+3h`, `+4h`) and live start time preview (e.g. `3 h (starts at 01:05)`).
- 🛡️ **Intelligent Protection & Safeguards:**
  - **Door open alert:** Prevents starting programs or setting delayed start if the door is open, displaying an in-UI warning banner and notification.
  - **Delay lock:** Disables delayed start button if washing is already running or paused.
  - **Unified Pause / Resume:** Single smart button toggling between `Pause` and `Resume`.
- 📱 **Sleek Pop-up Modal (Bubble Card):**
  - iOS-style modal dialog (`#dishwasher`) housing mode selectors, consumables status badges (salt, rinse aid), door state, and controls.

---

## 📦 Prerequisites (HACS)

Install the following plugins via **HACS** (Home Assistant Community Store):

1. **[`midea_ac_lan`](https://github.com/wuwentao/midea_ac_lan)** (Local LAN integration for Midea appliances).
2. **[`button-card`](https://github.com/custom-cards/button-card)** (Advanced customizable button card).
3. **[`bubble-card`](https://github.com/Clooos/Bubble-Card)** (Modern iOS-style cards and pop-ups).

*(Note: `card-mod` is NOT required! All animations and styles are built natively into `custom:button-card`.)*

---

## 🚀 Installation Guide

### Step 1: Enable Packages in `configuration.yaml`

Make sure your `configuration.yaml` allows packages:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

### Step 2: Add `dishwasher.yaml` Package

1. Create directory `/config/packages/` if it doesn't exist.
2. Copy [`dishwasher.yaml`](./dishwasher.yaml) into `/config/packages/dishwasher.yaml`.
3. Open `dishwasher.yaml` in an editor and replace all occurrences of `YOUR_DEVICE_ID` with your actual Midea device ID (e.g. `153931629132569`).
   * *Tip: Find your device ID in Home Assistant under Settings -> Devices & Services -> Midea AC LAN.*
4. Restart Home Assistant Core (`Developer Tools` -> `YAML` -> `Restart`).

### Step 3: Add Cards to Your Lovelace Dashboard

1. Open your Lovelace dashboard, click `Edit Dashboard` -> `Raw configuration editor` (or add via Manual Card).
2. Copy the contents of [`lovelace_cards.yaml`](./lovelace_cards.yaml):
   - **Main Card:** Add under your desired view section.
   - **Pop-up Card:** Add in a dedicated section or at the bottom of your view (it binds to `#dishwasher` hash navigation).
3. Replace all occurrences of `YOUR_DEVICE_ID` with your actual device ID.
4. Save and enjoy!

---

## 🔬 Protocol & Technical Details (M-Smart V3 E1)

For developers and contributors to `midea_ac_lan`:

### Work Frame (Program Start / Run)
- Command Type: `cmd_type: 2`
- Total Frame Length: 38 bytes (76 hex characters)
- Byte Structure:
  - `08 03` — Work command header
  - Byte 3: Mode hex byte (`01`: Auto, `02`: Intensive/Germ, `04`: Eco, `05`: Glass, `06`: 90min, `07`: Fast, `0D`: Quiet Night)
  - Bytes 4–13: Reserved zeros (`00` × 10)
  - **Byte 14 (Index 13):** Auto-door open — `01` = Enable, `02` = Explicit Disable (`00` means unchanged)
  - Bytes 15–38: Reserved zeros (`00` × 24)

### Order Frame (Delayed Start)
- Command Type: `cmd_type: 2`
- Total Frame Length: 38 bytes (76 hex characters)
- Byte Structure:
  - `08 02` — Order (delay) command header
  - Byte 3: Mode hex byte
  - Bytes 4–7: Reserved zeros (`00` × 4)
  - **Byte 8 (Index 7):** Delay hours in hex (e.g. `03` for 3h, `0A` for 10h)
  - Bytes 9–13: Reserved zeros (`00` × 5)
  - **Byte 14 (Index 13):** Auto-door open — `01` = Enable, `02` = Disable
  - Bytes 15–38: Reserved zeros (`00` × 24)

---

## 🔗 Credits & References

- **Midea AC LAN:** [Issue #709](https://github.com/wuwentao/midea_ac_lan/issues/709) — Reverse engineering discussion and community testing for Midea E1 dishwashers.
- **Card Concepts & Visual Inspiration:** [Anashost/HA-Animated-cards](https://github.com/Anashost/HA-Animated-cards/tree/main) for early ideas on animated appliances.

---

## 📄 License

MIT License. Free for personal and commercial use and modification. Community pull requests are welcome!
