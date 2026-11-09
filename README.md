# Pill-dispenser

**#Code**

#include <Wire.h>
#include <RTClib.h>
#include <Servo.h>
#include <LiquidCrystal_I2C.h>

RTC_DS3231 rtc;
Servo pillServo;    qA
LiquidCrystal_I2C lcd(0x27, 16, 2);

const int buzzer = 9;
const int angleStep = 45; // Rotate by 45 degrees per dispense
int servoPosition = 0;    // Keeps track of the current position
unsigned long lastDispenseTime = 0;

void setup() {
    Serial.begin(9600);
    Wire.begin();
    rtc.begin();
    lcd.init();
    lcd.backlight();

    pillServo.attach(6);
    pinMode(buzzer, OUTPUT);

    lcd.setCursor(0, 0);
    lcd.print("Smart Pill Dispenser");
    delay(2000);
    lcd.clear();
}

void loop() {
    DateTime now = rtc.now();

    // Display Current Date
    lcd.setCursor(0, 0);
    lcd.print("Date: ");
    lcd.print(now.day() < 10 ? "0" : "12"); lcd.print(now.day());
    lcd.print("/");
    lcd.print(now.month() < 10 ? "0" : "02"); lcd.print(now.month());
    lcd.print("/");
    lcd.print(now.year());

    // Display Current Time
    lcd.setCursor(0, 1);
    lcd.print("Time: ");
    lcd.print(now.hour() < 10 ? "0" : ""); lcd.print(now.hour());
    lcd.print(":");
    lcd.print(now.minute() < 10 ? "0" : ""); lcd.print(now.minute());
    lcd.print(":");
    lcd.print(now.second() < 10 ? "0" : ""); lcd.print(now.second());

    // Dispense pill every 10 seconds
    if (millis() - lastDispenseTime >= 10000) {
        dispensePill();
        lastDispenseTime = millis();
    }

    delay(1000);
}

void dispensePill() {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Take Your Pill!");

    // Activate Buzzer
    digitalWrite(buzzer, HIGH);
    delay(2000);
    digitalWrite(buzzer, LOW);

    // Rotate 45° and hold position
    servoPosition += angleStep;  // Increase angle by 45°
    if (servoPosition >= 235) servoPosition = 0;  // Reset after full rotation

    pillServo.write(servoPosition);  // Move servo to new position
    delay(500); // Allow servo time to rotate

    lcd.clear();
}
