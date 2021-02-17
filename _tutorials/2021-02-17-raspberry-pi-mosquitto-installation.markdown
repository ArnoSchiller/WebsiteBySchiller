---
layout: post
title:  "Installation von Mosquitto auf dem Raspberry Pi"
date:   2021-02-17 00:00:05 +0100
author: Arno Schiller
tag: tutorial 
categories: raspberry-pi mqtt mosquitto 
permalink: /tutorials/raspberry-pi/mosquitto-installation 
---

## Einleitung
[MQTT](https://mqtt.org/) findet in vielen Anwendungsbereichen seinen Einsatz. Gerade in Hinblick auf IoT etablierte sich dieses Kommunikationsprotokoll als Standard. Das MQTT-Protokoll implementiert die [Publisher-Subscriber-Architektur](https://docs.microsoft.com/en-us/azure/architecture/patterns/publisher-subscriber). Um eine Kommunikation via MQTT herzustellen, wird ein MQTT-Broker benötigt, der die Nachrichten empfängt und diese an alle verbundenen Geräte weiterleitet. Weitere Informationen zu MQTT können dieser [Erklärung](https://www.opc-router.de/was-ist-mqtt/) entnommen werden. 

## Erforderliches Equipment 
- Raspberry Pi (hier Modell 3 B)
- ggf. Maus, Tastatur und Monitor  

## Installation von Mosquitto auf dem Raspberry Pi
---
### Schritt 1: Importieren des Mosquitto Repository
Um das entsprechende Paket installieren zu können, muss es importiert werden.
```bash
wget http://repo.mosquitto.org/debian/mosquitto-repo.gpg.key
sudo apt-key add mosquitto-repo.gpg.key
```

### Schritt 2: Hinzufügen der Pakete zum Paketmanager
Nachdem die Pakete importiert wurden, können diese dem apt-Paketmanager hinzugefügt werden. 
```bash
cd /etc/apt/sources.list.d/
```
Abhängig von der verwendeten Debian-Version sollte die entsprechende list-Datei in den Ordner, der die verfügbaren Pakete enthält, heruntergeladen werden. 
```bash
# in diesem Tutorial wird Raspian Buster verwendet 
sudo wget http://repo.mosquitto.org/debian/mosquitto-buster.list
# Alternativen:
sudo wget http://repo.mosquitto.org/debian/mosquitto-jessie.list
sudo wget http://repo.mosquitto.org/debian/mosquitto-stretch.list
```
Anschließend muss der Paketmanager aktualisiert werden, damit die Paketliste die neuen Paktete enthält.
```bash
sudo apt-get update
```

### Schritt 3: Installation von Mosquitto 
Mit dem folgenden Befehl kann überprüft werden, ob Mosquitto dem Paketmanager bekannt ist und welche Pakete verfügbar sind. 
```bash
apt-cache search mosquitto
```
Zuletzt kann Mosquitto installiert werden. 
```bash
sudo apt-get install mosquitto
```

### Schritt 4: Testen der Installation 
Mosquitto wird automatisch als Service eingerichtet und gestartet. Der Status des Service kann wie folgt abgerufen werden. 
```bash
systemctl status mosquitto
```
Bei Problemen kann der Service neu gestartet werden. 
```bash
systemctl restart mosquitto
```
Auch nach einem Restart des Raspberry Pi wird der Service von Mosquitto gestartet.


---
### Quellen:
- [Installation von Mosquitto unter Debian](https://mosquitto.org/blog/2013/01/mosquitto-debian-repository/)