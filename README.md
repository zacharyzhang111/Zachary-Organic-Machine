# Zachary-code
a programme of interactive design
I developed an Arduino program using the MPR121 capacitive touch sensor to control LED strips, a buzzer and a servo. Each touch channel was assigned a specific response: touching the mushrooms triggered lighting effects, sound feedback or mechanical movement. The program translated physical contact into real-time electronic responses, creating an interactive connection between visitors and the installation.
#include <Wire.h>
#include "Adafruit_MPR121.h"
#include <Adafruit_NeoPixel.h>
#include <Servo.h>

#ifndef _BV
#define _BV(bit) (1 << (bit))
#endif

// Settings
#define ledsStrip 10
#define stripOrder NEO_RGB + NEO_KHZ800
#define onColor Adafruit_NeoPixel::Color(255, 255, 255)
#define offColor Adafruit_NeoPixel::Color(0, 0, 0)
#define servoPin 9
#define servoAngle 90
#define servoAngle 90
#define servoDelay 100
#define buzzerPin 8
#define buzzerLoop 1

// Global Objects
Adafruit_MPR121 touchSensor = Adafruit_MPR121();
uint16_t lastTouch = 0;
uint16_t currentTouch = 0;

// LED Strips
Adafruit_NeoPixel strip2 = Adafruit_NeoPixel(ledsStrip, 2, stripOrder);
Adafruit_NeoPixel strip3 = Adafruit_NeoPixel(ledsStrip, 3, stripOrder);
Adafruit_NeoPixel strip4 = Adafruit_NeoPixel(ledsStrip, 4, stripOrder);
Adafruit_NeoPixel strip5 = Adafruit_NeoPixel(ledsStrip, 5, stripOrder);
Adafruit_NeoPixel strip6 = Adafruit_NeoPixel(ledsStrip, 6, stripOrder);

// Servo Object
Servo myServo;

// Buzzer Flag
bool buzzerActive = false;

// Functions

// LED Output
void stripOutput(Adafruit_NeoPixel &strip, uint32_t color) {
  for(uint8_t i=0; i<ledsStrip; i++) {
    strip.setPixelColor(i, color);
  }
  strip.show();
}

// Sound Output
void buzzerSound() {
  unsigned char i;

  for(i=0; i<80; i++) {
    digitalWrite(buzzerPin, HIGH);
    delay(1);
    digitalWrite(buzzerPin, LOW);
    delay(1);
  }

  for(i=0; i<100; i++) {
    digitalWrite(buzzerPin, HIGH);
    delay(2);
    digitalWrite(buzzerPin, LOW);
    delay(2);
  }
}

// Initialization
void setup() {

  // Serial Setup
  Serial.begin(9600);
  while (!Serial) {
    delay(10);
  }
  Serial.println(" MPR121 Control Program Start ");

  // Sensor Setup
  Serial.println("MPR121 Capacitive Touch sensor test");
  if (!touchSensor.begin(0x5A)) {
    Serial.println("MPR121 not found, check wiring?");
    while (1);
  }
  Serial.println("MPR121 found!");
  Serial.println("Running auto configuration.");
  touchSensor.setAutoconfig(true);
  Serial.println("MPR121 Initialization complete.");

  // LED Setup
  strip2.begin();  strip2.show();
  strip3.begin();  strip3.show();
  strip4.begin();  strip4.show();
  strip5.begin();  strip5.show();
  strip6.begin();  strip6.show();

  strip2.setBrightness(120);
  strip3.setBrightness(120);
  strip4.setBrightness(120);
  strip5.setBrightness(120);
  strip6.setBrightness(120);

  Serial.println("5 LED strips initialized, mapped to pin 2/3/4/5/6");

  // Servo Setup
  myServo.attach(servoPin);
  myServo.write(servoAngle);
  delay(servoDelay);
  Serial.print("Servo initialized, homed to ");
  Serial.print(servoAngle);
  Serial.println(" degree");

  // Buzzer Setup
  pinMode(buzzerPin, OUTPUT);
  digitalWrite(buzzerPin, LOW);
  Serial.println("Buzzer initialized, mapped to pin 8, triggered by Ch8 touch");

}

// Main Loop
void loop() {

  // Touch Scan
  currentTouch = touchSensor.touched();

  // Channel Scan
  for (uint8_t i=0; i<12; i++) {

    // Touch Press
    if ((currentTouch & _BV(i)) && !(lastTouch & _BV(i))) {
      Serial.print(i);
      Serial.println(" touched");

      // Channel Mapping
      switch(i) {
        case 2:
          stripOutput(strip2, onColor);
          Serial.println("Strip on pin 2 is fully lit");
          break;

        case 3:
          stripOutput(strip3, onColor);
          Serial.println("Strip on pin 3 is fully lit");
          break;

        case 4:
          stripOutput(strip4, onColor);
          Serial.println("Strip on pin 4 is fully lit");
          break;

        case 5:
          stripOutput(strip5, onColor);
          Serial.println("Strip on pin 5 is fully lit");
          break;

        case 6:
          stripOutput(strip6, onColor);
          Serial.println("Strip on pin 6 is fully lit");
          break;

        case 8:
          Serial.println("Ch8 triggered, buzzer activated");
          buzzerActive = true;
          break;

        case 9:
          Serial.println("Ch9 triggered, servo start rotating");
          myServo.write(0);
          delay(servoDelay);
          myServo.write(180);
          delay(servoDelay);
          myServo.write(servoAngle);
          delay(servoDelay);
          Serial.println("Servo action completed, homed");
          break;

        default:
          break;
      }
    }

    // Touch Release
    if (!(currentTouch & _BV(i)) && (lastTouch & _BV(i))) {
      Serial.print(i);
      Serial.println(" released");

      // Channel Mapping
      switch(i) {
        case 2:
          stripOutput(strip2, offColor);
          Serial.println("Strip on pin 2 is fully off");
          break;

        case 3:
          stripOutput(strip3, offColor);
          Serial.println("Strip on pin 3 is fully off");
          break;

        case 4:
          stripOutput(strip4, offColor);
          Serial.println("Strip on pin 4 is fully off");
          break;

        case 5:
          stripOutput(strip5, offColor);
          Serial.println("Strip on pin 5 is fully off");
          break;

        case 6:
          stripOutput(strip6, offColor);
          Serial.println("Strip on pin 6 is fully off");
          break;

        case 8:
          Serial.println("Ch8 released, buzzer stopped");
          buzzerActive = false;
          digitalWrite(buzzerPin, LOW);
          break;

        default:
          break;
      }
    }
  }

  // Status Update
  lastTouch = currentTouch;

  // Sound Control
  if(buzzerActive) {
    for(uint8_t loopCount=0; loopCount<buzzerLoop; loopCount++) {
      if(!buzzerActive) break;
      buzzerSound();
    }
  }

  // Debug Mode
  return;

  // Debug Output
  Serial.print("\t\t\t\t\t\t\t\t\t\t\t\t\t 0x");
  Serial.println(touchSensor.touched(), HEX);

  Serial.print("Filt: ");
  for (uint8_t i=0; i<12; i++) {
    Serial.print(touchSensor.filteredData(i));
    Serial.print("\t");
  }
  Serial.println();

  Serial.print("Base: ");
  for (uint8_t i=0; i<12; i++) {
    Serial.print(touchSensor.baselineData(i));
    Serial.print("\t");
  }
  Serial.println();

  delay(100);
}
