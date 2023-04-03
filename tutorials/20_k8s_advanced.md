# Kubernetes Tutorial Part II

## Elastic Kubernetes Service (EKS)

Follow the below docs to create a cluster using the management console:  
https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

In order to get the kubeconfig content and connect to an EKS cluster, you should execute the following `aws` command from your local machine:

```shell
aws eks --region <region> update-kubeconfig --name <cluster_name>
```

Change `<region>` and `<cluster_name>` accordingly.

Read [here](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html) how to allow AWS IAM users and role access the cluster.


### Install and connect through OpenLens

https://github.com/MuhammedKalkan/OpenLens

### Install Ingress and Ingress Controller on EKS

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress) exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.
In order for the **Ingress** resource to work, the cluster must have an [**Ingress Controller**](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) running.

Kubernetes supports and maintains AWS, GCE, and [nginx](https://github.com/kubernetes/ingress-nginx) ingress controllers.

1. If working on a shared repo, create your own namespace by:
   ```shell
   kubectl create ns <my-ns-name>
   ```
2. **In your namespace**, Deploy the following Docker image as a Deployment, with correspond Service: [`alexwhen/docker-2048`](https://hub.docker.com/r/alexwhen/docker-2048).
3. Deploy the Nginx ingres controller (done only **once per cluster**). We will deploy the [Nginx ingress controller behind a Network Load Balancer](https://kubernetes.github.io/ingress-nginx/deploy/#aws) manifest.

We want to access the 2048 game application from a domain such as http://test-2048.int-devops-may22.com

4. Add a subdomain A record for the [int-devops-may22.com](https://us-east-1.console.aws.amazon.com/route53/v2/hostedzones#ListRecordSets/Z04765852WWE8ZAF7TX92) domain (e.g. test-2048.int-devops-may22.com). The record should have an alias to the NLB created by EKS after the ingress controller has been deployed.
5. Inspired by the manifests described in [Nginx ingress docs](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/#basic-usage-host-based-routing), create and apply an Ingress resource such that when visiting your registered DNS, the 2048 game will be displayed on screen.

## Helm

Helm is the package manager for Kubernetes.
The main big 3 concepts of helm are:

- A **Chart** is a Helm package. It contains all the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster.
- A **Repository** is the place where charts can be collected and shared.
- A **Release** is an instance of a chart running in a Kubernetes cluster.

[Install](https://helm.sh/docs/intro/install/) the Helm cli if you don't have.

You can familiarize yourself with this tool using [Helm docs](https://helm.sh/docs/intro/using_helm/).

### Deploy MySQL using Helm

Once you have Helm ready, you can add a chart repository. Check [Artifact Hub](https://artifacthub.io/packages/search?kind=0) for publicly available chart developed by the community.

Let's review the Helm chart written by Bitnami for MySQL provisioning in k8s cluster.

[https://github.com/bitnami/charts/tree/master/bitnami/mysql/#installing-the-chart](https://github.com/bitnami/charts/tree/master/bitnami/mysql/#installing-the-chart)

1. Add the Bitnami Helm repo to your local machine:
```shell
# or update if you have it already: `helm repo update bitnami`
helm repo add bitnami https://charts.bitnami.com/bitnami
```
2. First let's install the chart without any changes
```shell
# helm install <release-name> <repo-name>/<chart-name> 
helm install mysql bitnami/mysql
```

Whenever you install a **chart**, a new **release** is created. So the same chart can be installed multiple times into the same cluster.
During installation, the helm client will print useful information about which resources were created, what the state of the release is, and also whether there are additional configuration steps you can or should take.

You can always type `helm list` to see what has been released using Helm.

Now we want to customize the chart according to our business configurations.
To see what options are configurable on a chart, use `helm show values bitnami/mysql` or even better, go to the chart documentation on GitHub.

We will pass configuration data during the chart upgrade by specify a YAML file with overrides (`-f custom-values.yaml`).
This can be specified multiple times and the rightmost file will take precedence.

We want to change the chart's values such that the below architecture will be deployed in out cluster (Multi-AZ DB MySql cluster):

![](img/mysql-multi-instance.png)

3. Create a file called `mysql_values.yaml`, and override the chart's values or [add parameters](https://github.com/bitnami/charts/tree/master/bitnami/mysql/#parameters) according to your need.
4. Upgrade the `mysql` chart by
```shell
helm upgrade -f mysql-helm/values.yaml mysql bitnami/mysql
```

An upgrade takes an existing release and upgrades it according to the information you provide. Because Kubernetes charts can be large and complex, Helm tries to perform the least invasive upgrade. 
It will only update things that have changed since the last release.

If something does not go as planned during a release, it is easy to roll back to a previous release using `helm rollback [RELEASE] [REVISION]`:

```shell
helm rollback mysql 1
```

5. To uninstall this release:
```shell
helm uninstall mysql
```

