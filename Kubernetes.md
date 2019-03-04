# Kubernetes

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
