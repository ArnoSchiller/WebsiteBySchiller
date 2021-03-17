---
layout: post
title:  "Installation von Influx und Grafana auf dem Raspberry Pi"
date:   2021-03-01 00:00:05 +0100
author: Arno Schiller
tag: tutorial 
categories: esp32 mqtt  
permalink: /tutorials/esp32/mqtt-getting-started 
---

## Einleitung

## Erforderliches Equipment 
- Raspberry Pi (hier Modell 3B) mit [installiertem Image](https://www.raspberrypi.org/software/)
- ggf. Maus, Tastatur, Monitor und Internetzugang 

## Installation von Influxdb
---
### Schritt 1: Aktualisieren des Paketmanagers 

```bash
sudo apt update
sudo apt upgrade -y
```

### Schritt 2: Installation vorbereiten 
```bash
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
echo "deb https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
```

### Schritt 3: Influxdb installieren 
```bash
sudo apt update && sudo apt install -y influxdb
```

### Schritt 4: Influxdb als Service einrichten  
```bash
sudo systemctl unmask influxdb.service
sudo systemctl start influxdb
sudo systemctl enable influxdb.service
```

### Schritt 5: Einen Benutzer für Grafana erstellen
Kolsole starten: ```influx```
```bash
create database home
use home

create user grafana with password '<passwordhere>' with all privileges
grant all privileges on home to grafana

show users
```
Ausgabe: 
```bash 
user admin
---- -----
grafana true
```
Kolsole verlassen: ```exit```

## Installation von Grafana
---
### Schritt 1: Pakete dem Paket-Manager hinzufügen
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

## Schritt 2: Grafana installieren 
```bash
sudo apt update && sudo apt install -y grafana
```

## Schritt 3: Grafana als Service einrichten
```bash 
sudo systemctl unmask grafana-server.service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
```

## Schritt 4: Installation überprüfen 
Im Browser ```http://<ipaddress>:3000```

## Schritt 5: Grafana und Influx verlinken  
Im Browser ```http://<ipaddress>:3000```

unter Configuration -> Data sources eine neue Infux-Datenbank hinzufügen. 
- unter HTTP die URL anpassen: http://<ipaddress>:8086
- unter InfluxDB Details 
  - Datenbank auswählen: home
  - User hinterlegen: grafana
  - Passwort hinzufügen: -----
  - HTTP Method: GET
- save & test

## Beispielanwendung zur Datenakquise: Internet-Speed-Test
---
Quelle: https://simonhearne.com/2020/pi-speedtest-influx

### Schritt 1: Bibliotheken installieren 
```bash
sudo apt install -y python-pip
sudo pip install speedtest-cli influxdb
```

### Schritt 2: Skript für Speed-Test erzeugen
```rpi-speedtest-influx.py```

```python
#!/usr/bin/env python

import datetime
import speedtest
from influxdb import InfluxDBClient

# influx configuration - edit these
ifuser = "grafana"
ifpass = "<yourpassword>"
ifdb   = "home"
ifhost = "127.0.0.1"
ifport = 8086
measurement_name = "speedtest"

# take a timestamp for this measurement
time = datetime.datetime.utcnow()

# run a single-threaded speedtest using default server
s = speedtest.Speedtest()
s.get_best_server()
s.download(threads=1)
s.upload(threads=1)
res = s.results.dict()

# format the data as a single measurement for influx
body = [
    {
        "measurement": measurement_name,
        "time": time,
        "fields": {
            "download": res["download"],
            "upload": res["upload"],
            "ping": res["ping"]
        }
    }
]

# connect to influx
ifclient = InfluxDBClient(ifhost,ifport,ifuser,ifpass,ifdb)

# write the measurement
ifclient.write_points(body)
```

Als executable einstellen 
```chmod +x rpi-speedtest-influx.py```


### Schritt 3: Skript testen
```./rpi-speedtest-influx.py```

In Influx anzeigen lassen ```influx``` und dann
```influx
use home
select * from speedtest
```

### Schritt 4: Als Cronjob einrichten 
```crontab -e```
Alle 15 min: ```*/15 * * * * /home/pi/rpi-speedtest-influx.py```

### Schritt 5: Panel erstellen oder einfügen
https://simonhearne.com/2020/pi-speedtest-influx
https://gist.github.com/simonhearne/1b2b62e818fe6e097f8f8b2cbd6f2365

---
### Quellen:
- [Anleitung von Simon Hearne](https://simonhearne.com/2020/pi-influx-grafana)