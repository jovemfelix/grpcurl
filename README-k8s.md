# Deploy the imagem to test your gRPC connections
> Remember to select the namespace you desire to deploy!

## Deployment

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grpcurl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grpcurl
  template:
    metadata:
      labels:
        app: grpcurl
        deploymentconfig: support-tools
    spec:
      containers:
      - image: quay.io/rfelix/grpcurl:v1.8.7.bash
        imagePullPolicy: IfNotPresent
        name: grpcurl
        stdin: true
        tty: true
```

### Accesing it using terminal: using openshift client

```sh
$ oc rsh $(oc get pod -oname -l app=grpc)
```



----

## Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grpcurl
  labels:
    app: grpcurl
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: grpcurl
    image: quay.io/rfelix/grpcurl:latest
    imagePullPolicy: IfNotPresent
    stdin: true
    tty: true
```

### Accesing it using terminal: using openshift client

```shell
$ oc rsh grpcurl
```



## Need to add a Proto to test?

```yaml
---
kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    app: grpcurl
  name: grpcurl-config
data:
  helloworld.proto: |-
    syntax = "proto3";

    option java_multiple_files = true;
    option java_package = "com.redhat.example";
    option java_outer_classname = "HelloWorldProto";

    package helloworld;

    // The greeting service definition.
    service Greeter {
        // Sends a greeting
        rpc SayHello (HelloRequest) returns (HelloReply) {}
    }

    // The request message containing the user's name.
    message HelloRequest {
        string name = 1;
    }

    // The response message containing the greetings
    message HelloReply {
        string message = 1;
    }

---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: grpcurl
  name: grpcurl
spec:
  terminationGracePeriodSeconds: 0
  containers:
  - name: grpcurl
    image: quay.io/rfelix/grpcurl:latest
    imagePullPolicy: IfNotPresent
    stdin: true
    tty: true
    volumeMounts:
      - name: config-volume
        mountPath: /config
        readOnly: true
  volumes:
    - name: config-volume
      configMap:
        name: grpcurl-config
```

