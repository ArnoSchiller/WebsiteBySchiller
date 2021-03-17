---
layout: post
title:  "Installation tesseract"
date:   2021-03-03 00:00:05 +0100
author: Arno Schiller
tag: tutorial 
categories: jetson-nano ocr  
permalink: /tutorials/jetson-nano/tesseract 
---

## Einleitung
Die optische Zeichenerkennung (Optical Character Recognition - OCR) spielt im Bereich des maschinellem Sehen eine große Rolle. OCR wird verwendet, um aus Rastergrafiken Informationen zu extrahieren. So können beispielsweise PDF-Dateien eingelesen und dessen Text extrahiert werden, anschließend kann der Text editiert werden. 
[Tesseract](https://github.com/tesseract-ocr/tesseract) stellt eine Open-Source OCR Engine zur Verfügung. Im Folgenden wird diese auf dem Jetson Nano installiert und anhand von Beispielbildern getestet. 

## Erforderliches Equipment 
- NVIDIA Jetson Nano (hier 2GB Developer Kit) mit [installiertem Image](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-2gb-devkit#write)
- ggf. Maus, Tastatur, Monitor und Internetzugang 

## Installation der Tesseract Engine und PyTesseract
---
### Schritt 1: Aktualisieren des Paketmanagers 

```bash
sudo apt update
```

### Schritt 2: Installation von Tesseract für Linux
Weitere Informationen können beispielsweise dem [Wiki von Ubuntu](https://wiki.ubuntuusers.de/tesseract-ocr/) entnommen werden. 
```bash
sudo apt-get install tesseract-ocr 
```
Die Installation von Tesseract fügt die Engine automatisch den Systempfaden hinzu. Die Installation kann somit überprüft werden, indem ```tesseract``` in die Konsole eingegeben wird. Bei korrekter Installation erfolgt die Ausgabe der kurzen Dukumentation der Engine. 

### Schritt 3: Installation entsprechender Sprachpakete 
```bash
sudo apt-get install tesseract-ocr-eng sudo apt-get install tesseract-ocr-deu
```
Um alle Sprachpakete zu installieren kann der folgende Befehl ausgeführt werden.
```bash
sudo apt-get install tesseract-ocr-all
```

### Schritt 4: Installation der Python Schnittstelle
Falls nicht vorhanden:
```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py
```

```bash
python3 -m pip install pytesseract
```

### Schritt 4: Testen der Python Schnittstelle
Aus dem Verzeichnis [tests/data](https://github.com/madmaze/pytesseract/tree/master/tests/data) können die Testbilder heruntergeladen werden. Anschließend können Elemente aus dem [Usage](https://pypi.org/project/pytesseract/#description) in ein Python-Skript übernommen werden. Das Ausführen des Skripts führt die entsprechende Anwendung aus.  



### Quellen:
- [Dokumentation von Tesseract OCR](https://tesseract-ocr.github.io/)
- [Dokumentation von PyTesseract](https://pypi.org/project/pytesseract/)