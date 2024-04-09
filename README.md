# Embedded-S#include <ESP8266WiFi.h>
#include <PubSubClient.h>

// Replace with your WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// MQTT Broker details
const char* mqtt_server = "YOUR_MQTT_BROKER_IP";
const int mqtt_port = 1883;
const char* mqtt_username = "YOUR_MQTT_USERNAME";
const char* mqtt_password = "YOUR_MQTT_PASSWORD";

// MQTT topics
const char* soil_moisture_topic = "home/garden/soil_moisture";
const char* irrigation_control_topic = "home/garden/irrigation_control";

// Soil moisture threshold
const int moisture_threshold = 500; // Adjust according to your needs

// Pins
const int moisture_sensor_pin = A0;
const int irrigation_pin = D1;

// WiFi and MQTT client
WiFiClient espClient;
PubSubClient client(espClient);

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("WiFi connected - IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP8266Client", mqtt_username, mqtt_password)) {
      Serial.println("connected");
      client.subscribe(irrigation_control_topic);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  pinMode(irrigation_pin, OUTPUT);
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  int soil_moisture = analogRead(moisture_sensor_pin);
  Serial.print("Soil Moisture: ");
  Serial.println(soil_moisture);
  
  if (soil_moisture < moisture_threshold) {
    digitalWrite(irrigation_pin, HIGH); // Turn on irrigation
    client.publish(soil_moisture_topic, String(soil_moisture).c_str());
  } else {
    digitalWrite(irrigation_pin, LOW); // Turn off irrigation
  }

  delay(10000); // Adjust delay according to your needs
}

 Smart Irrigation System with IoT Integration
--------------------------------------------------------------------------------------------
CODE===>


