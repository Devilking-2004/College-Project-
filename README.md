# College-Project-

Embedded C Code for IOT Based Low Cost Monitoring of Water Quality in Real TIme

Code:
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ThingSpeak.h>
// Sensor pins
#define pH_PIN A0
#define TURBIDITY_PIN A0 // if using a multiplexer, change this
#define TEMP_PIN D4 // DS18B20 uses digital pin
#include <OneWire.h>
#include <DallasTemperature.h>
// DS18B20 Setup
OneWire oneWire(TEMP_PIN);
DallasTemperature sensors(&oneWire);
// WiFi Credentials
const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";
// ThingSpeak Settings
unsigned long myChannelNumber = YOUR_CHANNEL_ID;
const char * myWriteAPIKey = "YOUR_API_KEY";
WiFiClient client;
void setup() {
Serial.begin(115200);
WiFi.begin(ssid, password);
// Wait for WiFi to connect
while (WiFi.status() != WL_CONNECTED) {
delay(1000);
Serial.println("Connecting to WiFi...");
}
Serial.println("Connected to WiFi");
// Start sensors
sensors.begin();
// Initialize ThingSpeak
ThingSpeak.begin(client);
}
void loop() {
// Read temperature
sensors.requestTemperatures();
float tempC = sensors.getTempCByIndex(0);
// Read pH sensor
int pH_value = analogRead(pH_PIN);
float voltage = pH_value * (3.3 / 1024.0); // Adjust for 3.3V ADC
float pH = 7 + ((2.5 - voltage) / 0.18); // Example formula; calibrate for real sensor
// Read turbidity sensor
int turb_value = analogRead(TURBIDITY_PIN);
float turb_voltage = turb_value * (3.3 / 1024.0);
float turbidity = (turb_voltage > 2.5) ? 0 : (3000 * (2.5 - turb_voltage)); // Example NTU scale
// Print readings
Serial.print("Temperature: "); Serial.println(tempC);
Serial.print("pH: "); Serial.println(pH);
Serial.print("Turbidity: "); Serial.println(turbidity);
// Send data to ThingSpeak
ThingSpeak.setField(1, tempC);
ThingSpeak.setField(2, pH);
ThingSpeak.setField(3, turbidity);
int status = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
if (status == 200) {
Serial.println("Data sent to ThingSpeak successfully");
} else {
Serial.print("Error sending data. HTTP error code: ");
Serial.println(status);
}
delay(15000); // ThingSpeak allows data every 15 seconds
}
