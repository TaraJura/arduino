#include <Ethernet.h>
#include <SPI.h>

// Ethernet settings
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
IPAddress ip(192, 168, 0, 175); // Your Arduino's IP
EthernetClient client;

// HC-SR04 Pins
const int trigPin = 9;
const int echoPin = 8;

unsigned long previousMillis = 0; // will store last time a request was sent
const long interval = 3000; // interval at which to send requests (milliseconds)

void setup() {
  Serial.begin(9600); // Start serial communication for debugging
  Ethernet.begin(mac, ip); // Initialize Ethernet with your MAC and IP
  pinMode(trigPin, OUTPUT); // Initialize HC-SR04 pins
  pinMode(echoPin, INPUT);
  delay(1000); // Allow time for Ethernet shield to initialize
}

void loop() {
  unsigned long currentMillis = millis();

  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis; // save the last time a request was sent

    // Measure distance
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
    long duration = pulseIn(echoPin, HIGH);
    int distance = duration * 0.034 / 2;

    // Prepare JSON data
    char jsonBuffer[50];
    sprintf(jsonBuffer, "{\"distance\":%d}", distance);

    Serial.print("Distance: ");
    Serial.println(distance);
    Serial.println("Attempting to connect to server...");

    unsigned long startAttemptTime = millis();
    while (!client.connect(IPAddress(192, 168, 0, 180), 3005) && millis() - startAttemptTime < 5000) {
        // Attempt to connect for up to 5 seconds
    }

    if (client.connected()) {
      Serial.println("Connected to server, sending data...");

      // Send HTTP POST request with distance data
      client.println("POST /arduino/receiver HTTP/1.1"); // Change to your specific path
      client.println("Host: 192.168.0.180"); // Server host
      client.println("Content-Type: application/json");
      client.println("Connection: close");
      client.print("Content-Length: ");
      client.println(strlen(jsonBuffer)); // Length of the data
      client.println();
      client.println(jsonBuffer);

      delay(500); // Wait for server to process request

      Serial.println("Request sent. Keeping connection open for a short time.");
      
      // Wait for server response or timeout
      long timeoutStart = millis();
      while (client.connected() && millis() - timeoutStart < 1000) {
        if (client.available()) {
          char c = client.read();
          Serial.write(c);
        }
      }

      Serial.println("\nDisconnecting...");
      client.stop();
    } else {
      Serial.println("Failed to connect.");
    }
  }
}
