#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <SoftwareSerial.h>
#include <ThingSpeak.h>
#include <LiquidCrystal.h>
#include <dht.h>

#define DHTPIN 2  // Pin for DHT11 sensor
#define DHTTYPE DHT11  // DHT11 sensor type
dht DHT;

#define SEALEVELPRESSURE_HPA (1013.25) // Define the sea level pressure

// WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// ThingSpeak details
unsigned long myChannelNumber = YOUR_CHANNEL_NUMBER;  // Your ThingSpeak channel number
const char* myWriteAPIKey = "YOUR_WRITE_API_KEY";  // Your ThingSpeak write API key

// Initialize the ESP8266 client
WiFiClient client;

// Initialize LCD
LiquidCrystal lcd(3, 4, 5, 6, 7, 8); // RS, E, D4, D5, D6, D7

void setup() {
  Serial.begin(9600);
  delay(10);
  
  // Initialize the LCD
  lcd.begin(16, 2);
  lcd.print("Air Pollution");
  lcd.setCursor(0, 1);
  lcd.print("Monitoring");
  
  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  ThingSpeak.begin(client);  // Initialize ThingSpeak
}

void loop() {
  float mq2Value = analogRead(A0); // Read MQ2 sensor value
  float mq4Value = analogRead(A1); // Read MQ4 sensor value
  float mq135Value = analogRead(A2); // Read MQ135 sensor value
  float mq7Value = analogRead(A3); // Read MQ7 sensor value
  
  int chk = DHT.read11(DHTPIN); // Read DHT11 sensor values
  float temperature = DHT.temperature;
  float humidity = DHT.humidity;

  // Display sensor readings on LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Temp:");
  lcd.print(temperature);
  lcd.print("C ");
  lcd.setCursor(0, 1);
  lcd.print("Humidity:");
  lcd.print(humidity);
  lcd.print("%");

  // Update ThingSpeak fields with sensor readings
  ThingSpeak.setField(1, mq2Value);
  ThingSpeak.setField(2, mq4Value);
  ThingSpeak.setField(3, mq135Value);
  ThingSpeak.setField(4, mq7Value);
  ThingSpeak.setField(5, temperature);
  ThingSpeak.setField(6, humidity);

  // Write to ThingSpeak
  int writeSuccess = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);

  if (writeSuccess == 200) {
    Serial.println("Data sent to ThingSpeak");
  } else {
    Serial.println("Error in sending data to ThingSpeak");
  }

  delay(20000); // Delay for 20 seconds before sending the next set of data
}
