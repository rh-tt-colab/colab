# Kafka Bridge

In order to install the bridge in a k8s cluster (no OpenShift) you would need to install the Strimzi operator and get the CA root cert from the cluster where the Kafka cluster is running from.

## Extract CA from secret in OpenShift cluster:

```shell
oc extract secret/openshift-cluster-ca-cert --keys=ca.crt --to=- > ca.crt
```

1. Create the CA secret for TLS connection in East and West:

```shell
kubectl create secret generic openshift-cluster-ca-cert --from-file=ca.crt -n kafka-bridge
```

## Deploy the Operator in East and West

```shell
helm install my-strimzi-cluster-operator oci://quay.io/strimzi-helm/strimzi-kafka-operator --create-namespace -n kafka-operator --version 0.36.0 --set watchNamespaces="{kafka-bridge}"
```
## Deploy the bridge in the east cluster

```shell
kubectl apply -f kafka-bridge.yaml
```

Create the CA secret for TLS connection in west cluster

```shell
kubectl create secret generic openshift-cluster-ca-cert --from-file=ca.crt
```

Deploy the bridge in the west cluster

```shell
kubectl apply -f kafka-bridge.yaml
```

Test the bridge from the bridge pod

```shell
curl -X GET \
  http://localhost:8080/topics
```