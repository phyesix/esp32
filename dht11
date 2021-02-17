#include "esp32-hal.h"
#include "DHT.h"
#include <WiFi.h>
#include <HTTPClient.h>
#include <Wire.h>

const char* ssid     = ""; // WIFI NETWORK NAME
const char* password = ""; // WIFI NETWORK PASSWORD
const char* serverName = ""; // POST URL

String apiKeyValue = ""; // RANDOM STRING FOR SECURITY
String sensorName = "DHT11";
//String sensorLocation = "kitchen";
//String sensorLocation = "living";
String sensorLocation = "office";

#define DHTPIN 4
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

const uint64_t OneSecond = 1000000LL;
const uint64_t OneMinute = 60000000LL;          // 6000000 uS = 1 minute
const uint64_t MinutesInAnHour = 60LL;    // 60 min = 1 hour 
const uint64_t SleepTimeMicroseconds = OneMinute * MinutesInAnHour * 2LL; 
//const uint64_t SleepTimeMicroseconds = 5LL * OneSecond; // 15 s interval

RTC_DATA_ATTR int bootCount = 0;
#define ONBOARD_LED  2

void setup() {
  esp_sleep_enable_timer_wakeup(SleepTimeMicroseconds);
  pinMode (ONBOARD_LED, OUTPUT);

  Serial.begin(9600);

  WiFi.begin(ssid, password);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED) { 
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("");
  Serial.print("Connected to WiFi network with IP Address: ");
  Serial.println(WiFi.localIP());
  
  dht.begin();
}

void loop() {
  digitalWrite(ONBOARD_LED, HIGH);
  delay(50);
  digitalWrite(ONBOARD_LED, LOW);
  delay(50);

  
  if(WiFi.status()== WL_CONNECTED) {
    HTTPClient http;

    http.begin(serverName);
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    float h = dht.readHumidity();
    float t = dht.readTemperature();
  
    if (isnan(h) || isnan(t)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      //return;
    }
  
    float hic = dht.computeHeatIndex(t, h, false);
    
    String httpRequestData = "api_key=" + apiKeyValue + "&sensor=" + sensorName
                          + "&location=" + sensorLocation + "&humidity=" + String(h)
                          + "&temp=" + String(t) + "&heatindex=" + String(hic) + "";
    Serial.print("httpRequestData: ");
    Serial.println(httpRequestData);
    
    int httpResponseCode = http.POST(httpRequestData);

    while(httpResponseCode<0) { 
      delay(500);
      Serial.print(".");
      httpResponseCode = http.POST(httpRequestData);
      Serial.print(".");
      Serial.println(httpResponseCode);
    }

    http.end();
  } else {
    Serial.println("WiFi Disconnected");
  }

  digitalWrite(ONBOARD_LED, HIGH);
  delay(50);
  digitalWrite(ONBOARD_LED, LOW);
  delay(50);
  digitalWrite(ONBOARD_LED, HIGH);
  delay(50);
  digitalWrite(ONBOARD_LED, LOW);
  delay(50);
  Serial.println("Going to sleep now");
  esp_deep_sleep_start();
  //delay(2000);  
}
