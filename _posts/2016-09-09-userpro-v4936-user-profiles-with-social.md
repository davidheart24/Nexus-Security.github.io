---
title: 'UserPro v4.9.36 – User Profiles with Social Login nulled'
date: 2020-01-01T12:50:00+01:00
draft: false
---

[![chaos_logo](https://github.com/pingcap/chaos-mesh/raw/master/static/logo.png)](https://github.com/pingcap/chaos-mesh/blob/master/static/logo.png)

> **Note:**
> 
> This readme and related documentation are Work in Progress.

Chaos Mesh is a cloud-native Chaos Engineering platform that orchestrates chaos on Kubernetes environments. At the current stage, it has the following components:

*   **Chaos Operator**: the core component for chaos orchestration. Fully open sourced.
*   **Chaos Dashboard**: a visualized panel that shows the impacts of chaos experiments on the online services of the system; under development; curently only supports chaos experiments on TiDB.

See the following demo video for a quick view of Chaos Mesh:

[![Watch the video](https://github.com/pingcap/chaos-mesh/raw/master/static/demo.gif)](https://www.youtube.com/watch?v=ifZEwdJO868)

[](https://github.com/pingcap/chaos-mesh#chaos-operator)Chaos Operator
----------------------------------------------------------------------

Chaos Operator injects chaos into the applications and Kubernetes infrastructure in a manageable way, which provides easy, custom definitions for chaos experiments and automatic orchestration. There are three components at play:

**Controller-manager**: used to schedule and manage the lifecycle of CRD objects

**Chaos-daemon**: runs as daemonset with privileged system permissions over network, Cgroup, etc. for a specifc node

**Sidecar**: a special type of container that is dynamically injected into the target Pod by the webhook-server, which can be used for hacjacking I/O of the application container.

[![Chaos Operator](https://github.com/pingcap/chaos-mesh/raw/master/static/chaos-mesh-overview.png)](https://github.com/pingcap/chaos-mesh/blob/master/static/chaos-mesh-overview.png)

Chaos Operator uses [Custom Resource Definition (CRD)](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) to define chaos objects. The current implementation supports three types of CRD objects for fault injection, namely PodChaos, NetworkChaos, and IOChaos, which correspond to the following major actions (experiments):

*   pod-kill: The selected pod is killed (ReplicaSet or something similar may be needed to ensure the pod will be restarted)
*   pod-failure: The selected pod will be unavailable in a specified period of time
*   netem chaos: Network chaos such as delay, duplication, etc.
*   network-partition: Simulate network partition
*   IO chaos: simulate file system falults such as I/O delay, read/write errors, etc.

[](https://github.com/pingcap/chaos-mesh#prerequisites)Prerequisites
--------------------------------------------------------------------

Before deploying Chaos Mesh, make sure the following items have been installed. If you would like to have a try on your machine, you can refer to [get-started-on-your-local-machine](https://github.com/pingcap/chaos-mesh#get-started-on-your-local-machine) section.

*   Kubernetes >= v1.12 and < v1.16
*   [RBAC](https://kubernetes.io/docs/admin/authorization/rbac) enabled (optional)
*   [Helm](https://helm.sh/) version >= v2.8.2 and < v3.0.0

[](https://github.com/pingcap/chaos-mesh#deploy-chaos-mesh)Deploy Chaos Mesh
----------------------------------------------------------------------------

### [](https://github.com/pingcap/chaos-mesh#get-the-helm-files)Get the Helm files

```
git clone https://github.com/pingcap/chaos-mesh.git cd chaos-mesh/
```

### [](https://github.com/pingcap/chaos-mesh#create-custom-resource-type)Create custom resource type

To use Chaos Mesh, you must first create the related custom resource type.

```
kubectl apply -f manifests/ kubectl get crd podchaos.pingcap.com
```

### [](https://github.com/pingcap/chaos-mesh#install-chaos-mesh)Install Chaos Mesh

*   Install Chaos Mesh with Chaos Operator only

```
helm install helm/chaos-mesh --name=chaos-mesh --namespace=chaos-testing kubectl get pods --namespace chaos-testing -l app.kubernetes.io/instance=chaos-mesh
```

*   Install Chaos Mesh with Chaos Operator and Chaos Dashboard

```
helm install helm/chaos-mesh --name=chaos-mesh --namespace=chaos-testing --set dashboard.create=true
```

[](https://github.com/pingcap/chaos-mesh#get-started-on-your-local-machine)Get started on your local machine
------------------------------------------------------------------------------------------------------------

> **Warning:**
> 
> **This deployment is for testing only. DO NOT USE in production!**

You can try Chaos Mesh on your local K8s environment deployed using `kind` or `minikube`.

### [](https://github.com/pingcap/chaos-mesh#deploy-your-local-k8s-environment)Deploy your local K8s environment

#### [](https://github.com/pingcap/chaos-mesh#deploy-with-kind)Deploy with `kind`

1.  Clone the code
    
    ```
    git clone --depth=1 https://github.com/pingcap/chaos-mesh && \\ cd chaos-mesh
    ```
    
2.  Run the script and create a local Kubernetes cluster. Make sure you have installed [kind](https://kind.sigs.k8s.io/), and run the script below to create a local Kubernetes cluster.
    
    ```
    hack/kind-cluster-build.sh
    ```
    
3.  To connect the local Kubernetes cluster, set the default configuration file path of `kubectl` to `kube-config`.
    
    ```
    export KUBECONFIG="$(kind get kubeconfig-path)"
    ```
    
4.  Verify whether the Kubernetes cluster is on and running
    
5.  Install `chaos-mesh` on `kind` kubernetes cluster as suggested in [Deploy Chaos Mesh](https://github.com/pingcap/chaos-mesh#deploy-chaos-mesh).
    

#### [](https://github.com/pingcap/chaos-mesh#deploy-with-minikube)Deploy with `minikube`

1.  Start a `minikube` kubernetes cluster. Make sure you have installed [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), and run the script below to start a local `minikube` cluster
    
    ```
    minikube start --kubernetes-version v1.15.0 --cpus 4 --memory "8192mb" # we recommend that you allocate enough RAM(better more than 8192 MiB) to VM
    ```
    
2.  Install helm
    
    ```
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash helm init
    ```
    
3.  Check whether helm tiller pod is running
    
    ```
    kubectl -n kube-system get pods -l app=helm
    ```
    
4.  Install `chaos-operator` as suggested in [Deploy Chaos Mesh](https://github.com/pingcap/chaos-mesh#deploy-chaos-mesh).
    

**Note:**

There are some known restrictions for Chaos Operator deployed on `kind` and `minikube` clusters:

*   All network-related chaos is not supported for `kind` cluster.
    
    Chaos Operator uses docker pkg to transform between container id and pid, which is necessary to find network namespace for pods.`Kind` uses `containerd` as Introducing Container Runtime Interface (CRI) runtime and it's not supported in our implementation yet.
    
*   `netem chaos` is not supported for `minikube` clusters.
    
    In `minikube`, the default virtual machine driver's image doesn't contain the `sch_netem` kernel module. You can use `none` driver (if your host is Linux with the `sch_netem` kernel module loaded) to try these chaos actions on `minikube` or [build an image with sch\_netem by yourself](https://minikube.sigs.k8s.io/docs/contributing/iso/).
    

### [](https://github.com/pingcap/chaos-mesh#deploy-target-cluster)Deploy target cluster

After Chaos Mesh is deployed, we can deploy the target cluster to be tested, or where we want to inject faults. For illustration purposes, we use TiDB as our sample cluster.

You can follow the instructions in the following two documents to deploy a TiDB cluster:

### [](https://github.com/pingcap/chaos-mesh#define-chaos-experiment-config-file)Define chaos experiment config file

In this sample experiment config file, we will define a chaos experiment to kill one tikv pod randomly:

```
apiVersion: pingcap.com/v1alpha1 kind: PodChaos metadata: name: pod-failure-example namespace: chaos-testing spec: action: pod-failure # the specific chaos action to inject; supported actions: pod-kill/pod-failure mode: one # the mode to run chaos action; supported mode are one/all/fixed/fixed-percent/random-max-percent duration: "60s" # duration for the injected chaos experiment selector: # pods where to inject chaos actions namespaces: - tidb-cluster-demo labelSelectors: "app.kubernetes.io/component": "tikv" scheduler: #defines scheduler rules for the running time of the chaos experiments about pods. cron: "@every 5m"
```

### [](https://github.com/pingcap/chaos-mesh#create-a-chaos-experiment)Create a chaos experiment

```
kubectl apply -f pod-failure-example.yaml kubectl get podchaos --namespace=chaos-testing
```

You can see the QPS performance affected by the chaos experiment from TiDB Grafana dashboard:

[![tikv-pod-failure](https://github.com/pingcap/chaos-mesh/raw/master/static/tikv-pod-failure.png)](https://github.com/pingcap/chaos-mesh/blob/master/static/tikv-pod-failure.png)

### [](https://github.com/pingcap/chaos-mesh#update-a-chaos-experiment)Update a chaos experiment

```
vim pod-failure-example.yaml # modify pod-failure-example.yaml to what you want kubectl apply -f pod-failure-example.yaml
```

#### [](https://github.com/pingcap/chaos-mesh#delete-a-chaos-experiment)Delete a chaos experiment

```
kubectl delete -f pod-failure-example.yaml
```

#### [](https://github.com/pingcap/chaos-mesh#warch-your-chaos-experiments-in-dashboard)Warch your chaos experiments in Dashboard

Chaos Dashboard is currently only available for TiDB clusters. Stay tuned for more supports or join us in making it happen.

> **Note:**
> 
> Make sure you have used the [option](https://github.com/pingcap/chaos-mesh#deploy-chaos-mesh) to deploy Chaos Mesh with Chaos Dashboard. If Chaos Dashboard wasn't installed in your Chaos Mesh, you need to install it by upgrading Chaos Mesh:
> 
> `helm upgrade chaos-mesh helm/chaos-mesh --namespace=chaos-testing --set dashboard.create=true`

A typical way to access it is to use `kubectl port-forward`

```
kubectl port-forward -n chaos-testing svc/chaos-dashboard 8080:80
```

Then you can access [`http://localhost:8080`](http://localhost:8080) in browser.

[](https://github.com/pingcap/chaos-mesh#roadmap)Roadmap
--------------------------------------------------------

[](https://github.com/pingcap/chaos-mesh#license)License
--------------------------------------------------------

Chaos Mesh is licensed under the Apache License, Version 2.0. See [LICENSE](https://github.com/pingcap/chaos-mesh/blob/master/LICENSE) for the full license text.

  
  
from Hacker News https://ift.tt/39qSoPT