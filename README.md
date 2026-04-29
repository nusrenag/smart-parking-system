# 🚗 Smart Parking System

This project is a smart parking management solution developed using Arduino. The system automatically detects incoming vehicles, assesses the occupancy rate of parking spaces in the parking lot, and directs the driver to the most suitable available parking space.

The rapid increase in the number of vehicles today makes finding a parking space increasingly difficult. Such smart solutions enable more efficient use of existing parking spaces.

## 🎯 Project Objective

The main objective of this project is to make parking lot use more practical and efficient. To this end, the system automates parking lot entry, allowing vehicles to enter quickly. It also monitors parking lot occupancy in real-time and shows drivers the nearest available parking space. This prevents unnecessary waiting, minimizes time loss, and makes the parking process much more convenient.

## 🔧 Components Used

All components used in the project have been carefully selected to ensure the system operates smoothly and intelligently:

* Arduino Uno

It is at the heart of the system. It receives and processes data from all sensors and controls other components accordingly.

* Ultrasonic Sensor (HC-SR04)

Used to determine if there is a vehicle at the parking lot entrance. It measures the distance by sending sound waves and detects the vehicle.

* Infrared (IR) Sensors (4 units)

Placed in each parking space. Thanks to these sensors, it instantly detects whether the parking spaces are occupied or empty.

* Servo Motor (SG90)

Controls the parking lot entrance gate. It opens the gate when a vehicle is detected and closes it again when the process is complete.

* 16x2 I2C LCD Screen

Used to provide information to the driver. Important information such as empty parking spaces and directions are displayed here.

* Breadboard and Jumper Cables

Used for circuit connections.

## ⚙️ How Does the System Work?

The system works according to the following steps:

1. When the vehicle approaches the entrance, it is detected by the ultrasonic sensor.
2. A welcome message is displayed to the driver on the LCD screen.
3. The system analyzes the status of parking spaces by receiving data from the IR sensors.
4. Empty parking spaces are listed on the LCD screen.
5. The system determines the nearest empty parking space.
6. The driver is shown which parking space to go to.
7. The servo motor starts and opens the entrance gate.
8. The gate closes automatically after the vehicle enters.
9. If all parking spaces are full, the system does not allow entry.



## 📷 Circuit Diagram for Proteus
<img width="1016" height="671" alt="Ekran görüntüsü 2026-04-27 214845" src="https://github.com/user-attachments/assets/8ae21a17-5e84-45c5-a8a6-0c2a69a5ce52" />

Additionally, if you'd like to run the project in Proteus, you can download and run the file named *New Projects.pdsprj* from my page.

## 💻 Code for Circuit Diagram (Proteus)
After downloading the circuit diagram in Proteus, the code used is given below. You can also use it by downloading the *sketch_apr24a.ino* and *sketch_apr24a.ino.hex* files from my page.
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
## 🔬 Real System Application
This project was first designed and tested in the Proteus simulation environment. The circuit connections, sensor responses, and control algorithm were examined in detail in the virtual environment to ensure the system was functioning correctly.

After the simulation phase was completed, the same structure was transferred to the physical environment using real hardware components. The system was built using an Arduino Uno, ultrasonic sensor, IR sensors, servo motor, and I2C LCD.

The actual application was developed based on the results obtained and connections established in the simulation. This ensured compatibility between the simulation and the physical system, resulting in the project's reliable operation.

## 📸 Project Images
<img width="1600" height="1200" alt="5848ee2b-5c28-4694-8cc2-993159596018" src="https://github.com/user-attachments/assets/82fde1dc-5b81-4aef-ada3-2a742f46337b" />


<img width="1600" height="1200" alt="eac50e7c-2518-4d9d-ab35-90025ffc8355" src="https://github.com/user-attachments/assets/f752b793-abeb-4af7-b3b3-592828bc746b" />

## 💡 **Observations and Important Notes**

* The simulation and the real system worked together to a large extent. The circuit structure established in the Proteus environment was implemented seamlessly in the physical hardware with the same logic.

* Some `delay()` times used in the simulation needed to be readjusted in the real system. This is because the components operate under ideal conditions in Proteus. In real life, however, sensor response time, servo motor movement speed, connection quality, power fluctuations, and environmental factors can directly affect the system. Therefore, delay times were increased to achieve more stable and fluid operation.

* A significant difference was also observed regarding the I2C LCD address. While the 0x20 address was used for the LCD in the simulation, this address did not work in the Arduino. As a result of the tests, it was determined that the LCD worked correctly with the 0x27 address.

* The detection status of IR sensors can be changed using the sensitivity potentiometer on the sensor. Therefore, each sensor must be calibrated individually to detect whether parking spaces are occupied or empty.

* In the actual system, it is critical that all modules share a common GND line. If the Arduino, LCD, servo motor, ultrasonic sensor, and IR sensors are not connected to the same ground line, the system may operate unstably.

* Because the mechanical placement of the servo motor can behave differently in the real system than in the simulation, the door angles were determined by testing on the physical setup. The `gateClosed` value was set to `20` for the door to remain parallel to the surface in the closed position, and the `gateOpen` value was set to `110` for the door to open sufficiently. Therefore, instead of the standard `0` and `90`, the servo angles used in the code were `20` and `110` to match the mechanical position in the real system.

## 💻 Code for Real System 
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
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
const int gateClosed = 20;
const int gateOpen = 110;

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

  showMessage("Akilli Otopark", "Sistem Hazir", 1500);
}

void loop() {
  float distance = measureDistance();

  if (!carProcessed && distance <= carDetectDistance) {
    carProcessed = true;

    int emptyCount = countEmptySpaces();
    String emptyText = emptySpacesText();
    int nearestPark = nearestEmptySpace();

    showMessage("Hos geldiniz", "Arac algilandi", 1500);

    if (emptyCount == 0) {
      showMessage("OTOPARK DOLU", "Kapi acilmaz", 2000);
      gate.write(gateClosed);
    } 
    else {
      showMessage("Bos yer sayisi:", String(emptyCount), 1500);
      showMessage("Bos yerler:", emptyText, 2000);
      showMessage("Park ediniz:", String(nearestPark), 2000);

      showMessage("Kapi aciliyor", "Ileri gidiniz", 1500);
      gate.write(gateOpen);

      delay(2500);

      showMessage("Kapi kapaniyor", "Lutfen bekleyin", 1500);
      gate.write(gateClosed);

      delay(1500);
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
## 📌 Note

This project has been successfully developed for educational purposes and aims to help understand the fundamental logic of real-world systems.
