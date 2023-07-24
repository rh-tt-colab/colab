# Kafka Bridge

Extract CA from secret in OpenShift cluster:

```shell
oc extract secret/openshift-kafka-cluster --keys=ca.crt --to=- > ca.crt
```

Create the CA secret for TLS connection in east cluster

```shell
kubectl create secret generic openshift-cluster-ca-cert --from-file=ca.crt
```

Deploy the bridge in the east cluster

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