---
title: "HowTo infinimesh and MQTTBox on Windows 10"
date: 2019-03-31 14:34:00
description: How to connect MQTTBox on Windows to infinimesh to send and receive data
---

Level: Beginner
Time: 20 minutes
Tools: Windows 10, WSL enabled, Golang

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
  * tip: The windows filesystem is mounted over /mnt in WSL, to copy the files this might work for you, too: 

```
mkdir /mnt/c/users/YOUR-USERNAME/certificates 
cp *.key *.cert /etc/ssl/certs/ca-certificates.crt /mnt/c/users/YOUR-USERNAME/certificates
```
4. Since we wrote infinimesh completely in Go, you need to have Go properly in your WSL environment installed. To make it short you can easily follow this steps from within your linux bash:
```
wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz && tar -xvf go1.11.4.linux-amd64.tar.gz && sudo mv go /usr/local
```
to make go available at login you need to to edit .profile in your linux bash home directory, just copy and paste the following:
```
cd ~ && mkdir -p development && cat >> .profile <<EOL
# set go
export GOPATH=$HOME/developmentexport 
GOROOT=/usr/local/goexport 
PATH=$GOPATH/bin:$GOROOT/bin:$PATHEOL
```
5. Download and install MQTTBox from Windows App Store: <a href="https://www.microsoft.com/de-de/p/mqttbox/9nblggh55jzg?ocid=badge&rtc=1&activetab=pivot:overviewtab" target="_blank">https://www.microsoft.com/en-us/p/mqttbox</a>

#### Create your first device
Since we wrote infinimesh completely in Go, you need to have Go properly in your WSL environment installed. To make it short you can easily follow this steps from within your linux bash:
```
wget https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz && tar -xvf go1.11.4.linux-amd64.tar.gz && sudo mv go /usr/local
```
to make go available at login you need to to edit .profile in your linux bash home directory, just copy and paste the following:
```
cd ~ && mkdir -p development && cat >> .profile <<EOL  
# set go  
export GOPATH=$HOME/development  
export GOROOT=/usr/local/go  
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH  
EOL  
```









backup

  1. Using standard windows tools, a good read is <a href="https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8" target="_blank">https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8</a>
