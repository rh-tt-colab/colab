# Kafka Cluster

To deploy the Kafka Cluster on OpenShift follow the steps below.

## Pre-requisites

- cert-manager
- Openshift cluster
- Cluster onboarded into TSB
- Kafka operator installed

## Login into OCP

Get a token from the UI and login over the terminal:

```bash
oc login --token=sha256~<<TOKEN>> --server=https://api.<<HOSTNAME>>:6443 --insecure-skip-tls-verify=true
```

## Prep Certs

Create a cluster issuer for kafka:

```shell
kubectl apply -f cluster-issuer-selfsigned-issuer-kafka.yaml
```

Create a certificate:

`NOTE` Make sure to update the hostnames in the Certificate `dnsNames` to match the brokers routes usually in the form of:
- openshift-kafka-bootstrap-kafka-cluster.apps.<<YOUR_HOSTNAME>>
- openshift-kafka-0-kafka-cluster.apps.<<YOUR_HOSTNAME>>
- openshift-kafka-1-kafka-cluster.apps.<<YOUR_HOSTNAME>>
- openshift-kafka-2-kafka-cluster.apps.<<YOUR_HOSTNAME>>

```shell
kubectl apply -f certificate-openshift-kafka-cluster.yaml -n kafka-cluster
```

## Deploy Kafka Cluster

```shell
oc apply -f kafka-cluster-persistent.yaml -n kafka-cluster
```

## Check that you have all the kafka pods running:

```shell
k get pods -n kafka-cluster
NAME                                        READY   STATUS    RESTARTS   AGE
openshift-entity-operator-85c5f6949-v6wtp   3/3     Running   0          17h
openshift-kafka-0                           1/1     Running   0          17h
openshift-kafka-1                           1/1     Running   0          17h
openshift-kafka-2                           1/1     Running   0          17h
openshift-zookeeper-0                       1/1     Running   0          17h
openshift-zookeeper-1                       1/1     Running   0          17h
openshift-zookeeper-2                       1/1     Running   0          17h
```

## Test your kafka using `kcat`:

`NOTE` You can get `kcat` docs at https://docs.confluent.io/platform/current/clients/kafkacat-usage.html

```bash
kcat -X security.protocol=SSL -X enable.ssl.certificate.verification=false -b openshift-kafka-bootstrap-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443 -L
```

Expect an output like:

```bash
Metadata for all topics (from broker -1: ssl://openshift-kafka-bootstrap-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443/bootstrap):
 3 brokers:
  broker 0 at openshift-kafka-0-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443 (controller)
  broker 2 at openshift-kafka-2-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443
  broker 1 at openshift-kafka-1-kafka-cluster.apps.gke-nmocpt2-us-west1-0.nmocpt2-upic-0.gcp.sandbox.tetrate.io:443
 3 topics:
  topic "__strimzi-topic-operator-kstreams-topic-store-changelog" with 1 partitions:
    partition 0, leader 0, replicas: 0,2,1, isrs: 0,2,1
  topic "__strimzi_store_topic" with 1 partitions:
    partition 0, leader 2, replicas: 2,0,1, isrs: 2,0,1
  topic "__consumer_offsets" with 50 partitions:
    partition 0, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 1, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 2, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 3, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 4, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 5, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 6, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 7, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 8, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 9, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 10, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 11, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 12, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 13, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 14, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 15, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 16, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 17, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 18, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 19, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 20, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 21, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 22, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 23, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 24, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 25, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 26, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 27, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 28, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 29, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 30, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 31, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 32, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 33, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 34, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 35, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 36, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 37, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 38, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 39, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 40, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 41, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 42, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 43, leader 0, replicas: 0,2,1, isrs: 0,2,1
    partition 44, leader 2, replicas: 2,1,0, isrs: 2,1,0
    partition 45, leader 1, replicas: 1,2,0, isrs: 1,2,0
    partition 46, leader 0, replicas: 0,1,2, isrs: 0,1,2
    partition 47, leader 2, replicas: 2,0,1, isrs: 2,0,1
    partition 48, leader 1, replicas: 1,0,2, isrs: 1,0,2
    partition 49, leader 0, replicas: 0,2,1, isrs: 0,2,1
```

Now you have a functional Kafka cluster and can switch context to the other k8s clusters and install the Kafka bridge. For this you will need the CA used for the Kafka cluster:

```shell
oc extract secret/openshift-kafka-cluster -n kafka-cluster --keys=ca.crt --to=- > ca.crt
```