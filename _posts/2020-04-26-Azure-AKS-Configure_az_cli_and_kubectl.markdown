---
layout: post
title:  "Azure AKS - Configure az cli and kubectl"
date:   2020-04-26 19:09:54 +0200
categories: Kubernetes how-to
---

#### Login azcli

```
# az login --tenant <tenant_id>
```

#### Get kubernetes credentials

```
# az aks get-credentials --name <aks_cluster_name> --resource-group <resource_group_name>
Merged "<aks_cluster_name>" as current context in /home/max/.kube/config
```

#### Test kubectl

```
kubectl cluster-info
Kubernetes master is running at https://<aks_cluster_address>:443
[...]
Metrics-server is running at https://<aks_cluster_address>:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
