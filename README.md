# Telescope-control-with-stellarium-esp32-controller
Hello guys, I'm making a project and I'm new to this, please can you help with the code. I'm happy with everything here, only it gets the wrong coordinates from Stellarium. It doesn't match at all. That is, my DS motors should make revolutions according to the coordinates. The only problem is that it needs to get the correct coordinates from Stellarium, at least help with this. I'll figure out the revolutions and so on, just help me improve the coordinates of the J2000


#include <WiFi.h>

// === WiFi parameters ===
const char* ssid = "SSID";
const char* password = "PASSWORD";

// Motor pins
int motorA1 = 14;

int motorA2 = 27;

int motorB1 = 12;

int motorB2 = 26;

// === TCP/IP settings ===
WiFiServer server(4030);

void setup() {
Serial.begin(115200);

// Connect to Wi-Fi
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("\n✅ Wi-Fi connected!");
Serial.print("ESP32 IP: ");
Serial.println(WiFi.localIP());

// Starting the server
server.begin();
Serial.println("✅ Server started. Waiting for Stellarium connection...");

// Setting up motors
pinMode(motorA1, OUTPUT);
pinMode(motorA2, OUTPUT);
pinMode(motorB1, OUTPUT);
pinMode(motorB2, OUTPUT);
}

void loop() {
WiFiClient client = server.available();
if (client) {
Serial.println("\n🔵 Stellarium connected!");

uint8_t dataBuffer[8];
int dataIndex = 0; 

while (client.connected()) {
 while (client.available() > 0) {
 uint8_t data = client.read();
 if (dataIndex < 8) {
 dataBuffer[dataIndex++] = data;
 }
 if (dataIndex == 8) {
 processCoordinates(dataBuffer);
 dataIndex = 0;
 }
 }
 }

 client.stop();
 Serial.println("🔴 Stellarium is disabled.");
 }
}

// === RA and DEC processing function ===
void processCoordinates(uint8_t* dataBuffer) {
 uint32_t ra_raw = (dataBuffer[0] << 24) | (dataBuffer[1] << 16) | (dataBuffer[2] << 8) | dataBuffer[3];
uint32_t dec_raw = (dataBuffer[4] << 24) | (dataBuffer[5] << 16) | (dataBuffer[6] << 8) | dataBuffer[7];

float ra = ra_raw / 10000000.0; // Convert to degrees
float dec = dec_raw / 10000000.0;

Serial.print("📡 Received coordinates: RA = ");
Serial.print(ra, 6);
Serial.print("°, DEC = ");
Serial.println(dec, 6);

// Calculate the number of revolutions
int turnsA = ra / 10;
int turnsB = dec / 10;

Serial.print("🔄 Motor A: ");
Serial.print(turnsA);
Serial.print(" turns, Motor B: ");
Serial.print(turnsB);
Serial.println(" turns");

rotateMotor(motorA1, motorA2, turnsA);
rotateMotor(motorB1, motorB2, turnsB);
}

// === Motor rotation function ===
void rotateMotor(int pin1, int pin2, int turns) {
if (turns <= 0) return;

for (int i = 0; i < turns; i++) {
digitalWrite(pin1, HIGH);
digitalWrite(pin2, LOW);
delay(1000); // Time of one revolution (select for your motors)
digitalWrite(pin1, LOW);
digitalWrite(pin2, LOW);
delay(500); // Pause between revolutions
}
}
