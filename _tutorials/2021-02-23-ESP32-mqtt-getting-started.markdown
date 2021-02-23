---
layout: post
title:  "Einführung in MQTT auf dem ESP32 Development Board"
date:   2021-02-23 00:00:05 +0100
author: Arno Schiller
tag: tutorial 
categories: esp32 mqtt  
permalink: /tutorials/esp32/mqtt-getting-started 
---

## Einleitung
[MQTT](https://mqtt.org/) hat sich im Bereich des Internet of Things (IoT) als Standard-Kommunikationsprotokoll etabliert. Die ESP-Boards haben eine WLAN-Schnittstelle, wodurch sich diese für den Einsatz im IoT eignen. Im Folgenden wird erläutert, wie ein ESP-Board mit dem WLAN verbunden werden kann und wie eine Kommunikation mittels MQTT aufgebaut wird. 

## Erforderliches Equipment 
- MQTT Broker ([beispielsweise auf dem Raspberry Pi](/tutorials/raspberry-pi/mosquitto-installation))
- [ESP32 Development Board](https://www.az-delivery.de/products/esp32-developmentboard) oder ein vergleichbares ESP-Board
- [Arduino IDE](https://www.arduino.cc/en/Main.Software)

*Hinweis:* Um das ESP32 Dev Module mittels Arduino zu programmieren, muss das Board zuvor dem Programm hinzugefügt werden. Unter ```Werkzeuge > Board: "..." > Boardverwalter...``` können neue Boards hinzugefügt werden. Für dieses Tutorial wird das Paket [esp32 von Espressif Systems](https://github.com/espressif/arduino-esp32) in der Version 1.0.4 verwendet. 

## Verbindung des ESP-Boards mit dem WLAN
---
### Schritt 1: Einbinden der erforderlichen Pakete
Damit eine MQTT-Kommunikation herstellen zu können, muss vorerst eine WLAN-Verbindung vorhanden sein. Um das ESP-Board mit dem Netzwerk zu verbinden, wird die [WiFi-Bibliothek](https://www.arduino.cc/en/Reference/WiFi) (verwendete Version: 1.5.0) benötigt. Eine neue Bibliothek kann unter ```Werkzeuge > Bibliotheken verwalten...``` eingebunden werden. Nachdem das Paket hinzugefügt wurde, muss es in dem Projekt eingebunden werden, das kann durch den folgenden Code realisiert werden.
```c
#include <WiFi.h>
```

### Schritt 2: Zugangsdaten definieren
Um mit einem WLAN-Netzwerk eine Verbindung aufzubauen, werden dessen Zugangsdaten benötigt. Diese sind passend zum entsprechenden Netzwerk im Folgenden zu ersetzen.
```c
const char* ssid     = "***********";
const char* password = "***********";
```

### Schritt 3: Verbindung zum WLAN herstellen
Die Verbindung mit dem WLAN-Netzwerk wird in eine Funktion ausgelagert, diese sollte in der späteren setup-Funktion aufgerufen werden. Der Verbindungsaufbau wird währenddessen durch die serielle Schnittstelle dokumentiert. Ist die Verbindung erfolgreich, so wird die IP-Adresse des ESP-Boards im lokalen Netzwerk ausgegeben. Weitere Informationen und Beispiele zu der WiFi-Bibliothek können der [Dokumentation](https://www.arduino.cc/en/Reference/WiFi) entnommen werden. 
```c
void setup_wifi(){
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  // connection to WIFi Network 
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}
```


## Kommunikation mit einem MQTT-Broker
---
### Schritt 1: Einbinden der erforderlichen Pakete
Für die MQTT-Schnittstelle unter Arduino sind unterschiedliche Bibliotheken verfügbar. Im Folgenden wird die Bibliothek [PubSubClient von Nick O’Leary](https://pubsubclient.knolleary.net/) (Version 2.8.0) verwendet. Die Bibliothek kann ebenfalls unter ```Werkzeuge > Bibliotheken verwalten...``` eingebunden werden. Anschließend kann diese eingebunden werden. 
```c
#include <PubSubClient.h>
```

### Schritt 2: Zugangsdaten und benötigte Variablen definieren  
Um die Verbindung mit einem MQTT-Broker herstellen zu können, muss dessen Adresse definiert werden. Diese wird als ```mqtt_server``` gespeichert und kann eine IP-Adresse oder eine Domain sein. Zudem wird der entsprechende Port benötigt, dieser wird (z.B. unter [Mosquitto](/tutorials/raspberry-pi/mosquitto-installation)) meist auf ```1883``` festgesetzt. Unter MQTT werden bestimmte Topics verwendet, auf denen Nachrichten ausgetauscht werden, die Topics für das Publizieren und das Empfangen von Nachrichten werden folgend definiert. Zudem wird ein Identifier für den MQTT-Client benötigt, im Folgenden wird diesbezüglich ein ```DEVICE_NAME``` festgelegt. 

Um den MQTT-Client mit einem Broker zu verbinden, muss dieser zuvor initialisiert werden. Für dessen Erzeugung wird der Zugang zum WLAN benötigt, dieser wird über den WiFiClient realisiert. Weitere Informationen können der [Dokumentation](https://pubsubclient.knolleary.net/api) entnommen werden. 

```c
// MQTT access data 
const char* mqtt_server = "********";
#define mqtt_port ****

#define MQTT_PUBLISH_TOPIC "/iot/test"
#define MQTT_SUBSCRIBE_TOPIC "/iot/test"

const String DEVICE_NAME = "mqttTester";

WiFiClient wifiClient;
PubSubClient client(wifiClient);
```

### Schritt 3: Eine Callback-Funktion definieren 
Wie bereits erwähnt, können Nachrichten sowohl empfangen als auch gesendet werden. Um Nachrichten zu empfangen, wird eine bestimmte Topic subscribt. Wird eine Nachricht an diese Topic gesendet, wird ein Trigger ausgelöst, der die sogenannte Callback-Funktion ausführt. Die Callback-Funktion kann optional überschrieben werden. Im Folgenden wird eine Callback-Funktion implementiert, die eine neue Nachricht über die serielle Schnittstelle ankündigt und dessen Topic und dessen Inhalt ausgibt.  

```c
void callback(char* topic, byte *payload, unsigned int length) {
    Serial.println("--- new message from broker ---");
    Serial.print("Topic:    ");
    Serial.print(topic);
    Serial.print("Message:  ");
    Serial.write(payload, length);
    Serial.println();
}
```

### Schritt 4: Verbindung zum MQTT-Broker herstellen 
Damit der MQTT-Client verwendet werden kann, muss der Zielserver definiert werden. Diesbezüglich wird dem Client die Adresse und der Port des Servers übergeben. Zudem kann die Callback-Funktion überschrieben werden. In diesem Fall wird die zuvor definierte Funktion, die neue Nachrichten auf der seriellen Schnittstelle ausgibt, als Callback verwendet. 

Anschließend kann die Verbindung zum Server aufgebaut werden, das übernimmt die Funktion ```client.connect()```, die den festgelegten Identifier übergeben bekommt. Diese Funktion hat weitere optionale Übergabeparameter wie den Benutzernamen und ein Passwort, zudem kann über diese dem Gerät ein letzter Wille zugewiesen werden. Dabei handelt es sich um eine Nachricht, die gesendet wird, sobald die Verbindung zu dem Gerät für einige Sekunden unterbrochen ist. Weitere Informationen können der [Dokumentation](https://pubsubclient.knolleary.net/api) entnommen werden. 

In der erstellten Funktion wird der Verbindungsaufbau solange wiederholt, bis die Verbindung erfolgreich hergestellt werden konnte. Wenn keine Verbindung hergestellt werden kann, wird dieses über die serielle Schnittstelle kommuniziert und nach einigen Sekunden der Vorgang wiederholt. Bei einer erfolgreichen Verbindung wird mittels ```client.publish()``` die Nachricht gesendet, dass sich ein neues Gerät verbunden hat. Anschließend wird mittels ```client.subscribe()``` die Subscription auf ein bestimmtes Topic eingeleitet. Damit diese Funktion ausgeführt wird, muss in der loop-Funktion der Befehl ```client.loop()``` aufgerufen werden. Eine erste Kommunikation wird im unteren Bereich ausführlich beschrieben.

```c
void setup_mqtt(){
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  
  // Loop until we're (re)connected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    // Attempt to connect
    if (client.connect(DEVICE_NAME.c_str())) {
      Serial.println("connected");
      //Once connected, publish an announcement...
      client.publish(MQTT_PUBLISH_TOPIC, "new device connected");
      // ... and resubscribe
      client.subscribe(MQTT_SUBSCRIBE_TOPIC);
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}
```

## Eine erste Kommunikation 
Bisher wurde gezeigt, wie das ESP32 Development Board mit dem WLAN verbunden und eine Verbindung zu einem MQTT-Broker hergestellt werden kann. Der vollständige Code für dieses Tutorial ist unter [GitHub](https://github.com/ArnoSchiller/InternetOfThings/blob/main/ESP32_DevBoard/MQTT_Test/MQTT_Test.ino) verfügbar. Die Zugangsdaten für das WLAN-Netzwerk und den MQTT-Broker wurden aus Sicherheitsgründen in einer externen Datei gespeichert. Der Inhalt dieser Datei muss individuell angepasst werden. Eine Musterdatei ist ebenfalls unter [GitHub] gelistet. Entweder sollte die überarbeitete Datei als ```accessData.h``` in dem gleichen Projekt wie die ```.ino```-Datei gespeichert werden, oder die Zeile ```#include "accessData.h" ``` wird entfernt und die Zugangsdaten direkt in der Arduino-Datei definiert. 

In der loop-Funktion sendet der Client eine Nachricht mit dem Inhalt ```test message``` an den Broker. Programme wie  [MQTT.fx](https://mqttfx.jensd.de/) ermöglichen den Aufbau einer Kommunikation mit dem verwendeten MQTT-Broker. Wird die entsprechende Topic (in diesem Fall ```/iot/test```) subscribt, so können die Nachrichten des ESP-Boards durt gelesen werden. 

Neben der Publikation von Nachrichten wird ebenfalls eine Subscription durch das ESP-Board eingerichtet. Wenn auf der Topic, auf der publiziert wird, ebenfalls gelesen wird, so werden die gesendeten Nachrichten auch auf dem Gerät empfangen und folgende Ausgabe auf der seriellen Schnittstelle geprintet. 
```
--- new message from broker ---
Topic:    /iot/test
Message:  test message

``` 
Auch über MQTT.fx können Nachrichten an den Broker gesendet werden. Wird dabei die Topic verwendet, auf der das Board subscribt, werden diese ebenfalls auf der seriellen Schnittstelle ausgegeben. Dabei fällt auf, dass die Nachrichten im gleichen Zeitabstand gesendet als auch gelesen werden. Das bedeutet, wenn externe Nachrichten gesendet werden, werden diese zeitverzögert ausgegeben oder es kommt zu einem Verlust der Nachrichten.

*Hinweis:* Das ESP Board und andere Mikrocontroller sind nicht für die parallele Verarbeitung ausgelegt. Deswegen wird empfohlen, das Board primär als Sender oder als Empfänger zu verwenden, da es bei einer erhöhten Datenrate zu Verzögerungen oder Verlusten kommen kann. 

---
### Quellen:
- [WiFi-Bibliothek unter Arduino](https://www.arduino.cc/en/Reference/WiFi)
- [Arduino Client für MQTT](https://pubsubclient.knolleary.net/)