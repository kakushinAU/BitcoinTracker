# Bitcoin Price tracker using ESP32 smd chip

### Important links

Schematic: https://github.com/kakushinAU/BitcoinTracker/blob/main/ESP32_Bitcoin_Tracker_Schematic.pdf

Paper cutout pattern (A4 size): https://github.com/kakushinAU/BitcoinTracker/blob/main/Bitcoin_Tracker_Case-A4.pdf
Paper cutout pattern (Letter size): https://github.com/kakushinAU/BitcoinTracker/blob/main/Bitcoin_Tracker_Case-Letter.pdf

MegaCoreX board manager URL: https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

Pololu USB AVR Programmer v2.1: https://www.pololu.com/product/3172

### Example code

```
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

#include <NTPClient.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#define NTP_OFFSET  39600 // In seconds 39600 = +11 hours
#define NTP_INTERVAL 60 * 1000 // In miliseconds
#define NTP_ADDRESS  "1.asia.pool.ntp.org"
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, NTP_ADDRESS, NTP_OFFSET, NTP_INTERVAL);
Adafruit_SSD1306 display(128, 64, &Wire, 4);

const char* ssid = "mywifi";
const char* password = "mywifipassword";
String response = "";
String currentPrice = "";
String formattedTime = "";

volatile unsigned long lastPriceCheck = (millis() - 10000);

//JSON document
DynamicJsonDocument doc(2048);

void setup(void) {
  Serial.begin(9600);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextColor(WHITE);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.print("CONNECTING TO: ");
  display.println(ssid);
  display.println();
  display.display();
  delay(500);
  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    display.print(".");
    display.display();
  }
  Serial.print("WiFi connected with IP: ");
  Serial.println(WiFi.localIP());
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("WiFi connected ");
  display.println();
  display.print("IP: ");
  display.println(WiFi.localIP());
  display.display();
  delay(1000);
  // Start time client and display
  timeClient.begin();
}

String updateTime()
{
  timeClient.update();
  String formattedTime = timeClient.getFormattedTime();
  return (formattedTime);
}

String checkBitcoinPrice()
{
  HTTPClient http;
  String request = "http://api.coindesk.com/v1/bpi/currentprice/BTC.json";
  http.begin(request);
  http.GET();
  response = http.getString();
  DeserializationError error = deserializeJson(doc, response);
  if (error) {
    Serial.print(F("deserializeJson() failed: "));
    Serial.println(error.f_str());
    return ("error");
  }
  http.end();
  return (doc["bpi"]["USD"]["rate"].as<char*>());
}

void loop() {
  String formattedTime = updateTime();
  if ((lastPriceCheck + 10000) < millis()) {
    currentPrice = checkBitcoinPrice();
    lastPriceCheck = millis();
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.print(formattedTime);
  display.println("  BITCOIN USD");
  display.setTextSize(2);
  display.setCursor(0, 24);
  display.print("   BTC");
  display.setCursor(0, 48);
  display.print("$");
  display.println(currentPrice);
  display.display();   // write the buffer to the display
  delay(100);
}

```
