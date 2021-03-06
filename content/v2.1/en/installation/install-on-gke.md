---
title: "Install KubeSphere on GKE"
keywords: 'kubernetes, docker, helm, jenkins, istio, prometheus'
description: 'How to deploy KubeSphere on GKE'
---

![KubeSphere+GKE](https://pek3b.qingstor.com/kubesphere-docs/png/20191123145223.png)

This guide walks you through the steps of deploying KubeSphere on [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/).

## Prepare a GKE cluster

At first, a standard Kubernetes in GKE is a prerequisite of installing KubeSphere. We create a GKE cluster with `1.14.8-gke.17` in this demo, and  choose the `n1-standard-2 (2 vCPU, 7.5 GB memory)` and three nodes in **Machine configuration**.

> Note:
>
> - `n1-standard-2 (2 vCPU, 7.5 GB memory)` is the minimal requirements. It's recommended to choose higher machine configuration for production environment.
> - `n1-standard-2 (2 vCPU, 7.5 GB memory)` is used for minimal installation of KubeSphere. If you want to install any pluggable component, you need to provide more machine resource. Please see [Enabling pluggable components installation](../install-on-gke/#enable-pluggable-components) for more information.
> - Supported Kubernetes version: `1.13.0 ≤ K8s version < 1.16` for KubeSphere 2.1.0; `1.13.0 ≤ K8s version < 1.17` for KubeSphere 2.1.1.

![Choose GKE](https://pek3b.qingstor.com/kubesphere-docs/png/20191123120312.png)

![Prepare Machines](https://pek3b.qingstor.com/kubesphere-docs/png/20191123120440.png)

## Create Tiller Service Account

KubeSphere requires [Helm](https://v2.helm.sh/) **(>= v2.10.0, excluding v2.16.0)** to continue the installation. By default, [Tiller](https://v2.helm.sh/) is not ready on GKE, thus we need to install Tiller first.

When GKE cluster is ready, we can connect to Cloud Shell.

![Cloud Shell](https://pek3b.qingstor.com/kubesphere-docs/png/20191123122806.png)

Here, we create `helm-rbac.yaml` in GKE as follows:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Let's create these resources using kubectl:

```bash
kubectl apply -f helm-rbac.yaml
```

## Deploy Tiller

Initialize Helm using the following command.

```bash
helm init --service-account=tiller --tiller-image=gcr.io/kubernetes-helm/tiller:v2.14.1   --history-max 300
```

Check the Tiller status using kubectl, when it displays `1/1`, it means you are ready to continue.

```bash
kubectl get deployment tiller-deploy -n kube-system
```

## Install KubeSphere

Install KubeSphere using kubectl. The following command is only to start the default minimal installation:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml
```

Verify the real-time logs, when you see the following outputs, congratulation! You can access KubeSphere in your browser.

```bash
$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f


#####################################################
###              Welcome to KubeSphere!           ###
#####################################################
Console: http://10.128.0.34:30880
Account: admin
Password: P@88w0rd
NOTES：
  1. After logging into the console, please check the
     monitoring status of service components in
     the "Cluster Status". If the service is not
     ready, please wait patiently. You can start
     to use when all components are ready.
  2. Please modify the default password after login.
#####################################################
```

## Access KubeSphere console

In this section, we'll show you how to access KubeSphere console by changing service type to `LoadBalancer`.

![K8s Dashboard](https://pek3b.qingstor.com/kubesphere-docs/png/20191123124133.png)

Select `Services & Ingress` > `ks-console`, then click `EDIT` and modify the service type from `NodePort` to `LoadBalancer`.

![ks-console service](https://pek3b.qingstor.com/kubesphere-docs/png/20191123124325.png)

Now, you can access KubeSphere Console using the endpoint that was generated by GKE.

![ks-console endpoint](https://pek3b.qingstor.com/kubesphere-docs/png/20191123124744.png)

> Note: In addition to changing the service type to LoadBalancer, you can also access KubeSphere console via `NodeIP:NodePort`. You may need to allow port `30880` in firewall rules.

Log in KubeSphere console using the default account `admin / P@88w0rd`. You'll see the dashboard as the following screenshot shown.

![KubeSphere Dashboard](https://pek3b.qingstor.com/kubesphere-docs/png/20191123125116.png)

## Enable Pluggable Components

he installation above is only used for a default minimal installation. Execute the following command to open the configmap in order to enable more pluggable components. Make sure your cluster has enough CPU and memory. Please see [Configuration Table](https://github.com/kubesphere/ks-installer/blob/master/README.md#configuration-table) for more information.

<font color=red>If you want to enable DevOps or etcd monitoring, please create CA and etcd certificates first. See [ks-installer](https://github.com/kubesphere/ks-installer/blob/master/README.md) for complete guide.</font>

```bash
kubectl edit cm -n kubesphere-system ks-installer
```
