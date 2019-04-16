---
title: "Setting up a raspberry pi infinimesh python client"
date: 2019-03-31 14:34:00
description: How-to connect a raspberry pi to the infinimesh platform to send and receive data
---

Level: Medium  
Time: 30 minutes  
Set-up: Linux PC, Raspberry Pi 3 B+


#### Preparation
1. Subscribe to infimesh and obtain a username and password from <a href="https://www.infinimesh.io/signup.html" target="_blank">https://www.infinimesh.io/signup.html</a>
2. Install infinimesh CLI on your pc
```
curl -L https://bit.ly/2CNKWzJ | BINDIR=$HOME/bin bash  
echo "export PATH=$HOME/bin:$PATH" >> ~/.profile && . ~/.profile  
inf config set-context saas --apiserver grpc.api.infinimesh.io:443 --tls=true
```

3. On your pi, use the native openssl command from your linux terminal (bash) and create self-signed X509 certificates   
```
openssl genrsa -out my-first-device.key 4096 && \
openssl req -new -x509 -sha256 -key my-first-device.key -out my-first-device.crt -days 365    
```
4. Copy all certificates (.crt, .key and ca-certificate.crt) to a directory of your choice (you will reference it later)


#### Create an online device in infinimesh  

On your pc, login to the infinimesh CLI with your obtained username and password:
```
inf login
```  

Before we can send data from our device we need to register it. That typically happens at an ODM (Original Design Manufacturer), but we can also do this on our own:
```
inf device create raspi-test1 --cert-file PATH/TO/YOUR/CERTIFICATE/my-first-device.crt -n <YOUR USERNAME>
```
For example:
```
inf device create infinimesh-QA --cert-file /mnt/c/users/QA/tests/certificates/qa-test159.crt -n infinmesh-qa
```
To control that the device is properly registered use the list command:
```
inf device list
```
Your device (including other you may have created before) will be printed to the screen.
That's all, now the device is registered and ready to work!

#### Set-up a python script to exchange data with the device
