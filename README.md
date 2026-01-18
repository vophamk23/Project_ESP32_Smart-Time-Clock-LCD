# 🕐 Smart Clock ESP32

### **Hệ thống đồng hồ thông minh đa chức năng sử dụng ESP32**

<div align="center">
A feature-rich smart clock built with ESP32, featuring real-time display, temperature monitoring, alarm, stopwatch, and countdown timer.
<br>
<img src="" alt="Smart Clock Image"/>
</div>

---

## 📋 **Mục lục**

* [Giới thiệu](#giới-thiệu)
* [Tính năng](#tính-năng)
* [Yêu cầu phần cứng](#yêu-cầu-phần-cứng)
* [Sơ đồ chân](#sơ-đồ-chân)
* [Cài đặt](#cài-đặt)
* [Hướng dẫn sử dụng](#hướng-dẫn-sử-dụng)
* [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)
* [Định dạng hiển thị](#định-dạng-hiển-thị)
* [Cấu trúc dự án](#cấu-trúc-dự-án)
* [Gỡ lỗi](#gỡ-lỗi)
* [Hướng phát triển](#hướng-phát-triển)
* [Tác giả & License](#tác-giả--license)

---

## 🌟 **Giới thiệu**

**Smart Clock ESP32** là một hệ thống nhúng tích hợp nhiều chức năng thời gian thực, được xây dựng trên nền tảng **ESP32 DevKit**. Hệ thống sử dụng kiến trúc **Cooperative Scheduler** để xử lý đa nhiệm, đồng thời hiển thị thông tin trên **LED 7-Segment MAX7219** và **LCD I2C 16x2**.

**Điểm nổi bật:**

* ⏰ **5 chế độ hoạt động**: Thời gian–Ngày tháng, Nhiệt độ–Độ ẩm, Báo thức, Bấm giờ, Đếm ngược.
* 🔄 **Bộ lập lịch thời gian thực** (10ms tick – O(1) update).
* 📊 **Hệ thống hiển thị kép**: MAX7219 + LCD 16x2.
* 🌡️ **Giám sát môi trường**: Cảm biến DHT11.
* 🔔 **Cảnh báo thông minh**: Buzzer + LED.

---

## ✨ **Tính năng**

### 🕰️ Chế độ 1: Ngày & Giờ

* Đồng bộ thời gian với **RTC DS3231**.
* Hiển thị HH:MM:SS và DD/MM/YYYY.

### 🌡️ Chế độ 2: Nhiệt độ & Độ ẩm

* Đọc cảm biến DHT11.
* Cập nhật mỗi 2 giây.

### ⏰ Chế độ 3: Báo thức

* Chỉnh giờ/phút.
* Báo bằng LED + Buzzer.
* Nhấp nháy khi chỉnh.

### ⏱️ Chế độ 4: Đồng hồ bấm giờ

* Độ chính xác centiseconds.
* Ghi tối đa 5 vòng (Lap).

### ⏲️ Chế độ 5: Đếm ngược

* Cài đặt giờ–phút–giây.
* Reset nhanh / cảnh báo khi hết thời gian.

---

## 🔧 **Yêu cầu phần cứng**

| Thành phần            | Mô tả               | Số lượng |
| --------------------- | ------------------- | -------- |
| ESP32 DevKit          | Vi điều khiển chính | 1        |
| MAX7219               | LED 7-seg 8 digit   | 1        |
| LCD I2C 16x2          | Hiển thị phụ        | 1        |
| DS3231                | Mô-đun RTC          | 1        |
| DHT11                 | Cảm biến            | 1        |
| Buzzer                | Còi cảnh báo        | 1        |
| LED + điện trở        | Báo hiệu            | 1        |
| Nút nhấn MODE/SET/INC | Điều khiển          | 3        |

---

## 📌 **Sơ đồ chân**

```
ESP32
├── MAX7219: DIN=13, CLK=14, CS=12
├── LCD I2C: SDA=21, SCL=22
├── DHT11: DATA=27
├── Buttons: MODE=16, SET=17, INC=5
├── Buzzer: 32
└── LED: 33 → 220Ω → GND
```

---

## 💻 **Cài đặt**

### 1. Cài Arduino IDE hoặc PlatformIO

### 2. Thêm board ESP32

```
https://dl.espressif.com/dl/package_esp32_index.json
```

### 3. Cài thư viện

* LedControl
* RTC DS3231 (RTClib)
* DHT sensor library
* LiquidCrystal_I2C

### 4. Nạp chương trình

* Chọn board: **ESP32 Dev Module**
* Chọn COM
* Upload

---

## 📖 **Hướng dẫn sử dụng**

### Bảng điều khiển

| Nút  | Chức năng              |
| ---- | ---------------------- |
| MODE | Chuyển chế độ          |
| SET  | Vào chỉnh/Start-Stop   |
| INC  | Tăng giá trị/Lap/Reset |

### Tóm tắt 5 chế độ

* **Mode 1:** Hiển thị nhiệt độ & độ ẩm
* **Mode 2:** Hiển thị ngày giờ
* **Mode 3:** Đặt báo thức
* **Mode 4:** Bấm giờ, xem Lap
* **Mode 5:** Đếm ngược, reset nhanh

---

## 🏗️ **Kiến trúc hệ thống**

### ⏱️ Cooperative Scheduler

```
Timer 10ms → ISR → SCH_Update() → O(1)
                   ↓
           SCH_Dispatch_Tasks()
```

| Task          | Period | Mô tả             |
| ------------- | ------ | ----------------- |
| CheckButtons  | 50ms   | Quét nút          |
| UpdateDisplay | 100ms  | LED 7-seg         |
| UpdateLCD     | 100ms  | LCD               |
| LEDBlink      | 100ms  | Chớp LED          |
| CheckAlarm    | 1000ms | Kiểm tra báo thức |
| ReadSensors   | 2000ms | Đọc DHT11         |

---

## 📺 **Định dạng hiển thị**

### LED 7-Segment Format
```
Mode 1: [HH.MM] [DD.MM]     (Time & Date)
Mode 2: [TT.T°C] [HH.H%]    (Temp & Humidity)
Mode 3: [HH.MM] [HH.MM]     (Current Time | Alarm Time)
Mode 4: [HH.MM.SS.CC]       (Stopwatch)
Mode 5: [HH.MM.SS.CC]       (Countdown)
```

### LCD 16x2 Format
```
┌────────────────┐
│Temp: 25.5°C    │  Mode 1
│Humi: 67.2%     │
└────────────────┘

┌────────────────┐
│Time: 14:35:42  │  Mode 2
│Date: 25/12/2024│
└────────────────┘

┌────────────────┐
│Now: 14:35:42   │  Mode 3
│Alarm: 10:30    │
└────────────────┘

┌────────────────┐
│Stopwatch RUN   │  Mode 4
│01:23:45.67 L3  │
└────────────────┘

┌────────────────┐
│Countdown RUN   │  Mode 5
│00:03:00.00     │
└────────────────┘
```


---

## 📁 **Cấu trúc dự án**

```
smart-clock-esp32/
├── main.cpp
├── scheduler/
├── App_Tasks/
├── button_handler/
├── lcd_display/
├── led_7seg_display/
├── sensor_handler/
├── global_vars/
└── README.md
```

---

## 🐛 **Gỡ lỗi**

| Lỗi                | Nguyên nhân       | Khắc phục           |
| ------------------ | ----------------- | ------------------- |
| LCD không hiển thị | Sai địa chỉ I2C   | Thử 0x27 hoặc 0x3F  |
| DHT lỗi            | Nguồn/đấu dây kém | Kiểm tra lại wiring |
| Nút loạn           | Debounce thấp     | Tăng debounce       |

---

## 🚀 **Hướng phát triển**

* [ ] Đồng bộ thời gian NTP qua WiFi
* [ ] Giao tiếp MQTT
* [ ] Thêm nhiều báo thức
* [ ] Ghi log vào SD Card
* [ ] App điều khiển qua Bluetooth

---

## 👨‍💻 **Tác giả & License**

**Author:** *Phạm Công Võ*

* GitHub: **@vophamk23**
* Email: [congvolv1@gmail.com](mailto:your.email@example.com)

# 🕐 Smart Clock ESP32

## A Multi-Function Smart Clock System Based on ESP32

<div align="center">

A feature-rich embedded smart clock built on **ESP32**, supporting real-time clock display, temperature & humidity monitoring, alarm, stopwatch, and countdown timer.

<br>

<img src="assets/smart_clock_overview.jpg" alt="Smart Clock ESP32" width="600"/>

</div>

---

## 📌 Table of Contents

- Introduction
- Key Features
- System Modes
- Hardware Requirements
- Pin Configuration
- Software Setup
- User Guide
- System Architecture
- Display Format
- Project Structure
- Troubleshooting
- Future Improvements
- Author & License

---

## 🌟 Introduction

**Smart Clock ESP32** is an embedded system project developed using the **ESP32 DevKit**.
The project demonstrates real-time task scheduling, sensor integration, and multi-mode user interaction.

The system is designed with a **Cooperative Scheduler** using a **10 ms system tick**, ensuring deterministic and efficient task execution.

---

## ✨ Key Features

- Real-Time Clock using **DS3231**
- Temperature & Humidity Monitoring (DHT11)
- Alarm with LED and Buzzer
- Stopwatch with centisecond accuracy
- Countdown Timer with alert
- Dual display system (7-segment + LCD)
- Button-based user interface

---

## 🧭 System Modes

| Mode | Description |
|------|------------|
| Mode 1 | Date & Time Display |
| Mode 2 | Temperature & Humidity |
| Mode 3 | Alarm Setup |
| Mode 4 | Stopwatch (Lap supported) |
| Mode 5 | Countdown Timer |

---

## 🔧 Hardware Requirements

| Component | Description | Quantity |
|----------|------------|----------|
| ESP32 DevKit | Main controller | 1 |
| MAX7219 | 8-digit 7-segment display | 1 |
| LCD 16x2 I2C | Secondary display | 1 |
| DS3231 | RTC module | 1 |
| DHT11 | Temperature & humidity sensor | 1 |
| Buzzer | Audio alert | 1 |
| LED + Resistor | Visual alert | 1 |
| Push Buttons | MODE / SET / INC | 3 |

---

## 📌 Pin Configuration

```text
ESP32
├── MAX7219: DIN = GPIO13, CLK = GPIO14, CS = GPIO12
├── LCD I2C: SDA = GPIO21, SCL = GPIO22
├── DHT11: DATA = GPIO27
├── Buttons:
│   ├── MODE = GPIO16
│   ├── SET  = GPIO17
│   └── INC  = GPIO5
├── Buzzer = GPIO32
└── LED    = GPIO33 → 220Ω → GND
💻 Software Setup
1. Install Arduino IDE or PlatformIO
2. Add ESP32 Board Support
text
Sao chép mã
https://dl.espressif.com/dl/package_esp32_index.json
3. Required Libraries
LedControl

RTClib

DHT sensor library

LiquidCrystal_I2C

4. Upload Firmware
Board: ESP32 Dev Module

Select correct COM port

Upload firmware

📖 User Guide
Button Functions
Button	Function
MODE	Change operating mode
SET	Enter setting / Start / Stop
INC	Increase value / Lap / Reset

🏗️ System Architecture
Cooperative Scheduler
text
Sao chép mã
Timer Interrupt (10 ms)
        ↓
   SCH_Update()
        ↓
 SCH_Dispatch_Tasks()
Scheduled Tasks
Task	Period	Description
CheckButtons	50 ms	Button scan & debounce
UpdateDisplay	100 ms	7-segment update
UpdateLCD	100 ms	LCD refresh
LEDBlink	100 ms	LED blinking
CheckAlarm	1000 ms	Alarm check
ReadSensors	2000 ms	Read DHT11

📺 Display Format
7-Segment Display
text
Sao chép mã
Mode 1: [HH.MM] [DD.MM]        Time & Date
Mode 2: [TT.T°C] [HH.H%]       Temperature & Humidity
Mode 3: [HH.MM] [HH.MM]        Current | Alarm Time
Mode 4: [HH.MM.SS.CC]          Stopwatch
Mode 5: [HH.MM.SS.CC]          Countdown
LCD 16x2 Display
text
Sao chép mã
┌────────────────┐
│ Temp: 25.5°C   │
│ Humi: 67.2%    │
└────────────────┘
📁 Project Structure
text
Sao chép mã
smart-clock-esp32/
├── main.cpp
├── scheduler/
├── app_tasks/
├── button_handler/
├── lcd_display/
├── led_7seg_display/
├── sensor_handler/
├── global_vars/
└── README.md
🐛 Troubleshooting
Issue	Cause	Solution
LCD not displayed	Wrong I2C address	Try 0x27 or 0x3F
DHT11 error	Power or wiring issue	Check connections
Button bouncing	Debounce too small	Increase debounce time

🚀 Future Improvements
 NTP time synchronization via WiFi

 MQTT communication

 Multiple alarms

 SD card data logging

 Bluetooth / Mobile App control

👨‍💻 Author & License
Author: Phạm Công Võ
GitHub: @vophamk23
Email: congvolv1@gmail.com

License: MIT License

Sao chép mã






Bạn đã nói:
lại đi chỉ cần mã edit


ChatGPT đã nói:
md
Sao chép mã
# 🕐 Smart Clock ESP32

## A Multi-Function Smart Clock System Based on ESP32

<div align="center">
A feature-rich embedded smart clock built on **ESP32**, supporting real-time clock display, temperature & humidity monitoring, alarm, stopwatch, and countdown timer.
<br>
<img src="assets/smart_clock_overview.jpg" alt="Smart Clock ESP32" width="600"/>
</div>

---

## 📌 Table of Contents
- Introduction
- Key Features
- System Modes
- Hardware Requirements
- Pin Configuration
- Software Setup
- User Guide
- System Architecture
- Display Format
- Project Structure
- Troubleshooting
- Future Improvements
- Author & License

---

## 🌟 Introduction

**Smart Clock ESP32** is an embedded system project developed using the **ESP32 DevKit**.  
The system applies a **Cooperative Scheduler** with a **10 ms system tick** to handle multiple real-time tasks efficiently.

---

## ✨ Key Features

- DS3231 Real-Time Clock
- Temperature & Humidity Monitoring (DHT11)
- Alarm with LED & Buzzer
- Stopwatch (centisecond accuracy)
- Countdown Timer
- Dual display: MAX7219 + LCD 16x2
- Button-based user interface

---

## 🧭 System Modes

| Mode | Description |
|------|------------|
| Mode 1 | Date & Time |
| Mode 2 | Temperature & Humidity |
| Mode 3 | Alarm |
| Mode 4 | Stopwatch |
| Mode 5 | Countdown |

---

## 🔧 Hardware Requirements

| Component | Description | Qty |
|---------|------------|-----|
| ESP32 DevKit | Main MCU | 1 |
| MAX7219 | 7-seg display | 1 |
| LCD 16x2 I2C | LCD display | 1 |
| DS3231 | RTC module | 1 |
| DHT11 | Temp/Humidity | 1 |
| Buzzer | Alarm | 1 |
| LED + Resistor | Indicator | 1 |
| Push Buttons | MODE / SET / INC | 3 |

---

## 📌 Pin Configuration

```text
ESP32
├── MAX7219: DIN=13, CLK=14, CS=12
├── LCD I2C: SDA=21, SCL=22
├── DHT11: DATA=27
├── Buttons: MODE=16, SET=17, INC=5
├── Buzzer: GPIO32
└── LED: GPIO33 → 220Ω → GND
💻 Software Setup
Install Arduino IDE / PlatformIO

Add ESP32 board:

arduino
Sao chép mã
https://dl.espressif.com/dl/package_esp32_index.json
Install libraries:

LedControl

RTClib

DHT sensor library

LiquidCrystal_I2C

Upload firmware (ESP32 Dev Module)

📖 User Guide
Button	Function
MODE	Change mode
SET	Set / Start / Stop
INC	Increase / Lap / Reset

🏗️ System Architecture
text
Sao chép mã
Timer 10ms
   ↓
SCH_Update()
   ↓
SCH_Dispatch_Tasks()
Task	Period
CheckButtons	50 ms
UpdateDisplay	100 ms
UpdateLCD	100 ms
LEDBlink	100 ms
CheckAlarm	1000 ms
ReadSensors	2000 ms

📺 Display Format
7-Segment
text
Sao chép mã
Mode 1: Time & Date
Mode 2: Temp & Humi
Mode 3: Current | Alarm
Mode 4: Stopwatch
Mode 5: Countdown
LCD 16x2
text
Sao chép mã
Temp: 25.5°C
Humi: 67.2%
📁 Project Structure
text
Sao chép mã
smart-clock-esp32/
├── main.cpp
├── scheduler/
├── app_tasks/
├── button_handler/
├── lcd_display/
├── led_7seg_display/
├── sensor_handler/
├── global_vars/
└── README.md
🐛 Troubleshooting
Issue	Solution
LCD not working	Try I2C 0x27 / 0x3F
DHT error	Check wiring
Button noise	Increase debounce

🚀 Future Improvements
NTP time sync (WiFi)

MQTT support

Multiple alarms

SD card logging

Bluetooth control

👨‍💻 Author & License
Author: Phạm Công Võ
GitHub: @vophamk23
Email: congvolv1@gmail.com
