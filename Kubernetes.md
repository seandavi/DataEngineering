# Kubernetes

## Resources

- [Kubernetes for docker users](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)
- [fire up an interactive bash within kubernetes (for throwaway stuff)](https://gc-taylor.com/blog/2016/10/31/fire-up-an-interactive-bash-pod-within-a-kubernetes-cluster)

## Google Kubernetes Engine

Google Kubernetes Engine is a managed platform for running kubernetes clusters.

### Premliminaries

On my Mac, I have to do this:

```sh
export KUBECONFIG=$HOME/.kube/config-main
```

to avoid errors like:

```
$ gcloud container clusters get-credentials $CLUSTER_NAME
Fetching cluster endpoint and auth data.
ERROR: (gcloud.container.clusters.get-credentials) Unable to create private file [/private/tmp]: [Errno 1] Operation not permitted: '/private/tmp'
```

### Creating a cluster

```sh
export CLUSTER_NAME=<YOUR CLUSTER NAME>
```

```sh
gcloud container clusters create $CLUSTER_NAME
gcloud container clusters get-credentials $CLUSTER_NAME
```

## Administering clusters

In kubernetes `kubectl` world, a `context` is a cluster with which `kubectl` is configured to work. 

- list all contexts

```
kubectl config get-contexts
```

## Jobs

- https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
- https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/#writing-a-job-spec

### Example

Compute pi to 200 digits (about 10 seconds)

File: job.yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

Run the `pi` job.

```sh
kubectl create -f https://k8s.io/examples/controllers/job.yaml
kubectl describe jobs/pi
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}') \
  && echo $pods \
  && kubectl logs $pods
```

The last command will give us pi to 200 digits as output.

And cleanup. 

```
kubectl delete jobs/pi
# or
kubectl delete -f ./job.yaml
```

### Authentication for google

On kubernetes, one needs to set up a key file to allow tools like gsutil to work correctly. Assuming that one has
a key file (from a service account):

```
kubectl create secret generic google-key --from-file=key.json=PATH-TO-KEY-FILE.json
```

File: job2.yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: omicidx
spec:
  template:
    metadata:
      labels:
        app: omicidx
    spec:
      volumes:
      - name: google-cloud-key
        secret:
          secretName: google-key
      containers:
      - name: subscriber
        # just to test gsutil, for example
        image: google/cloud-sdk:latest 
        command: ['gsutil', 'ls']
        volumeMounts:
        - name: google-cloud-key
          mountPath: /var/secrets/google
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/key.json
      restartPolicy: Never
  backoffLimit: 4
```

This job will run once and the output can be found in the logs.

```
pods=$(kubectl get pods --selector=job-name=omicidx --output=jsonpath='{.items[*].metadata.name}')  \
  && echo $pods \
  && kubectl logs $pods
```

## Configuration

### secrets and configmaps

- https://medium.com/google-cloud/kubernetes-configmaps-and-secrets-68d061f7ab5b

```
kubectl create secret generic apikey --from-literal=API_KEY=123–456
```

```
kubectl create configmap language --from-literal=LANGUAGE=English
```

```
kubectl get secrets
kubectl get configmaps
```

And to use:

```

# Copyright 2017, Google, Inc.
# Licensed under the Apache License, Version 2.0 (the "License")
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: envtest
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: envtest
    spec:
      containers:
      - name: envtest
        image: gcr.io/<PROJECT_ID>/envtest
        ports:
        - containerPort: 3000
        env:
        - name: LANGUAGE
          valueFrom:
            configMapKeyRef:
              name: language
              key: LANGUAGE
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: apikey
              key: API_KEY
```


Replacing values:

```
kubectl create configmap language --from-literal=LANGUAGE=Spanish \
        -o yaml --dry-run | kubectl replace -f -
kubectl create secret generic apikey --from-literal=API_KEY=098765 \
        -o yaml --dry-run | kubectl replace -f -
```

And then restart pods:

```
kubectl delete pods -l name=XXXXXX # from label
```


## Quick answers

### Run an ubuntu instance on kubernetes with bash.

```
kubectl run my-shell --rm -i --tty --image ubuntu -- bash
```


## NOT USED

Get the name of the pod 

```
kubectl get pods
```

And run a command:

```
kubectl exec ubuntu-box-XXXXXXX -- cat /etc/hostname
```

Or attach to the pod interactively:

```
kubectl attach -ti ubuntu-box-XXXXXXX 
```
