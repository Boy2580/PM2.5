# PM2.5
// project PM2.5
#include <Adafruit_NeoPixel.h>
#include <SoftwareSerial.h>
//=========== NEOPIXEL ==============
#define PIXELS_PER_SEGMENT 11  // Number of LEDs in each Segment
#define PIXELS_DIGITS 3        // Number of connected Digits
#define PIXELS_PIN 7           // GPIO Pin
bool Debug = false;
int LITUP_STRIPS_COLOR[] = { 100, 10, 0 };
//=========== SENSOR ==============
SoftwareSerial mySerial(2, 3);  // TX, RX
unsigned int pm1 = 0;
unsigned int pm2_5 = 0;
unsigned int pm10 = 0;
int Previous_pm2_5 = -1;
Adafruit_NeoPixel strip = Adafruit_NeoPixel(PIXELS_PER_SEGMENT * 7 * PIXELS_DIGITS, PIXELS_PIN, NEO_GRB + NEO_KHZ800);
byte segments[7] = {
  0b0000001,  // Segment g;
  0b0000100,  // Segment e;
  0b0001000,  // Segment d;
  0b0010000,  // Segment c;
  0b0100000,  // Segment b;
  0b1000000,  // Segment a;
  0b0000010   // Segment f;
}
byte digits[10] = {
  0b1111110,  // 0;
  0b0110000,  // 1;
  0b1101101,  // 2;
  0b1111001,  // 3;
  0b0110011,  // 4;
  0b1011011,  // 5;
  0b1011111,  // 6;
  0b1110000,  // 7;
  0b1111111,  // 8;
  0b1111011   // 9;
};
void clearDisplay() {
  for (int i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, strip.Color(0, 0, 0));
 }
  strip.show();
}
void setup() {
  strip.begin();
  Serial.begin(9600);
  while (!Serial) ;
  mySerial.begin(9600);
}
void loop() {
  int index = 0;
  char value;
  char previousValue;
  if (Debug == false) {
    while (mySerial.available()) {
      value = mySerial.read();
      if ((index == 0 && value != 0x42) || (index == 1 && value != 0x4d)) {
        Serial.println("Cannot find the data header.");
        break;
      }
      if (index == 4 || index == 6 || index == 8 || index == 10 || index == 12 || index == 14) {
        previousValue = value;
      } 
else if (index == 5) {
        pm1 = 256 * previousValue + value;
        Serial.print("{ ");
        Serial.print("\"pm1\": ");
        Serial.print(pm1);
        Serial.print(" ug/m3");
        Serial.print(", ");
      }
 else if (index == 7) {
        pm2_5 = 256 * previousValue + value;
        Serial.print("\"pm2_5\": ");
        Serial.print(pm2_5);
        Serial.print(" ug/m3");
        Serial.print(", ");
      } 
else if (index == 9) {
        pm10 = 256 * previousValue + value;
        Serial.print("\"pm10\": ");
        Serial.print(pm10);
        Serial.print(" ug/m3");
      } else if (index > 15) {
        break;
      }
      index++;
    }
    while (mySerial.available()) mySerial.read();
    Serial.println(" }");
    delay(1000);
  }
  if (Debug == false && Previous_pm2_5 != pm2_5) {
    Previous_pm2_5 = pm2_5;
    clearDisplay();
    strip.show();
    int hundreds = floor(pm2_5 / 100);
    int tens = floor((pm2_5 % 100) / 10);
    int units = pm2_5 % 10;
    if (hundreds != 0) {
      writeDigit(0, hundreds, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
    }
    if (tens != 0) {
      writeDigit(1, tens, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
    }
    writeDigit(2, units, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
  }
  //=========== TEST ==============
  else if (Debug == true) {
    for (int count = 0; count <= 9; count++) {
      clearDisplay();
      strip.show();
      writeDigit(0, count, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
      writeDigit(1, count, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
      writeDigit(2, count, LITUP_STRIPS_COLOR[0], LITUP_STRIPS_COLOR[1], LITUP_STRIPS_COLOR[2]);
      strip.show();
      delay(1000);
      clearDisplay();
    }
  }
  strip.show();
}

void writeDigit(int index, int val, int R, int G, int B) {
  byte digit = digits[val];
  for (int i = 6; i >= 0; i--) {
    int offset = index * (PIXELS_PER_SEGMENT * 7) + i * PIXELS_PER_SEGMENT;
    uint32_t color;
    if (digit & 0x01 != 0) {
      if (val == 1) color = strip.Color(R, G, B);
      if (val == 2) color = strip.Color(R, G, B);
      if (val == 3) color = strip.Color(R, G, B);
      if (val == 4) color = strip.Color(R, G, B);
      if (val == 5) color = strip.Color(R, G, B);
      if (val == 6) color = strip.Color(R, G, B);
      if (val == 7) color = strip.Color(R, G, B);
      if (val == 8) color = strip.Color(R, G, B);
      if (val == 9) color = strip.Color(R, G, B);
      if (val == 0) color = strip.Color(R, G, B);
    } else
      color = strip.Color(0, 0, 0);
    for (int j = offset; j < offset + PIXELS_PER_SEGMENT; j++) {
      strip.setPixelColor(j, color);
    }
    digit = digit >> 1;
  }
}
void writeSegment(int index, int val) {
  byte seg = segments[val];
  for (int i = 6; i >= 0; i--) {
    int offset = index * (PIXELS_PER_SEGMENT * 7) + i * PIXELS_PER_SEGMENT;
    uint32_t color;
    if (seg & 0x01 != 0) {
      if (val == 0) color = strip.Color(255, 0, 0);
      if (val == 1) color = strip.Color(255, 0, 0);
      if (val == 2) color = strip.Color(255, 0, 0);
      if (val == 3) color = strip.Color(255, 0, 0);
      if (val == 4) color = strip.Color(255, 0, 0);
      if (val == 5) color = strip.Color(255, 0, 0);
      if (val == 6) color = strip.Color(255, 0, 0);
    } else
      color = strip.Color(0, 0, 0);
    for (int j = offset; j < offset + PIXELS_PER_SEGMENT; j++) {
      strip.setPixelColor(j, color);
    }
    seg = seg >> 1;
  }
}
