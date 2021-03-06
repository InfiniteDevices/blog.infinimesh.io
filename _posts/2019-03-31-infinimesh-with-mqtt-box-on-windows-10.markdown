---
title: "HowTo infinimesh and MQTTBox on Windows 10"
date: 2019-03-31 14:34:00
description: How to connect MQTTBox on Windows to infinimesh to send and receive data
---

Level: Beginner  
Time: 20 minutes  
Tools: Windows 10, WSL enabled  

To get Windows 10 and infinimesh working together takes you approximately 20 minutes, if you are an experienced user probably under 3 minutes. 

#### Preparation
1. Subscribe to infimesh and obtain a username and password from <a href="https://www.infinimesh.io/signup.html" target="_blank">https://www.infinimesh.io/signup.html</a>
2. Make sure you have WSL (Windows Subsystem for Linux) enabled and you have installed a distro of your choice (<a href="https://docs.microsoft.com/en-us/windows/wsl/install-win10" target="_blank">https://docs.microsoft.com/en-us/windows/wsl/install-win10)</a>
3. Install infinimesh CLI (Command Line Interface) within WSL (Linux bash)
``` 
curl -L https://bit.ly/2CNKWzJ | BINDIR=$HOME/bin bash  
echo "export PATH=$HOME/bin:$PATH" >> ~/.profile && . ~/.profile  
inf config set-context saas --apiserver grpc.api.infinimesh.io:443 --tls=true
``` 
 
4. Use the native openssl command from your linux terminal (bash) and create self-signed X509 certificates   
```
openssl genrsa -out my-first-device.key 4096 && \
openssl req -new -x509 -sha256 -key my-first-device.key -out my-first-device.crt -days 365    
```
5. Copy all certificates (.crt, .key and ca-certificate.crt) to a directory you can easily remember and which can be read by Windows Explorer
  * Tip: The windows filesystem is mounted over /mnt in WSL, to copy the files this might work for you, too: 

```
mkdir /mnt/c/users/YOUR-USERNAME/certificates 
cp *.key *.cert /etc/ssl/certs/ca-certificates.crt /mnt/c/users/YOUR-USERNAME/certificates
```
6. Download and install MQTTBox from <a href="https://www.microsoft.com/en-us/p/mqttbox/9nblggh55jzg" target="_blank">Windows App Store</a>  

#### Create your first device
Let's now play with our new toy, the preparation was long enough!  

Login to infinimesh with your obtained username and password:
```
inf login
```  

As usual, before we can send data from our device we need to register them. That typically happens at an ODM (Original Design Manufacturer), but since we play with our own toys we do that on our own, too:
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

That's all, now the device is registered and ready to work!

#### Use MQTTBox to send data to the platform
First setup a new MQTT Client connection using those parameters:  
```
Protocol: mqtts / tls  
Host: mqtt.api.infinimesh.io:8883  
SSL / TLS version: TLSv1.2  
SSL / TLS Certificate Type: Choose "Self signed certificates"  
```

In the three SSL certificate boxes choose the right certificate per field:  
CA file: your CA, the file named ```ca-certificates.crt``` (typical found in WSL under ```/etc/ssl/certs/ca-certificates.crt```)  
Client certificate file: ```my-first-device.crt```  
Client key file: ```my-first-device.key```  

All other parameters can be left as they are. Klick Save. 

#### Subscribe to the device topic and send data to the device twin
Now, after MQTTBox is properly connected we also want to send some data. In the case you don't see the publisher box, click on ```Add publisher```. Our IoT platform has per default Device State Management builtin, so we just need to publish to ```devices/<YOUR DEVICE ID>/state/reported/delta``` and add the proper device id we can see with ```inf device list```. For example, the correct publisher connection string should look like:  
```
devices/0x9e/state/reported/delta
```  

As QoS we use ```Almost Once``` or ```Atleast once```. As payload you can send any allowed JSON value.
Since using ```Hello world``` as the first command when something new is learned let's use that and paste the following content into the ```Payload``` field:
```
{
"hello":"World",
"infinimesh":"Works"
}
```
To see if we did all things right we use the state command ```inf state get YOUR DEVICEID``` - it should look like this:
```
inf state get 0x9e
Reported State:
  Version:    1
  Timestamp:  2019-03-31 14:43:32.993803713 +0200 DST
  Data:
    {
      "hello": "World",
      "infinimesh": "Works"
    }
Desired State: <none>
Configuration: <none>
```
Congratulations! You have sent the first data from your emulated device to infinimesh and you have proven that it worked. 

### Conclusion  
In this HowTo we have prepared our Windows 10 system to work properly with infinimesh. We created X509 certificates for our device, and we registered the device so it is known in your own infinmesh name space. Short after that we have sent data to the device and to proof that the data was properly received we were able to read the status change from our CLI. A good read would be also our <a href="https://infinimesh.github.io/infinimesh/docs/#/quickstart" target="_blank">Quickstart Manual</a>.

#### For Windows 10 admins and high experienced users
To create SSL certificates using standard windows tools is also possible, a good read is <a href="https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8" target="_blank">https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8</a>
