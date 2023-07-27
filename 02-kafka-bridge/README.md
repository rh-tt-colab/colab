# Kafka Bridge

In order to install the bridge in a k8s cluster (no OpenShift) you would need to install the Strimzi operator first and get the CA root cert from the cluster where the Kafka cluster is running from.

## Prep namespace

```shell
kubectl create ns kafka-bridge
```

## Deploy the Operator

`NOTE` Perform this step in each k8s target cluster

```shell
helm install my-strimzi-cluster-operator oci://quay.io/strimzi-helm/strimzi-kafka-operator --create-namespace -n kafka-operator --version 0.36.0 --set watchNamespaces="{kafka-bridge}"
```

Check the operator pods are running:

```shell
k get pods -n kafka-operator
NAME                                        READY   STATUS    RESTARTS   AGE
strimzi-cluster-operator-85d7965bf9-hcdfw   1/1     Running   0          2d10h
```
## Deploy the Kafka Bridges

`NOTE` Perform this step in each k8s target cluster

### Prep certs in the target cluster:

1. Using the previously extracted CA create the secret for TLS connection:

```shell
kubectl create secret generic openshift-cluster-ca-cert --from-file=ca.crt -n kafka-bridge
```

### Deploy the bridge

```shell
kubectl apply -f kafka-bridge.yaml -n kafka-bridge
```

`NOTE` Notice how the Kafka Bridge CR has the label `failover: enabled` which is going to be used later on.

Check the bridge pod is running:

```shell
k get pods -n kafka-bridge -w
NAME                                  READY   STATUS    RESTARTS   AGE
kafka-bridge-bridge-b86c49748-cbltz   1/1     Running   0          23s
```

Check the logs to make sure the bridge is working fine:

```bash
k logs kafka-bridge-bridge-b86c49748-cbltz -n kafka-bridge
Preparing truststore
Certificate was added to keystore
Preparing truststore is complete
Kafka Bridge configuration:
#Bridge configuration
bridge.id=kafka-bridge

#Kafka common properties
kafka.bootstrap.servers=openshift-kafka-bootstrap-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443
kafka.security.protocol=SSL
#TLS/SSL
kafka.ssl.truststore.location=/tmp/strimzi/bridge.truststore.p12
kafka.ssl.truststore.password=[hidden]
kafka.ssl.truststore.type=PKCS12
...
...
...
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-0] AppInfoParser:119 - Kafka version: 3.5.0
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-0] AppInfoParser:120 - Kafka commitId: c97b88d5db4de28d
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-0] AppInfoParser:121 - Kafka startTimeMs: 1690470479736
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-0] HttpBridge:102 - HTTP-Kafka Bridge started and listening on port 8080
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-0] HttpBridge:103 - HTTP-Kafka Bridge bootstrap servers openshift-kafka-bootstrap-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443
2023-07-27 15:07:59 INFO  [vert.x-eventloop-thread-2] Application:125 - HTTP verticle instance deployed [2d8cbf0e-49e3-4a9d-8ba7-3ba3cdf38da0]
2023-07-27 15:13:00 INFO  [kafka-admin-client-thread | adminclient-1] NetworkClient:977 - [AdminClient clientId=adminclient-1] Node -1 disconnected.
```

### Test the bridge

Test the bridge by forwarding the port:

```shell
k port-forward deployment/kafka-bridge-bridge -n kafka-bridge 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
```

```shell
curl -X GET http://localhost:8080/topics
["__strimzi_store_topic","__strimzi-topic-operator-kstreams-topic-store-changelog"]
```

Now you have working Kafka bridges.