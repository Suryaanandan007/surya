


#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);   // LCD 16x2 I2C
SoftwareSerial BT(2, 3);              // RX=2, TX=3 for Bluetooth module

#define motorPin 7     // Motor control pin
#define ledPin 8       // LED indicator pin
#define soilPin A0     // Soil (and rain) sensor analog input
#define gndPin 6       // Extra GND pin

bool autoMode = true;
int soilValue = 0;

void setup() {
  pinMode(motorPin, OUTPUT);
  pinMode(ledPin, OUTPUT);
  pinMode(gndPin, OUTPUT);

  digitalWrite(motorPin, LOW);
  digitalWrite(ledPin, LOW);
  digitalWrite(gndPin, LOW);   // Use D6 as GND

  Serial.begin(9600);
  BT.begin(9600);

  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Smart Irrigation");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Read soil (and rain) sensor value
  soilValue = analogRead(soilPin);

  // --- Bluetooth Commands ---
  if (BT.available()) {
    char cmd = BT.read();

    if (cmd == 'A' || cmd == 'a') {
      autoMode = true;
      BT.println("AUTO MODE ENABLED");
    } 
    else if (cmd == 'M' || cmd == 'm') {
      autoMode = false;
      BT.println("MANUAL MODE ENABLED");
    } 
    else if (!autoMode) {
      if (cmd == '1') {
        digitalWrite(motorPin, HIGH);
        digitalWrite(ledPin, HIGH);
        BT.println("PUMP ON (Manual)");
      } 
      else if (cmd == '0') {
        digitalWrite(motorPin, LOW);
        digitalWrite(ledPin, LOW);
        BT.println("PUMP OFF (Manual)");
      }
    }
  }

  // --- Auto Mode Operation ---
  if (autoMode) {
    if (soilValue > 500) {
      digitalWrite(motorPin, HIGH);
      digitalWrite(ledPin, HIGH);
    } else {
      digitalWrite(motorPin, LOW);
      digitalWrite(ledPin, LOW);
    }
  }

  // --- LCD Display ---
  lcd.setCursor(0, 0);
  lcd.print("Soil:");
  lcd.print(soilValue);
  lcd.print("   "); // clears old digits

  lcd.setCursor(0, 1);
  lcd.print("Rain: Auto ");
  lcd.print(autoMode ? "A" : "M");

  delay(1000);
}

