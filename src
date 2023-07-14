#include <LiquidCrystal_I2C.h>
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
#include <ESP8266HTTPClient.h>

// Initialize the LCD display
LiquidCrystal_I2C lcd(0x27, 16, 2);

char auth[] = "qGzXefmqXWEWSeyIaVViBlDPr3sDQvSA"; // Enter your Auth token
char ssid[] = "Gaurav"; // Enter your WIFI name
char pass[] = "01234567"; // Enter your WIFI password

BlynkTimer timer;
bool Relay = 0;

// Define component pins
#define sensor A0
#define motor D3
#define buzzer D4
#define LED D5

// ThingSpeak configuration
const char* server = "api.thingspeak.com";
String apiKey = "CNI4G3YPKFYM7RGG";

WiFiClient client;

// Variables
int alcoholPercentage;

void setup() {
  Serial.begin(9600);
  pinMode(motor, OUTPUT);
  digitalWrite(motor, HIGH);
  lcd.init();
  lcd.backlight();
  pinMode(buzzer, OUTPUT);
  digitalWrite(buzzer, LOW);
  pinMode(LED, OUTPUT);
  digitalWrite(LED, LOW);
  Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);

  lcd.setCursor(1, 0);
  lcd.print("System Loading");
  for (int a = 0; a <= 15; a++) {
    lcd.setCursor(a, 1);
    lcd.print(".");
    delay(500);
  }
  lcd.clear();

  // Call the function
  timer.setInterval(100L, alcoholSensor);
  timer.setInterval(15000L, sendToThingSpeak); // Send data to ThingSpeak every 15 seconds
}

// Get the button value
BLYNK_WRITE(V1) {
  Relay = param.asInt();

  if (Relay == 1) {
    digitalWrite(motor, LOW);
    lcd.setCursor(0, 1);
    lcd.print("Engine is ON ");
  } else {
    digitalWrite(motor, HIGH);
    lcd.setCursor(0, 1);
    lcd.print("Engine is OFF");
  }
}

void alcoholSensor() {
  int value = analogRead(sensor);
  value = map(value, 0, 1024, 0, 1024);

  alcoholPercentage = value;
  
  Blynk.virtualWrite(V2, value);
  
  lcd.setCursor(0, 0);
  lcd.print("Alcohol: ");
  lcd.print(value);
  lcd.print(" ");
  
  sendAlcoholnotification();
}

void sendAlcoholnotification() {
  if (alcoholPercentage > 450) {
    Blynk.logEvent("notify", "Alcohol more than 450");
    digitalWrite(buzzer, HIGH);
    digitalWrite(LED, HIGH);
  }
  else {
    digitalWrite(buzzer, LOW);
    digitalWrite(LED, LOW);
  }
}

void sendToThingSpeak() {
  if (client.connect(server, 80)) {
    String postStr = apiKey; 
    postStr += "&field1="; 
    postStr += String(alcoholPercentage); 
    postStr += "&field2=";
    postStr += String(alcoholPercentage); 
    postStr += "&field3="; 
    postStr += String(alcoholPercentage); 
    postStr += "\r\n";
  
    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);
    
    Serial.println("Data Sent to ThingSpeak");
  }
  
  client.stop();
}

void loop() {
  Blynk.run();
  timer.run();
}
