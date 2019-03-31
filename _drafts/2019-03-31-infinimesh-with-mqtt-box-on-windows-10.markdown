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
1. Subscribe to infimesh and obtain a username and password (<a href="https://www.infinimesh.io/signup.html" _target="_blank">https://www.infinimesh.io/signup.html</a>)
2. Make sure you have WSL (Windows Subsystem for Linux) enabled and you have installed a distro of your choice (<a href="https://docs.microsoft.com/en-us/windows/wsl/install-win10" target="_blank">https://docs.microsoft.com/en-us/windows/wsl/install-win10)</a>
3. Install infinimesh CLI (Command Line Interface) within WSL (Linux bash)
   ``` curl https://raw.githubusercontent.com/infinimesh/infinimesh/master/godownloader.sh | BINDIR=$HOME/bin bash```
   ``` echo "export PATH=$HOME/bin:$PATH" >> ~/.profile && . ~/.profile```
   ``` inf config set-context saas --apiserver grpc.api.infinimesh.io:443 --tls=true``` 
 
4. Create Self-signed X509 certificates
   Use the native openssl command from your linux terminal (bash):   
```
openssl genrsa -out my-first-device.key 4096 && \
openssl req -new -x509 -sha256 -key my-first-device.key -out my-first-device.crt -days 365    
```
5. copy all certificates (.crt, .key and ca-certificate.crt) to a directory you can easily remember and which can be read by Windows Explorer
  * Tip: The windows filesystem is mounted over /mnt in WSL, to copy the files this might work for you, too: 

```
mkdir /mnt/c/users/YOUR-USERNAME/certificates 
cp *.key *.cert /etc/ssl/certs/ca-certificates.crt /mnt/c/users/YOUR-USERNAME/certificates
```
6. Download and install MQTTBox from Windows App Store: <a href="https://www.microsoft.com/de-de/p/mqttbox/9nblggh55jzg?ocid=badge&rtc=1&activetab=pivot:overviewtab" target="_blank">https://www.microsoft.com/en-us/p/mqttbox</a>

#### Create your first device
Lets now play with our new toy, the preparation was long enough!  

As usual, before we can send data from our device we need to register them. Thats typically happen at an ODM (Original Design Manufacturer), but since we play with our toys we do that:
```
inf device create raspi-test1 --cert-file PATH/TO/YOUR/CERTIFICATE/my-first-device.crt -n <YOUR USERNAME>
```
as example:
```
inf device create infinimesh-QA --cert-file /mnt/c/users/QA/tests/certificates/qa-test159.crt -n infinmesh-qa
```
To control that the device in properly registered use the list command:
```
inf device list
```

Thats all, now the device is registered and ready to work!

#### Use MQTTBox to send and receive data
First setup a new MQTT Client connection using those parameter:  
```
Protocol: mqtts / tls  
Host: mqtt.api.infinimesh.io:8883  
SSL / TLS version: TLSv1.2  
SSL / TLS Certificate Type: Choose "Self signed certificates"  
Username: YOUR UNSERNAME  
Password: YOUR PASSWORD
```

In the three SSL certificate fileds choose the right certificates:  
CA file: your CA, the file named ```ca-certificates.crt``` (typical found in WSL under ```/etc/ssl/certs/ca-certificates.crt```)  
Client certificate file: ```my-first-device.crt```  
Client key file: ```my-first-device.key```  

All other parameter can be left as they are. Klick Save. 

#### Send data and subscribe to the device topic
Now, after MQTTBox is properly connected we also want to send some data. In the case you don't see the publisher box, click on ```Add publisher```. Our IoT platform has per default Device Shadow builtin, so we just need to publish to ```shadows/``` and add the proper device id we can see with ```inf device list```. As QoS we use ```Exactly Once```, but that depends on your use case you want to prove. As payload type choose one which is suffienct for you, in that HowTo we use simply ```Strings / JSON / XML / Characters```, but infimesh works also with binary data.  
Since using ```Hello world``` as the first ever made command is the standard lets use that and paste the following content into the ```Payload``` field:
```
{
"hello":"infinimesh",
"it":"works"
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
      "hello": "infinimesh",
      "it": "works"
    }
Desired State: <none>
Configuration: <none>
```
Congratulations! You have sent the first data from your emulated device to infinimesh and you have proven that it worked. 

### Conclusion  
In this HowTo we have prepared our Windows 10 system to work properly with infinimesh, including installing Go. We created X509 certificates for our device, and we registered the device so it is known in your own infinmesh name space. Short after that we have sent data to the device and to proof that the data was properly received we were able to read the status change from our CLI. A good read would be also our <a href="https://infinimesh.github.io/infinimesh/docs/#/quickstart" target="_new">Quickstart Manual</a>.

#### For Windows 10 admins and high experienced users
To create SSL certificates using standard windows tools is also possible, a good read is <a href="https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8" target="_blank">https://medium.com/the-new-control-plane/generating-self-signed-certificates-on-windows-7812a600c2d8</a>
