# Kubernetes Tutorial Part I

Kubernetes is shipped by [many different distributions](https://nubenetes.com/matrix-table/#), each aimed for a specific purpose. Throughout this tutorial we will work with Minikube, which is a Kubernetes cluster that can be run locally on your machine.  

## Install Minikube

[Minikube](https://kubernetes.io/docs/tasks/tools/#minikube) is a tool that lets you run Kubernetes locally.

Installation as well as starting a cluster instructions can be found here: https://minikube.sigs.k8s.io/docs/start/


## Install `kubectl`

1. Download the `kubectl` binary from [Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-kubectl-binary-with-curl-on-windows) official site.

2. Windows users, put the `kubectl.exe` binary in a directory accessible in your PATH environment variable.

## Start the Minikube cluster and dashboard

The Minikube cluster can be started by `minikube start --driver-docker`

The Kubernetes Dashboard allows you to get easily acclimated to your new cluster.

1. Execute
```shell
minikube dashboard
```

2. To access the dashboard endpoint, open the printed link with a web browser.

# Kubernetes Tasks

The Kubernetes documentation contains pages that show how to do individual tasks.
During this module we will walk through core tasks. 

https://kubernetes.io/docs/tasks/

## Run a Stateless Application Using a Deployment

Follow:    
https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

### Further reading and doing

#### Understanding [Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

Almost every Kubernetes object includes a nested object that govern the object's configuration: the object `spec`.
The `spec` provides a description of the characteristics you want the resource to have: its **desired state**.

In the .yaml file for the Kubernetes object you want to create, you'll need to set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a name string, UID, and optional namespace.
- `spec` - What state you desire for the object.

**Labels** are key/value pairs that are attached to objects, such as Deployment.
Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users. E.g.:

- `"release" : "stable"`
- `"environment" : "dev"`
- `"tier" : "backend"`

Via a **Label Selector**, the client/user can identify a set of objects. 

## Use a Service to Access an Application in the Cluster

Follow:   
https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/

Notes: 

- Follow _Using a service configuration file_ section (use YAML file instead `kubectl expose` command).


## Use Port Forwarding to Access Applications in a Cluster

Follow:    
https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/


## Assign Memory Resources to Containers and Pods

Follow:    
https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/


## Assign CPU Resources to Containers and Pods

Follow:    
https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/


## Configure Liveness, Readiness and Startup Probes

Follow:     
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

> The following sections can be skipped:  
> - Define a TCP liveness probe
> - Define a gRPC liveness probe
> - Use a named port
> - Protect slow starting containers with startup probes


## The Zero-downtime Deployment challenge

TBD

## Distribute Credentials Securely Using Secrets

Follow:     
https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/


## ConfigMap

In this demo we will deploy MySQL server in Kuberenetes cluster using Deployment.

All Yaml files are under `k8s/mysql-configmap-secret-demo`.


1. Create a Secret object containing the root username password for MySQL

`kubectl apply -f mysql-secret.yaml`

2. Deploy the MySQL deployment by applying `mysql-deployment.yaml` configuration file.

Now let's say we want to allow maximum of 50 connection to our DB. We would like to find a useful way to "inject" this config to our pre-built `mysql:5.7` image (we surely don't want to build the MySQL image ourselves).
For that, the [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) object can assist.
In the `mysql` Docker image, custom configurations for the MySQL server can be placed in `/etc/mysql/mysql.conf.d` directory, any file ends with `.cnf` under that directory, will be applied as an additional configurations to MySQL. But how can we "insert" a custom file to the image? keep reading...

5. Review the ConfigMap object under `mysql-config.yaml`. And apply it.
6. Comment **in** the two snippets in `mysql-deployment.yaml` and apply the changes.
7. Make sure the new configurations applied.


