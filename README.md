# multiplesensors_thingspeak


![Capture](https://user-images.githubusercontent.com/44220596/105461808-b9bced80-5cb3-11eb-9c7f-51eb36c60d23.PNG)

Components Required -

NodeMCU ESP8266
Soil Moisture Sensor Module
Water Pump Module
Relay Module
DHT11
Connecting Wires

Circuit Diagram -

![Circuit-Diagram-for-IoT-based-Smart-Irrigation-System-using-Soil-Moisture-Sensor-and-ESP8266-NodeMCU](https://user-images.githubusercontent.com/44220596/105461696-95611100-5cb3-11eb-9ffe-132122e9ebed.png)



Programming ESP8266 NodeMCU for Automatic Irrigation System
For programming the ESP8266 NodeMCU module, only the DHT11 sensor library is used as external library. The moisture sensor gives analog output which can be read through the ESP8266 NodeMCU analog pin A0. Since the NodeMCU cannot give output voltage greater than 3.3V from its GPIO so we are using a relay module to drive the 5V motor pump. Also the Moisture sensor and DHT11 sensor is powered from external 5V power supply.

Complete code with a working video is given at the end of this tutorial, here we are explaining the program to understand the working flow of the project.

Start with including necessary library.

#include <DHT.h>
#include <ESP8266WiFi.h>
Since we are using the ThingSpeak Server, the API Key is necessary in order to communicate with server. To know how we can get API Key from ThingSpeak you can visit previous article on Live Temperature and Humidity Monitoring on ThingSpeak.


 
String apiKey = "X5AQ445IKMBYW31H
const char* server = "api.thingspeak.com"; 
The next Step is to write the Wi-Fi credentials such as SSID and Password.

const char *ssid =  "CircuitDigest";     
const char *pass =  "xxxxxxxxxxx"; 
Define the DHT Sensor Pin where the DHT is connected and Choose the DHT type.

#define DHTPIN D3          
DHT dht(DHTPIN, DHT11);
The moisture sensor output is connected to Pin A0 of ESP8266 NodeMCU. And the motor pin is connected to D0 of NodeMCU.

const int moisturePin = A0;
const int motorPin = D0;
We will be using millis() function to send the data after every defined interval of time here it is 10 seconds. The delay() is avoided since it stops the program for a defined delay where microcontroller cannot do other tasks. Learn more about the difference between delay() and millis() here.

unsigned long interval = 10000;
unsigned long previousMillis = 0;
Set motor pin as output, and turn off the motor initially. Start the DHT11 sensor reading.

pinMode(motorPin, OUTPUT);
digitalWrite(motorPin, LOW); // keep motor off initally
dht.begin();
Try to connect Wi-Fi with given SSID and Password and wait for the Wi-Fi to be connected and if connected then go to next steps.

WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
}
Define the current time of starting the program and save it in a variable to compare it with the elapsed time.

unsigned long currentMillis = millis();
Read temperature and humidity data and save them into variables.

float h = dht.readHumidity();
float t = dht.readTemperature();
If DHT is connected and the ESP8266 NodeMCU is able to read the readings then proceed to next step or return from here to check again.

if (isnan(h) || isnan(t))
  {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
Read the moisture reading from sensor and print the reading.

moisturePercentage = ( 100.00 - ( (analogRead(moisturePin) / 1023.00) * 100.00 ) );
  Serial.print("Soil Moisture is  = ");
  Serial.print(moisturePercentage);
  Serial.println("%");
If the moisture reading is in between the required soil moisture range then keep the pump off or if it goes beyond the required moisture then turn the pump ON. 

if (moisturePercentage < 50) {
    digitalWrite(motorPin, HIGH);
  }
   if (moisturePercentage > 50 && moisturePercentage < 55) {
    digitalWrite(motorPin, HIGH);
  }
 if (moisturePercentage > 56) {
    digitalWrite(motorPin, LOW);
  }
Now after every 10 seconds call the sendThingspeak() function to send the moisture, temperature and humidity data to ThingSpeak server.

  if ((unsigned long)(currentMillis - previousMillis) >= interval) {
    sendThingspeak();
    previousMillis = millis();
    client.stop();
  }
In the sendThingspeak() function we check if the system is connected to server and if yes then we prepare a string where moisture, temperature, humidity reading is written and this string will be sent to ThingSpeak server along with API key and server address.

if (client.connect(server, 80))
    {
      String postStr = apiKey;
      postStr += "&field1=";
      postStr += String(moisturePercentage);
      postStr += "&field2=";
      postStr += String(t);
      postStr += "&field3=";
      postStr += String(h);      
      postStr += "\r\n\r\n";
Finally the data is sent to ThingSpeak server using client.print() function which contains API key, server address and the string which is prepared in previous step.

client.print("POST /update HTTP/1.1\n");
      client.print("Host: api.thingspeak.com\n");
      client.print("Connection: close\n");
      client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
      client.print("Content-Type: application/x-www-form-urlencoded\n");
      client.print("Content-Length: ");
      client.print(postStr.length());
      client.print("\n\n");
      client.print(postStr);
