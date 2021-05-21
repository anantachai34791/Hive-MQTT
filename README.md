#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT12.h>

const char* ssid = "Mi Phone";
const char* password = "0880954763";
const char* mqtt_server = "broker.mqttdashboard.com";

WiFiClient  espClient;
PubSubClient client(espClient);
DHT12 dht12;

long now = millis();
long lastMeasure = 0;

void setup_wifi() {
    delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("WiFi connected - ESP IP address: ");
  Serial.println(WiFi.localIP());
}
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");

    if (client.connect("ESP32Client")) {
      Serial.println("connected");  
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}
void setup()
{
    Serial.begin(115200);
    setup_wifi();
    delay(10);
    dht12.begin();
  client.setServer(mqtt_server, 1883);
  }
 void loop() {

  if (!client.connected()) {
    reconnect();
  }
  if(!client.loop())
    client.connect("ESP32Client");

  now = millis();
  if (now - lastMeasure > 30000) {
    lastMeasure = now;
    float h = dht12.readHumidity();
    float t = dht12.readTemperature();
  
    if (isnan(h) || isnan(t))  {
      Serial.println("Failed to read from DHT sensor!");
      return;
    }

    float hic = dht12.computeHeatIndex(t, h, false);
    static char temperatureTemp[7];
    dtostrf(hic, 6, 2, temperatureTemp);
    
    static char humidityTemp[7];
    dtostrf(h, 6, 2, humidityTemp);

    client.publish("room/temperature", temperatureTemp);
    client.publish("room/humidity", humidityTemp);
    
    Serial.print("Humidity: ");
    Serial.print(h);
    Serial.print(" %\t Temperature: ");
    Serial.print(t);
    Serial.print(" *C ");
    Serial.print(hic);
    Serial.println(" *C ");

  }
}
