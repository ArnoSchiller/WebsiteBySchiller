---
layout: post
title:  "Installation von PyTorch auf dem NVIDIA Jetson Nano"
date:   2021-02-11 00:00:05 +0100
author: Arno Schiller
tag: tutorial 
categories: machine-learning jetson-nano pytorch 
permalink: /tutorials/jetson-nano/pytorch-setup 
---

## Einleitung
Der NVIDIA Jetson Nano ist eine preiswerte Hardware für die Entwicklung im Bereich KI und Robotik. Aus diesem Grund wird im Folgenden die Konfiguration des NVIDIA Jetson Nano thematisiert. Der Fokus wird dabei auf die Installation von PyTorch gelegt. 

## Erforderliches Equipment 
- [NVIDIA Jetson Nano](https://www.nvidia.com/de-de/autonomous-machines/embedded-systems/jetson-nano/)
- SD-Karte
- ggf. Maus und Tastatur 

## Installation von PyTorch und torchvision
---
### Schritt 1: Image auf SD-Karte installieren
Zuerst wird ein kompatibles Image benötigt, welches auf der SD-Karte installiert wird. Hierfür wird auf das [Getting Started](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) von NVIDIA verwiesen. Im Rahmen diese Tutorials wirde das JetPack SDK in der Version 4.5 installiert. Mit dem SDK werden wichtige Bibliotheken installiert, die relevant für die weiteren Schritte sind. Weitere Informationen zu dem SDK und allen installierten Toolkits sind in der [Dokumentation](https://developer.nvidia.com/embedded/jetpack) (für [JetPack 4.5](https://developer.nvidia.com/jetpack-sdk-45-archive)) zu finden. 

Achtung: Es gibt Unterschiede bei der Installation, wenn das [Jetson Nano 2GB Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-2gb-devkit) verwendet wird. 

### Schritt 2: Installation von torch
Die Installationsschritte von torch v1.7.0 unter Python 3.6 sind im Folgenden dargestellt. Für weitere Informationen wird auf die [Dokumentation](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-7-0-now-available/72048) verwiesen. 
```bash
wget https://nvidia.box.com/shared/static/cs3xn3td6sfgtene6jdvsxlr366m2dhq.whl -O torch-1.7.0-cp36-cp36m-linux_aarch64.whl
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev 
pip3 install Cython
pip3 install numpy torch-1.7.0-cp36-cp36m-linux_aarch64.whl
```

### Schritt 3: Installation von torchvision
Neben torch wird auch torchvision benötigt. Die Version von torchvision muss abhängig von der Version von torch gewählt werden. Für torch v1.7.0 wird torchvision v0.8.1 vorausgesetzt. Die Installation kann wie folgt vorgenommen werden. Detailiertere Informationen sind in der [Dokumentation](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-7-0-now-available/72048) zu finden. 
```bash
sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev
git clone --branch v0.8.1 https://github.com/pytorch/vision torchvision   
cd torchvision
export BUILD_VERSION=0.8.1  
python3 setup.py install --user
cd ../  # attempting to load torchvision from build dir will result in import error
```

### Schritt 4: Testen der Installation
Nachdem alle Pakete installiert wurden, kann die Einbindung in Python überprüft werden. 
```python
import torch

# Informationen über das verwendete System 
print(torch.__version__)
print('CUDA available: ' + str(torch.cuda.is_available()))
print('cuDNN version: ' + str(torch.backends.cudnn.version()))

# erste Rechenoperationen 
a = torch.cuda.FloatTensor(2).zero_()
print('Tensor a = ' + str(a))
b = torch.randn(2).cuda()
print('Tensor b = ' + str(b))
c = a + b
print('Tensor c = ' + str(c))
```

Entsprechend kann auch die Installation von torchvision überprüft werden, indem die verwendete Version ausgegeben wird.
```python
import torchvision
print(torchvision.__version__)
```

---
### Quellen:
- [Informationen zu JetPack](https://developer.nvidia.com/embedded/jetpack)
- [Dokumentation zur Installation von PyTorch auf dem NVIDIA Jetson Nano](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-7-0-now-available/72048)
