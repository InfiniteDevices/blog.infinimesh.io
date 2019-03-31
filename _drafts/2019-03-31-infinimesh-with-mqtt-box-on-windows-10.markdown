---
title: "HowTo infinimesh and MQTTBox on Windows 10"
date: 2019-03-31 14:34:00
description: How to connect MQTTBox on Windows to infinimesh to send and receive data
---

To get Windows 10 and inifimesh working together takes you approximately 20 minutes, if you are an experienced user probably under 3 minutes. 

#### Preparation
1. Subscribe to infimesh and obtain a username and password (<a href="https://www.infinimesh.io/signup.html" _target="_blank">https://www.infinimesh.io/signup.html</a>)
2. Create Self-signed X509 certificates
  * Make sure you have WSL (Windows Subsystem for Linux) enabled and install a distro of your choice (<a href="https://docs.microsoft.com/en-us/windows/wsl/install-win10" target="_blank">https://docs.microsoft.com/en-us/windows/wsl/install-win10)</a>
  * Use the native openssl command from your linux terminal (bash): 
```
openssl genrsa -out my-first-device.key 4096 && openssl req -new -x509 -sha256 -key my-first-device.key -out my-first-device.crt -days 365    
```
3. copy all certificates (.crt, .key and ca-certificate.crt) to one directory you can easily remember
  * tip: The windows filesystem is mounted over /mnt in WSL, to copy the files use: 

```
mkdir /mnt/c/users/YOUR-USERNAME/certificates 
cp *.key *.cert /etc/ssl/certs/ca-certificates.crt /mnt/c/users/YOUR-USERNAME/certificates
```
4. Download and install MQTTBox from Windows App Store: <a href="https://www.microsoft.com/de-de/p/mqttbox/9nblggh55jzg?ocid=badge&rtc=1&activetab=pivot:overviewtab" target="_blank">https://www.microsoft.com/en-us/p/mqttbox</a>

#### Create your first device









backup

  1. Using standard windows tools, a good read is <a href="https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8" target="_blank">https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8</a>
