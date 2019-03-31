---
title: "Install Go in your Windows 10 environment"
date: 2019-04-04 21:34:00
description: Installing Go and start to develop and contribute to infinmesh
---

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
