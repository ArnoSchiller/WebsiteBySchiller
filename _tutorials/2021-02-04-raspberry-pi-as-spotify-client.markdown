---
layout: post
title:  "Raspberry Pi als Spotify Client"
date:   2021-02-04 15:50:05 +0100
author: Arno Schiller
tag: tutorial 
categories: iot raspberry-pi spotify-client 
permalink: /tutorials/raspberry-pi/spotify-client 
---

## Einleitung
In diesem Tutorial wird erläutert, wie der Raspberry Pi als Spotify Client eingerichtet und verwendet werden kann. Der Spotify Client auf dem Pi ist headless, das bedeutet, dass keine Benutzeroberfläche vorhanden ist. Stattdessen kann der Client auf dem Pi durch andere Geräte wie das Smartphone gesteuert werden. 

## Erforderliches Equipment 
- **Spotify Premium** 
- **Raspberry Pi** (2 (+ WiFi-Dongle), 3 oder 4) mit einem [installierten Image](https://www.raspberrypi.org/software/) (Raspbian) und einer Stromzufuhr - verwendet wurde der Raspberry Pi 3B v1.2 
- ggf. Maus und Tastertur

## Einrichten des Raspberry Pi Spotify Connect Device 
---
### Schritt 1: Raspberry Pi updaten
Um sicher zu gehen, dass das Betriebssystem auf dem neusten Stand ist und somit Spotify so einfach wie möglich installiert werden kann, wird zuerst die Paketliste geupdatet und anschließend werde alle installierten Pakete aktualisiert. 
```
sudo apt update
sudo apt upgrade
```

### Schritt 2: Erforderliche Pakete installieren 
Als nächstes müssen alle Pakete installiert werden, die für die Installation von *raspotify* unter Raspbian. Dies sind *curl* und  *apt-transport-https*. 
```
sudo apt install -y apt-transport-https curl
```

### Schritt 3: GPG-Key und Repository von *raspotify* hinzufügen
Ohne den GPG-Key kann der apt Paketmanager die Dateien, die aus dem Repositpry von *raspotify* geladen werden, nicht verifizieren. Zudem muss auch das Repository eingebunden werden.   
```
curl -sSL https://dtcooper.github.io/raspotify/key.asc | sudo apt-key add -v -
echo 'deb https://dtcooper.github.io/raspotify raspotify main' | sudo tee /etc/apt/sources.list.d/raspotify.list
```

### Schritt 4: Installation von *raspotify* 
Nachdem das Repository dem Raspberry Pi hinzugefügt wurde, kann das Paket *raspotify* installiert werden. Dieses Paket kümmert sich darum, den Raspberry Pi als Spotify Connect Device nutzen zu können.
Aufgrund dessen, dass ein neues Paket hinzugefügt wurde, muss der Paketmanager zuvor erneut aktualisiert werden. Ohne die Aktualisierung weiß der Paketmanager nicht, was in dem Repository enthalten ist. 
```
sudo apt update
sudo apt install raspotify
```

### Schritt 5: Verwendung des Raspberry Pi als Spotify Connect Device
Nachdem *raspotify* installiert wurde, kann der Raspberry Pi als Spotify Connect Device verwendet werden. Die Software sollte automatisch gestartet werden und somit für eine Verbindung zur Verfügung stehen. 
Für diesen Zweck kann Spotify auf einem mobilen Gerät oder Computer gestartet werden und unter **Mit einem Gerät verbinden** der Raspberry Pi ausgewählt werden. 

## Konfiguration der Spotify Connect Software 
---
Die Spotify Connect Software kann hinsichtlich verschiedener Parameter konfiguriert werden. Die Konfigurationsdatei kann wie folgt aufgerufen werden. 
```
sudo nano /etc/default/raspotify
```
Im Folgenden werden einige Möglichkeiten der Konfiguration dargelegt. 
### Möglichkeit 1: Name des Device
Der Name des Device kann modifiziert werden. Dieser Name wird unter Spotify Connect angezeigt. Die folgende Zeile ändert den Namen zu "raspotify BySchiller".  
```
DEVICE_NAME="raspotify BySchiller"
```

### Möglichkeit 2: Übertragungsqualität 
Zudem kann die Qualität der Übertragung beeinflusst werden. Die Übertragungsgeschwindigkeit kann mittels Bitrate festgelegt werden. Es sind drei Qualitätsstufen möglich:
- geringe Qualität: ```BITRATE="96"```
- standardmäßige Qualität: ```BITRATE="160"```
- höchste Qualität: ```BITRATE="320"```

### Möglichkeit 3: Audio-Output
Sind mehrere Audio-Outputs des Raspberry Pi vorhanden, kann zwischen diesen gewählt werden. Besteht beispielsweise eine HDMI- und eine AUX-Verbindung, so kann der AUX-Ausgang wie folgt eingestellt werden.  
```
OPTIONS="--device hw:1"
```
Die Nummerierung der Ausgänge beginnend bei 0 kann sich von der verwendeten Zuweisung unterscheiden. Alternativ kann der Output des Raspberry Pi über [Systemeinstellungen](https://www.raspberrypi.org/documentation/configuration/audio-config.md) adaptiert werden.

### Möglichkeit 4: Kontoinformationen
Der Username und das Passwort des verwendeten Spotify Accounts können wie folgt eingespeichert werden.
```
OPTIONS="--username <USERNAME> --password <PASSWORD>"
``` 


Zusätzlich zu den genannten Optionen können weitere Konfigurationen vorgenommen werden. Die [Dokumentation](https://dtcooper.github.io/raspotify/) von *raspotify* liefert mehr Informationen.


---
### Quellen:
- [Dokumentation von raspotify](https://dtcooper.github.io/raspotify/)