---
title: "Install infinimesh platform in Kubernetes"
date: 2019-04-22 13:25:00
description: Get infinimesh IoT Platform running in kubernetes for local or on-premise installation
---

We have built infinimesh (<a href="https://www.infinimesh.io" target="_blank">https://www.infinimesh.io</a>) to run everywhere. 
In this blogpost we use <a href="https://microk8s.io/" target="_blank">microk8s</a> as local installation, but
any kubernetes distribution would work, too. 
We did installations with Canonical Multi-Cloud installations 
(<a href="https://www.ubuntu.com/kubernetes" target="_blank">https://www.ubuntu.com/kubernetes</a>), as example. 
  
Okay, let's start.
  
## Installing operators
Infinimesh is a Cloud Native Application and leverages Kubernetes Operators to install and maintain the platform. Depending on your setup, you may need to install:
- Infinimesh Operator
- Kafka Strimzi Operator (optional)
- KubeDB Operator
<br><br>
### Set the default config for your cluster
infinimesh can be installed in different clusters from one console. To do so export the config you want to use, as example:
```
export KUBECONFIG=$KUBECONFIG:~./kubeconfig
```
  
### Infinimesh Operator
The following commands install the Infinimesh Kubernetes Operator into `infinimesh-system`.
```
kubectl apply -f https://raw.githubusercontent.com/infinimesh/operator/master/manifests/crd.yaml
kubectl apply -f https://raw.githubusercontent.com/infinimesh/operator/master/manifests/operator.yaml
```
  
### KubeDB Operator (optional)
To provision Postgres and Redis instances, infinimesh uses `KubeDB`.
```
curl -fsSL https://raw.githubusercontent.com/kubedb/cli/0.11.0/hack/deploy/kubedb.sh | bash
```
  
### Strimzi Kafka Operator (optional)
Strimzi is a Kubernetes operator to install and maintain Kafka in a Cloud Native way. Strimzi is not required by Infinimesh. However, if you don't already have a Kafka installation, it is the recommended way to install Kafka.
*If you already have a Kafka Cluster, you can skip this step.*
  
Install Strimzi Kafka Operator into the namespace `kafka`:
```
kubectl create namespace kafka
curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.11.1/strimzi-cluster-operator-0.11.1.yaml \
  | sed 's/namespace: .*/namespace: kafka/' \
  | kubectl -n kafka apply -f -
```
  
For more details refer to <a href="https://strimzi.io/quickstarts/minikube/" target="_blank">https://strimzi.io/quickstarts/minikube/</a>.
  
### NGINX Ingress Controller
  
For API Servers and App, infinimesh uses Ingrees Controller. When using Minikube, enable it as addon:
```
minikube addons enable ingress
```
or with microk8s:
```
microk8s.enable ingress
```
If you use microk8s with multipass use: 
```
multipass exec microk8s-vm -- /snap/bin/microk8s.enable ingress
```
  
When deploy to a real cluster, follow the instructions here: (https://kubernetes.github.io/ingress-nginx/deploy/). 
This involves installing some manifests or a HELM chart and may vary slightly, depending on your infrastructure.
  
## TLS Certicate Setup
Infinimesh requires X509 KeyPairs for the following applications:
- API Server gRPC
- API Server REST
- MQTT Bridge
- App (User Interface)
  
To manage the certificates, infinimesh uses `cert-manager`. It is a Kubernetes operator to provision certificates. 
A typical usage is provisioning via `Let's Encrypt`, but infinimesh uses it for all kinds of certificates.
  
You may serve these domains from any host/domain you want. It's just critical that you have TLS server certificates for each of them, with the used domain name.
For local and development installation, certificates can be generated with `openssl` and self-signed. This is not recommended for producation scenarios, because other parties (e.g. users of the platform) will have to import the certificates into their trust store. This is not just inconvenient, but in most scenarios also makes the platform vulnerable to `Man in the middle atacks`. Thus, use self-signed certificates only local & testing environments.
  
You need `openssl` for the following steps.
  
0. Generate Root CA
0.1 Generate Private Key
```
openssl genrsa -out ca.key 4096
openssl req -subj '/CN=infinimesh.local/O=Infinimesh' -new -x509 -sha256 -key ca.key -out ca.crt -days 3650
```

1. Generate Platform Private Keys & Certificates
1.1 API Server
```
openssl genrsa -out apiserver_grpc.key 4096
openssl req -subj /CN=grpc.api.infinimesh.local -out apiserver_grpc.csr -key apiserver_grpc.key -new
openssl x509 -req -days 3650 -in apiserver_grpc.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver_grpc.crt -sha256 
```
1.2 API Server REST
```
openssl genrsa -out apiserver_rest.key 4096
openssl req -subj /CN=api.infinimesh.local -out apiserver_rest.csr -key apiserver_rest.key -new
openssl x509 -req -days 3650 -in apiserver_rest.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver_rest.crt -sha256 
```
1.3 MQTT Bridge
```
openssl genrsa -out mqtt_bridge.key 4096
openssl req -subj /CN=mqtt.api.infinimesh.local -out mqtt_bridge.csr -key mqtt_bridge.key -new
openssl x509 -req -days 3650 -in mqtt_bridge.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out mqtt_bridge.crt -sha256 
```

1.4 App
```
openssl genrsa -out app.key 4096
openssl req -subj /CN=app.infinimesh.local -out app.csr -key app.key -new
openssl x509 -req -days 3650 -in app.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out app.crt -sha256 
```

If you want to use different domains, replace `localhost` in the `-subj` parameter.
  
Now we transform these Cert & Key files into Kubernetes secrets, and deploy them to the cluster.

```
kubectl create secret tls apiserver-grpc-tls --cert apiserver_grpc.crt --key apiserver_grpc.key 
kubectl create secret tls apiserver-rest-tls --cert apiserver_rest.crt --key apiserver_rest.key 
kubectl create secret tls mqtt-bridge-tls --cert mqtt_bridge.crt --key mqtt_bridge.key 
kubectl create secret tls app-tls --cert app.crt --key app.key 
```
  
## Deploy Kubernetes Resources
Now we're installing the resources required by Infinimesh into Kubernetes:
- Infinimesh itself via the `Platform` CRD
- Kafka via Strimzi Operator (optional)
- Databases
  
Note: We already installed Secrets in the previous step.
  
1. Create a onfinimesh Platform Resource
Copy the following into a local file, I named it platform.yaml for simplicity. Tip: "name" can be edited to fit your installation.
Important: Take care that the host&tls config match to the deployed secrets we created before!
  
```
apiVersion: infinimesh.infinimesh.io/v1beta1
kind: Platform
metadata:
  labels:
    controller-tools.k8s.io: "1.0"
  name: my-infinimesh
spec:
  kafka:
    bootstrapServers: "infinimesh-kafka-bootstrap.kafka.svc.cluster.local:9092"
  app:
    host: "app.infinimesh.minikube"
    tls:
      - hosts:
        - "app.infinimesh.minikube"
        secretName: "app-tls"
  mqtt:
    secretName: "mqtt-bridge-tls"
  apiserver:
    restful:
      host: "api.infinimesh.minikube"
      tls:
        - hosts:
          - "api.infinimesh.minikube"
          secretName: "apiserver-rest-tls"
    grpc:
      host: "grpc.api.infinimesh.minikube"
      tls:
        - hosts:
          - "grpc.api.infinimesh.minikube"
          secretName: "apiserver-grpc-tls"
```
and apply the manifedt via

```
kubectl apply -f platform.yaml
```
  
2. Kafka
infinimesh uses Apache Kafka (<a href="https://kafka.apache.org" target="_blank">https://kafka.apache.org</a>) as message broker,
to get Kafka within Kubernetes easy installed the use of Strimzi is highly recommended.
```
apiVersion: kafka.strimzi.io/v1alpha1
kind: Kafka
metadata:
  name: infinimesh
spec:
  kafka:
    version: 2.1.0
    replicas: 1
    listeners:
      plain: {}
      tls: {}
    config:
      offsets.topic.replication.factor: 1
      transaction.state.log.replication.factor: 1
      transaction.state.log.min.isr: 1
      log.message.format.version: "2.1"
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  zookeeper:
    replicas: 1
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
```
And apply again:

```
kubectl apply -f kafka.yaml -n kafka
```
  
It is important that the kafka resource is created in the same namewhere where the kafka operator is located (by default `kafka`)
  
## Access Infinimesh

### DNS
Since we use TLS and use specific hostnames, we have to add those to our `hosts` file.

```
192.168.99.106 grpc.api.infinimesh.local
192.168.99.106 api.infinimesh.local
192.168.99.106 app.infinimesh.local
192.168.99.106 mqtt.api.infinimesh.local
```
  
Replace 192.168.99.106 with the address of your minikube instance. You can find it by running `multipass service list` or `minikube service list`.

### Trust self-signed CA

In addition, since we use self-signed certificates, we must trust these certificates. Note: just trusting the certificate in the UI is not sufficient; since we have multiple certs for api, grpc, mqtt, app.
But we were smart enough to sign these with *one* self signed root cert, so we only have to import `ca.crt` into our browser.

To trust the root certificate, you must go to your browser settings and add the file `ca.crt` as an certificate `Authority`.
  
### Access CLI
If you haven't before, update or install `inf`, our CLI tool via:
```
curl -L https://bit.ly/2CNKWzJ | BINDIR=$HOME/bin bash
```

Use `set-context` to add a config in the CLI:
```
inf config set-context microk8s --apiserver grpc.api.infinimesh.local:443 --tls=true --ca-file ca.crt
```
    
In order to log in, you have to get the password of the root user. The Kubernetes operator took care of this; it auto-generated the root user with a random password and stored it in the Kubernetes secret `my-infinimesh-root-account`.
  
```
kubectl get secret my-infinimesh-root-account -o=jsonpath='{.data.password}' | base64 -d
```
Now login using root/ and the password:
```
inf login
```

Use the native openssl command from your linux terminal (bash) and create self-signed X509 certificates for your test device 
```
openssl genrsa -out my-first-device.key 4096 && \
openssl req -new -x509 -sha256 -key my-first-device.key -out my-first-device.crt -days 365  
```

Create a device within infinimesh
```
inf device create sample-device --cert-file my-first-device.crt
```
  
### Access UI
You can access the UI at <a href="https://app.infinimesh.local" target="_new">https://app.infinimesh.local</a>.
  
### Access MQTT
Since we are on microk8s or Minikube, we have to find out the `NodePort` of the service (type LoadBalancer is not available on both):
  
```
kubectl get svc my-infinimesh-mqtt-bridge -o=jsonpath='{.spec.ports[].nodePort}' 
```
Note that the name differ from your installation when you have selected another name (platform.yaml).
  
Send a state message:
```
mosquitto_pub --cafile ca.crt --cert sample_1.crt --key sample_1.key -m '{"sensor" : {"temp" : 41}}' -t "devices/0x2711/state/reported/delta" -h mqtt.api.infinimesh.minikube --tls-version tlsv1.2 -d -p 31108
```
  
That's all - now your have your own IoT platform running. We recommend this kind of installation in every IoT or IIoT project to
enable rapid prototyping. But we have also a free SaaS offering available, you just need to <a href="https://www.infinimesh.io/signup.html" target="_new">signup</a>, 
25 devices are free. 




