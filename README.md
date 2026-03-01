# 智慧桌燈系統 Smart Desk Lamp

本專題以 ESP32 為核心控制器，結合多種感測器與 MQTT 通訊協定，打造一套具備環境感知與遠端控制能力的智慧桌燈系統。

系統透過光敏電阻自動偵測環境亮度並反比調整燈光強度，搭配 DHT11 溫濕度感測器持續監測學習環境；超音波感測器則用於偵測使用者是否長時間在座，一旦超過設定閾值即以紅燈閃爍提示起身活動。8×8 LED 矩陣作為本地顯示介面，輪流顯示當前溫度與濕度數值。

所有感測數據透過 MQTT 協定上傳至 Mosquitto Broker，再由 Node-RED 進行視覺化呈現與自動化控制，使用者亦可透過 Node-RED 儀表板遠端調整燈光亮度或重置久坐提醒。

---

## 目錄

- [系統架構](#系統架構)
- [功能特色](#功能特色)
- [硬體規格](#硬體規格)
- [環境需求](#環境需求)
- [安裝與部署](#安裝與部署)
- [MQTT 通訊協定](#mqtt-通訊協定)
- [專案結構](#專案結構)
- [故障排除](#故障排除)

---

## 系統架構

```
┌─────────────────────────────────────────────────────────┐
│                      ESP32 韌體                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │ DHT11    │  │ 超音波   │  │ 光敏電阻 │  │LED矩陣 │  │
│  │ 溫濕度   │  │ 距離偵測 │  │ 環境亮度 │  │8x8顯示 │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └───┬────┘  │
│       └─────────────┴──────────────┴─────────────┘      │
│                        ESP32 Core                        │
│                     WiFi + MQTT Client                   │
└───────────────────────────┬─────────────────────────────┘
                            │ MQTT (port 1883)
                            ▼
                   ┌─────────────────┐
                   │ Mosquitto Broker │
                   └────────┬────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │   Node-RED      │
                   │  儀表板 :1880   │
                   │ - 燈光控制      │
                   │ - 環境監測      │
                   │ - 久坐提醒      │
                   └─────────────────┘
```

---

## 功能特色

| 功能 | 說明 |
|------|------|
| 智慧光線控制 | 光敏電阻偵測環境亮度，自動反比調整 LED 亮度 |
| 環境監測 | DHT11 每 5 秒讀取溫濕度，透過 MQTT 上傳 |
| 久坐提醒 | 超音波感測器偵測在座狀態，超時紅燈閃爍警示 |
| LED 矩陣顯示 | 8×8 矩陣交替顯示溫度與濕度數值 |
| 遠端控制 | Node-RED 儀表板遠端調整亮度、重置提醒 |
| 數據可視化 | Node-RED 即時圖表顯示環境趨勢 |

---

## 硬體規格

### 元件清單

| 元件 | 型號 | 腳位 | 用途 |
|------|------|------|------|
| 微控制器 | ESP32 | — | 核心處理器 |
| 溫濕度感測器 | DHT11 | GPIO 25 | 環境監測 |
| 超音波感測器 | HC-SR04 | Trig: GPIO 16 / Echo: GPIO 17 | 久坐偵測 |
| 光敏電阻 | LDR | GPIO 27 (ADC) | 環境亮度偵測 |
| LED 燈 | 白色 LED | GPIO 26 | 主要照明 |
| 紅色 LED | 5mm LED | GPIO 5 | 久坐警示燈 |
| LED 矩陣 | MAX7219 8×8 | DIN: 19 / CLK: 21 / CS: 18 | 數值顯示 |

### 腳位配置圖

```
ESP32
├── GPIO 16 ──► HC-SR04 Trig
├── GPIO 17 ◄── HC-SR04 Echo
├── GPIO 18 ──► MAX7219 CS
├── GPIO 19 ──► MAX7219 DIN
├── GPIO 21 ──► MAX7219 CLK
├── GPIO 25 ◄── DHT11 DATA
├── GPIO 26 ──► White LED (PWM)
├── GPIO 27 ◄── LDR (ADC)
└── GPIO 5  ──► Red LED
```

---

## 環境需求

### 軟體需求

| 軟體 | 版本需求 | 說明 |
|------|---------|------|
| Arduino IDE | 2.x 以上 | ESP32 韌體開發 |
| Node.js | 18.x 以上 | Node-RED 執行環境 |
| Mosquitto | 2.x | MQTT Broker |
| Node-RED | 3.x 以上 | 視覺化流程控制 |

### Arduino 函式庫

請在 Arduino IDE 的函式庫管理員中安裝以下套件：

- `PubSubClient` — MQTT 客戶端
- `DHT sensor library` — DHT11/DHT22 支援
- `LedControl` / `LedControlMS` — MAX7219 LED 矩陣驅動
- `Ultrasonic` — HC-SR04 超音波感測器

---

## 安裝與部署

### 1. 複製專案

```bash
git clone <repository-url>
cd esp32-smart-desk-lamp
```

### 2. 安裝系統依賴（macOS）

```bash
# 安裝 Homebrew（若尚未安裝）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安裝 Node.js 與 Mosquitto
brew install node mosquitto
```

### 3. 安裝 Node-RED

```bash
npm install -g --unsafe-perm node-red
```

### 4. 安裝專案 Node-RED 依賴

```bash
npm install
```

### 5. 設定 ESP32 韌體

開啟 `Arduino_DHT/Arduino_DHT.ino`，修改以下設定：

```cpp
// WiFi 設定
const char* ssid     = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// MQTT Broker IP（請填入主機的實際 IP）
const char* mqtt_server = "192.168.x.x";
```

### 6. 啟動系統

**步驟 1：啟動 MQTT Broker**
```bash
mosquitto -c mosquitto.conf
```

**步驟 2：啟動 Node-RED（使用專案設定）**
```bash
npm start
```

> **注意**：請務必從專案根目錄執行 `npm start`，以確保載入 `node-red-data/flows.json` 中的流程設定。請勿直接執行 `node-red` 指令，否則將使用系統預設路徑的流程。

**步驟 3：開啟儀表板**

在瀏覽器中訪問：[http://localhost:1880](http://localhost:1880)

**步驟 4：上傳韌體至 ESP32**

1. 以 Arduino IDE 開啟 `Arduino_DHT/Arduino_DHT.ino`
2. 選擇正確的開發板（`ESP32 Dev Module`）與 COM 埠
3. 點擊「上傳」

### 7. 驗證系統運作

```bash
# 監聽所有 smartlamp 主題的 MQTT 訊息
mosquitto_sub -h localhost -t "smartlamp/#" -v
```

---

## MQTT 通訊協定

### 主題結構

| 主題 | 方向 | 說明 |
|------|------|------|
| `smartlamp/light` | 雙向 | 燈光亮度控制與狀態回報 |
| `smartlamp/environment` | ESP32 → Broker | 溫度、濕度、光線強度 |
| `smartlamp/reminder` | 雙向 | 久坐偵測狀態與重置指令 |
| `smartlamp/timer` | 雙向 | 計時器狀態 |
| `smartlamp/alarm` | Node-RED → ESP32 | 鬧鐘設定 |
| `smartlamp/settings/sitting_reminder` | Node-RED → ESP32 | 久坐提醒時間閾值設定 |

### Payload 格式範例

```jsonc
// smartlamp/environment（每 5 秒發布）
{ "temperature": 27.00, "humidity": 84.00, "light": 141 }

// smartlamp/reminder（每 5 秒發布）
{ "sitting": true, "alarm": false, "duration": 134 }

// smartlamp/light（亮度控制）
{ "brightness": 200 }

// 重置久坐提醒（Node-RED 發送）
"reset"
```

---

## 專案結構

```
esp32-smart-desk-lamp/
├── Arduino_DHT/
│   └── Arduino_DHT.ino        # ESP32 主程式韌體
├── node-red-data/
│   ├── flows.json              # Node-RED 流程定義
│   ├── flows_cred.json         # Node-RED 流程憑證（請勿公開）
│   ├── settings.js             # Node-RED 伺服器設定
│   └── package.json            # Node-RED 依賴套件
├── mosquitto.conf              # MQTT Broker 設定檔
├── package.json                # 專案啟動設定
└── README.md
```

---

## 故障排除

**Q: ESP32 無法連接 MQTT Broker**
- 確認 ESP32 與主機在同一個 WiFi 網段
- 確認 `mosquitto.conf` 中已啟用 `allow_anonymous true` 或設定正確的認證資訊
- 確認防火牆未封鎖 TCP Port `1883`

**Q: Node-RED 啟動後看不到流程**
- 確認是在專案根目錄執行 `npm start`，而非直接執行 `node-red`
- 若曾使用預設路徑，可手動複製流程：
  ```bash
  cp ~/.node-red/flows.json ./node-red-data/flows.json
  ```

**Q: Node-RED 無法啟動**
- 確認 Port `1880` 未被其他程序佔用：`lsof -i :1880`
- 確認 `node-red-data/` 目錄存在且具有讀寫權限

**Q: DHT11 讀取失敗**
- 確認接線正確（VCC / DATA / GND）
- DATA 腳位需加上 10kΩ 上拉電阻
- 確認 Arduino 程式中 `DHTPIN` 定義與實際接線一致（GPIO 25）
