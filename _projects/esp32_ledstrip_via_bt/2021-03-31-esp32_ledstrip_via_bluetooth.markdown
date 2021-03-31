---
layout: post
title:  "LED-Strip via Bluetooth ansteuern"
date:   2021-03-31 00:00:05 +0100
author: Arno Schiller
tag: project 
categories: esp32 led-strip ws2812b  
permalink: /projects/ledstrip-via-bluetooth
---

## Einleitung
Immer häufiger finden LED-Strips Anwendung in Projekten, ob nun ein PC mit einem transparenten Gehäuse ausgestattet wird und eine Lichtannimation zu sehen ist, oder ob die Glasvitrine mit ein wenig Licht aufgepeppt wird, es ist alles möglich. Um in die Thematik einzusteigen, wird in diesem Projekt ein einfaches LED-Band via Bluetooth angesteuert. Dabei wird das Ganze vom Testing bis zum Resultat beschrieben. 

## Erforderliches Equipment 
Das hier aufgeführte Equipment bezieht sich auf das gesamte Projekt, den einzelnen Abschnitten ist zu entnehmen, welche Teile wann zum Einsatz kommen.
- ESP32 (Dev Module) [z.B. bei Amazon](https://www.amazon.de/gp/product/B074RGW2VQ/ref=ppx_yo_dt_b_asin_title_o04_s00?ie=UTF8&psc=1)
- drei Transistoren (IRLZ34N) [z.B. bei Amazon](https://www.amazon.de/gp/product/B01FUSRARW/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
- RGB LED-Strip 
- Adapter Stromkabel auf Buchse [z.B. bei Amazon](https://www.amazon.de/gp/product/B00TMCDSXS/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) 

## Version 1: Ansteuerung des LED-Strip via Bluetooth
---
### Vorbereitung der Hardware
Das verwendete LED-Band ist ein 12V-RGB-LED-Band, die LEDs sind nicht einzeln ansteuerbar, sondern das gesamte LED-Band wird mit der gleichen Farbe angesteuert. Es verfügt über insgesamt vier Pins, eines für die 12V Spannungszufuhr und jeweils eine Verbindung für die drei Farben. Um das LED-Band anzusteuern, wird pro Farbe ein Transistor benötigt. Dieser wird verwendet, um die Spannungsversorgung der LEDs zu regeln. Jede LED des LED-Strip besteht aus drei einzelnen LEDs, die jeweils eine Farbe repräsentieren. Bei genaueren Hinsehen ist dies deutlich zu erkennen. 
Zuerst müssen die Komponenten korrekt verbunden werden. Das rechte Bein, der Emitter, wird mit Ground verbunden, das linke Bein, der Kollektor, wird mit dem entsprechenden Pin des Boards verbunden. Das mittlere Bein, die Basis, wird mit dem RGB-Strip verbunden. Das RGB-Band benötogt eine 12V Spannungsversorgung. Die meiten LED-Strips, die mit einer Fernbedienung ausgestattet sind, beinhalten meist bereits eine Spannungsversorgung. Um diese in die Schaltung zu integrieren. Die Pins wurden wie folgt verwendet: 

|ESP32-Pin|  Farbe  |
|:-------:|---------|
|   G25   |  blau   |
|   G26   |  rot    |
|   G27   |  grün   |

Die fertige Schaltung ist in der folgenden Abbildung dargestellt. 

![Verkabelung des ESP32 mit dem LED-Strip](./esp32_ledstrip_via_bt/images/esp32_rgb_ledstrip_wiring.jpg)

### Erste Ansteuerung des LED-Strip
Die Ansteuerung des LED-Bands mit dem ESP32 unterscheidet sich von der Ansteuerung mittels Arduino-Board. Für die Steuerung der LEDs durch die Pins wird die Pulsweitenmodulation (PWM) verwendet. Dieses [Tutorial](https://techexplorations.com/guides/esp32/begin/pwm/) thematisiert die Steuerung einer LED mit dem ESP32, wobei auf die Unterschiede zu einem Arduino eingeganen wird. 
Bevor die eigentliche Programmierung beginnen kann, müssen einige Variablen definiert werden. Zudem muss jeder Farbe ein Pin zugewiesen werden, sowie ein Kanal für die PWM.  
```c 
// define LED color pins 
#define RED_LED         26
#define GREEN_LED       27
#define BLUE_LED        25

// define PWM channel for every color
#define RED_CHANNEL     0
#define GREEN_CHANNEL   1
#define BLUE_CHANNEL    2
```
Nachdem alle relevanten Variablen definiert wurden, kann die ```setup```-Funktion implementiert werden. In dieser wird für jeden Farbkanal die PWM mit dem entsprechenden Pin in Verbindung gebracht und anschließend initialisiert. Weitere Informationen können der [Dokumentation der LED Control](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/ledc.html) entnommen werden. 
```c
void setup() {
  // ESP 32 
  ledcAttachPin(RED_LED,    RED_CHANNEL);
  ledcAttachPin(GREEN_LED,  GREEN_CHANNEL);
  ledcAttachPin(BLUE_LED,   BLUE_CHANNEL);
  
  ledcSetup(RED_CHANNEL,    4000, 8);
  ledcSetup(GREEN_CHANNEL,  4000, 8);
  ledcSetup(BLUE_CHANNEL,   4000, 8);  
}
```

Ist das Setup erstmal beendet, ist die Einstellung einer Farbe ganz einfach. Die Funktion ```ledcWrite``` übernimmt diese Aufgabe. Dazu muss nur der entsprechende PWM-Kanal der Farbe und der neue Farbwert übergeben werden. Die Farbwerte können hierbei zwischen 0 und 255 liegen. Um das Testskript ein wenig spannender zu gestalten, wird eine Fade-Funktion implementiert. In einer Schleife wird hierbei die Intensität einer Farbe konstant erhöht, dabei wird eine kurze Verzögerung eingebunden, um die Änderung besser sichtbar zu machen. Eine zweite Schleife wird verwendet, um die Intensität der Farbe wieder zu dimmen, bis diese den Wert 0 erreicht. Damit jede Farbe überprüft werden kann, wird die ```fade_color```-Funktion dreimal aufgerufen und dabei der PWM-Kanal variiert. Die ```fade_color```- und die ```loop```-Funktion sind im folgenden Code-Element zu finden.   
```c
void loop() {
  fade_color(RED_CHANNEL);
  fade_color(GREEN_CHANNEL);
  fade_color(BLUE_CHANNEL);
}

void fade_color(int channel){
  for(int i = 0; i < 256; i++){
    ledcWrite(channel, i);
    delay(10);
  }
  for(int i = 255; i >= 0; i--){
    ledcWrite(channel, i);
    delay(10);
  }
}
```
Das vollständige Skript ist ebenfalls unter [Github](https://github.com/ArnoSchiller/InternetOfThings/blob/main/ESP32_DevBoard/LED_strip_test/LED_strip_test.ino) zu finden. Wenn alles korrekt verbunden wurde und es keine Probleme beim Upload des Skripts gab, sollten alle drei Farben nacheinander mittels eingeblendet und ausgeblendet werden. 

### Coming soon: Ansteuerung via Bluetooth

### Quellen:
- [Dokumentation LED Control von ESPRESSIF](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/ledc.html)
