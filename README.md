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

## 📷 Circuit Diagram
<img width="1016" height="671" alt="Ekran görüntüsü 2026-04-27 214845" src="https://github.com/user-attachments/assets/8ae21a17-5e84-45c5-a8a6-0c2a69a5ce52" />

## 💻 Code
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>

LiquidCrystal_I2C lcd(0x20, 16, 2);
Servo gate;

// Ultrasonic sensor
const int trigPin = 7;
const int echoPin = 6;

// Servo
const int servoPin = 9;

// IR parking sensors
const int ir1 = 2;   // Park 1
const int ir2 = 3;   // Park 2
const int ir3 = 4;   // Park 3
const int ir4 = 5;   // Park 4

// Servo angles
const int gateClosed = 0;
const int gateOpen = 90;

// Distance limits
const float carDetectDistance = 12.0;
const float carClearDistance  = 20.0;

bool carProcessed = false;

// HIGH = empty, LOW = occupied
bool isEmpty(int irPin) {
  return digitalRead(irPin) == HIGH;
}

float measureDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(5);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000);

  if (duration == 0) {
    return 999.0;
  }

  float distance = duration * 0.0343 / 2.0;
  return distance;
}

int countEmptySpaces() {
  int count = 0;

  if (isEmpty(ir1)) count++;
  if (isEmpty(ir2)) count++;
  if (isEmpty(ir3)) count++;
  if (isEmpty(ir4)) count++;

  return count;
}

String emptySpacesText() {
  String text = "";

  if (isEmpty(ir1)) text += "1 ";
  if (isEmpty(ir2)) text += "2 ";
  if (isEmpty(ir3)) text += "3 ";
  if (isEmpty(ir4)) text += "4 ";

  if (text == "") text = "Yok";

  return text;
}

int nearestEmptySpace() {
  if (isEmpty(ir1)) return 1;
  if (isEmpty(ir2)) return 2;
  if (isEmpty(ir3)) return 3;
  if (isEmpty(ir4)) return 4;

  return 0;
}

void showMessage(String line1, String line2, int waitTime) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(line1);
  lcd.setCursor(0, 1);
  lcd.print(line2);
  delay(waitTime);
}

void setup() {
  lcd.init();
  lcd.backlight();

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(ir1, INPUT);
  pinMode(ir2, INPUT);
  pinMode(ir3, INPUT);
  pinMode(ir4, INPUT);

  gate.attach(servoPin);
  gate.write(gateClosed);

  showMessage("Akilli Otopark", "Sistem Hazir", 150);
}

void loop() {
  float distance = measureDistance();

  if (!carProcessed && distance <= carDetectDistance) {
    carProcessed = true;

    int emptyCount = countEmptySpaces();
    String emptyText = emptySpacesText();
    int nearestPark = nearestEmptySpace();

    showMessage("Hos geldiniz", "Arac algilandi", 150);

    if (emptyCount == 0) {
      showMessage("OTOPARK DOLU", "Kapi acilmaz", 200);
      gate.write(gateClosed);
    } 
    else {
      showMessage("Bos yer sayisi:", String(emptyCount), 150);
      showMessage("Bos yerler:", emptyText, 200);
      showMessage("Park ediniz:", String(nearestPark), 200);

      showMessage("Kapi aciliyor", "Ileri gidiniz", 150);
      gate.write(gateOpen);

      delay(2500);

      showMessage("Kapi kapaniyor", "Lutfen bekleyin", 150);
      gate.write(gateClosed);

      delay(150);
    }

    lcd.clear();
  }

  if (carProcessed && distance >= carClearDistance) {
    carProcessed = false;
    showMessage("Yeni arac icin", "Sistem hazir", 1500);
  }

  if (!carProcessed) {
    lcd.setCursor(0, 0);
    lcd.print("Arac bekleniyor ");
    lcd.setCursor(0, 1);
    lcd.print("Mesafe:");
    lcd.print(distance, 1);
    lcd.print("cm   ");
  }

  delay(300);
}
 ```
