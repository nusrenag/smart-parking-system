# smart-parking-system
Arduino-based smart parking system (Ultrasonic + IR + Servo + I2C LCD)

//CODE
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
