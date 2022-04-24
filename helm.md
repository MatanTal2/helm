# Helm

Tutorial for helm

To start new Helm chart we just need this simple command and choose a name - we will call it rhinochart
```
helm create rhinochart
```

This will create new folder named rhinochart that contain a few thing. 
lets see the contant by `ls rhinochart`
```
Chart.yaml   charts/      templates/   values.yaml
```
The Chart.yaml and values.yaml define what the chart is and what values will be in it at deployment.


## Examine the chart's structure

Let examine the chart.yaml, `cat` the conatant of `chart.ymal`, the output should be something like this.

```yaml
apiVersion: v2
name: rhinochart
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
```

The first part includes the API version that the chart is using (this is required),
the name of the chart, and a description of the chart.

The next section describes the type of chart (an application by default, the other type can be is library), 
the version of the chart you will deploy, and the application version (which should be incremented as you make changes).

The most important part of the chart is the template directory.
It holds all the configurations for your application that will be deployed into the cluster.
As you can see below, this application has a basic deployment, ingress, service account, and service.
This directory also includes a test directory, which includes a test for a connection into the app.
 Each of these application features has its own template files under templates/:

 `ls templates` to see what inside.

 ```
NOTES.txt  _helpers.tpl  deployment.yaml  hpa.yaml  ingress.yaml  service.yaml  serviceaccount.yaml  tests
 ```

 There is another directory, called charts, which is empty.
 It allows you to add dependent charts that are needed to deploy your application.

## Understand and edit values

Template files are set up with formatting that collects deployment information from the values.yaml file.
Therefore, to customize your Helm chart, you need to edit the values file
By default, the values.yaml file looks like:

```yaml
# Default values for rhinochart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
 # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```

### Basic configurations

Beginning at the top, you can see that the replicaCount is automatically set to 1,
which means that only one pod will come up. You only need one pod for this example,
but you can see how easy it is to tell Kubernetes to run multiple pods for redundancy.

The **image** section has two things you need to look at: the **repository** where you are pulling your image and the **pullPolicy**.
The pullPolicy is set to **IfNotPresent**; which means that the image will download a new version of the image if one does not already exist in the cluster.
There are two other options for this: **Always**, which means it will pull the image on every deployment or restart (I suggest this in case of image failure),
and **Latest**, which will always pull the most up-to-date version of the image available.
Latest can be useful if you trust your image repository to be compatible with your deployment environment, but that's not always the case.

Let change the **pullPolicy** to **Always**
```yaml
pullPolicy: Always
```

## Naming and secrets

Next, take a look at the overrides in the chart. The first override is imagePullSecrets,
which is a setting to pull a secret, such as a password or an API key you've generated as credentials for a private registry.

an example for **imagePullSecrets**
```yaml
imagePullSecrets:
  - name: regcred
```

Next are nameOverride and fullnameOverride. From the moment you ran helm create, 
its name (rhinochart) was added to a number of configuration files—from the YAML ones above to the templates/helper.tpl file.
If you need to rename a chart after you create it, this section is the best place to do it, so you don't miss any configuration files.


**NOTE** nameOverride replaces the name of the chart in the Chart.yaml file, when this is used to construct Kubernetes object names.
fullnameOverride completely replaces the generated name.

## Accounts

Service accounts provide a user identity to run in the pod inside the cluster.
If it's left blank, the name will be generated based on the full name using the helpers.tpl file
I recommend always having a service account set up so that the application will be directly associated with a user that is controlled 
in the chart.

we can change the name in the serviceAccount section
```yaml
    Name: cherrybomb
```

## Security

You can configure pod security to set limits on what type of filesystem group to use or which user can and cannot be used.
Understanding these options is important to securing a Kubernetes pod, but for this example, I will leave this alone.

```yaml
podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000
```

## Networking

There are two different types of networking options in this chart.
One uses a local service network with a ClusterIP address, which exposes the service on a cluster-internal IP.
Choosing this value makes the service associated with your application reachable only from within the cluster (and through ingress, which is set to false by default). 
The other networking option is NodePort, which exposes the service on each Kubernetes node's IP address on a statically assigned port.
This option is recommended for running minikube, so use it for this how-to.

Let change the **service**

```yaml
service:
  type: NodePort
```

## Resources

Helm allows you to explicitly allocate hardware resources. 
You can configure the maximum amount of resources a Helm chart can request and the highest limits it can receive. 
Since I'm using Minikube on a laptop, I'll set a few limits by removing the curly braces and the hashes to convert the comments into commands.


```yaml
   limits:
     cpu: 100m
     memory: 128Mi
   requests:
     cpu: 100m
     memory: 128Mi
```

Extra reading
## Tolerations, node selectors, and affinities

These last three values are based on node configurations. Although I cannot use any of them in my local configuration, I'll still explain their purpose.

**nodeSelector** comes in handy when you want to assign part of your application to specific nodes in your Kubernetes cluster.
If you have infrastructure-specific applications, you set the node selector name and match that name in the Helm chart. Then,
when the application is deployed, it will be associated with the node that matches the selector.

**Tolerations**, **tainting**, and **affinities** work together to ensure that pods run on separate nodes.
Node affinity is a property of pods that attracts them to a set of nodes (either as a preference or a hard requirement). 
Taints are the opposite—they allow a node to repel a set of pods.

In practice, if a node is tainted, it means that it is not working properly or may not have enough resources to hold the application deployment.
Tolerations are set up as a key/value pair watched by the scheduler to confirm a node will work with a deployment.

Node affinity is conceptually similar to **nodeSelector**: it allows you to constrain which nodes your pod is eligible to be scheduled based on labels on the node.
However, the labels differ because they match rules that apply to scheduling.

## Deploy

The general command is `helm install [NAME] [CHART] [flags]`
```yaml
helm install my-app-rhino rhinochart/ --values rhinochart/values.yaml
```

The output will be
```bash
NAME: my-app-rhino
LAST DEPLOYED: Sun Apr 24 12:34:43 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services rhino-chart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

fixig

`export POD_NAME=` need to get the opd name so just execute `kubectl get pods` and copy the full name

```bash
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl port-forward $POD_NAME 8080:80
```