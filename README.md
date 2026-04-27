# 🚗 Smart Parking System

This project is an Arduino-based smart parking system designed to automate vehicle entry and efficiently manage parking space allocation. The system detects incoming vehicles, analyzes parking availability, and guides the driver to the nearest available spot.

## 🔧 Components Used

* Arduino Uno – main microcontroller
* Ultrasonic Sensor (HC-SR04) – vehicle detection at the entrance
* 4 IR Sensors – parking space occupancy detection
* Servo Motor – barrier gate control
* 16x2 I2C LCD Display – user interface

## ⚙️ Working Principle

* When a vehicle approaches the entrance, it is detected by the ultrasonic sensor
* A "Welcome" message is displayed on the LCD screen
* The system reads data from IR sensors to determine which parking spots are occupied or available
* Available parking spaces are listed on the LCD
* The system selects and recommends the nearest available parking spot
* The servo motor opens the gate to allow vehicle entry
* After a short delay, the gate automatically closes
* If all parking spots are occupied, the system displays a "Parking Full" message and keeps the gate closed

## 🅿️ Parking System

* IR1 → Parking Spot 1
* IR2 → Parking Spot 2
* IR3 → Parking Spot 3
* IR4 → Parking Spot 4

📷 Circuit Diagram
<img width="1016" height="671" alt="Ekran görüntüsü 2026-04-27 214845" src="https://github.com/user-attachments/assets/8ae21a17-5e84-45c5-a8a6-0c2a69a5ce52" />

