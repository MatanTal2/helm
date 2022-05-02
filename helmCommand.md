# Helm Command

## Prerequisites

- Docker installed and running
- Minikube installed
- Helm installed

1. Let starts with running minikube

    ```bash
    minikube start
    ```

    if the output is like so make sure your Docker is running.

    ```bash
    😄  minikube v1.25.2 on Ubuntu 20.04
    ✨  Using the docker driver based on existing profile

    💣  Exiting due to PROVIDER_DOCKER_VERSION_EXIT_1: "docker version --format -" exit status 1:
    📘  Documentation: https://minikube.sigs.k8s.io/docs/drivers/docker/
    ```

    The output for successful minikube luanch is:

    ```bash
    😄  minikube v1.25.2 on Ubuntu 20.04
    ✨  Using the docker driver based on existing profile
    👍  Starting control plane node minikube in cluster minikube
    🚜  Pulling base image ...
    🔄  Restarting existing docker container for "minikube" ...
    🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
        ▪ kubelet.housekeeping-interval=5m
    🔎  Verifying Kubernetes components...
        ▪ Using image kubernetesui/dashboard:v2.3.1
        ▪ Using image kubernetesui/metrics-scraper:v1.0.7
        ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    🌟  Enabled addons: storage-provisioner, default-storageclass, dashboard
    🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
    ```

2. Now that we have minikube running, we can try some helm command through simple demo and helm hub

Go to [Here](https://artifacthub.io/) and search for prometheus. or jsut click [here](https://artifacthub.io/packages/helm/prometheus-community/prometheus)

## Helm repo

let add the repo by clicking the installation button and copy the command `helm repo`, or here

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

let see the repo localy  - `helm repo list`

## Helm template

we can examine the chart by executing `helm template`

```bash
helm template prometheus-community/prometheus
```

## Helm install

now we can innstall with `helm command` the prometheus chart, simply by,

```bash
helm install my-prometheus prometheus-community/prometheus --version 15.8.5
```
Let see the result by execute `kubectl get service` and copy the `releasName-prometheus-server` and add ToPort:FromPort. loke so `service/[releasName]-prometheus-server 8080:80` and `port-forward` 

```bash
kubectl port-forward service/[releasName]-prometheus-server 8080:80
```

Go to to your local host in port 8080 and see the result.

## Helm upgrade

Let say we have differant values that we want to apply, in this case we will cahnge the `values.yaml` and `helm upgrade` the chart.
we will change the tag for one that isn't exist. and see the result.
In the helm hub/prometheus download the value `yaml` file and open it. change with text editor the tag in line 496 to 1.7.9 for example.
Save the file and execute the `helm upgrade` command.

```bash
helm upgrade promo prometheus-community/prometheus -f values-prometheus.yaml 
```

Let look over the pods. `kubectl get pods`. its look like the `prometheus-node-exporter` pod have err in the status `ErrImagePull`. let fix taht by rollback.
by default roolbask will roll back to the previous release if the revision is omitted

```bash
helm rollback releaseName  3
```

## Helm uninstall

To uninstall prometheus we will `helm uninstall releaseName`

```bash
helm uninstall releaseName
```

## helm remove


to remove repo from our repo list

```bash
helm repo remove releaseName
```

## Demo from our own

let now use our own deployment. in [here](https://github.com/MatanTal2/helm) we have Natali demo for helm.

we will use the same command and this time on Natali's demo.

for the tag value we will change from 1.0.2 to 1.0.3
