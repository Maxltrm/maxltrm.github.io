---
layout: post
title:  "Gitlab on minikube quickstart"
date:   2024-09-22 15:48:54 +0100
categories: notes how-to gitlab homelab kubernetes devops
---

## **Gitlab on Minikube Quickstart guide**

### Introduction

I need to test some gitlab features such as storing terraform modules and state files, and I want to experiment with [Atlantis](https://www.runatlantis.io/){:target="_blank"}.

I mostly followed the Gitlab official documentation but I had to take some additional steps so I am taking the opportunity to write a quickstart guide so that I can quickly spin up a gitlab Lab environment if I need it again in the future.

### Create minikube cluster

```bash
minikube start --cpus 4 --memory 10240 --driver docker --addons ingress
```

### Create kustomizations

When I was setting up this lab I had some problems with pvc permissions, pods that needed a pvc would not start giving permission denied errors, searching on github I found the following closed but unresolved issue: https://github.com/kubernetes/minikube/issues/1990 so I decided to work around the problem and run the statefulsets as user 0. Don't do this in prod!

I decided to do it using helm post-renderer and kustomize but it can probably be done via helm values as well.

1. Create a post-renderer script:

```bash
tee hook.sh <<EOF
#!/bin/bash
cat <&0 > resources.yaml
kubectl kustomize
rm resources.yaml
EOF
```

2. Create a kustomization file:

```bash
tee kustomizations.yaml << EOF
kind: Kustomization
resources:
  - resources.yaml

patches:
  - target:
      kind: StatefulSet
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/securityContext/runAsUser
        value: 0
EOF
```

### Install Gitlab

```bash
git clone https://gitlab.com/gitlab-org/charts/gitlab.git
cd gitlab
helm dependency update

helm upgrade --install gitlab . -f https://gitlab.com/gitlab-org/charts/gitlab/raw/master/examples/values-minikube-minimum.yaml \
 --timeout 600s \
 --set global.hosts.domain=$(minikube ip).nip.io \
 --set global.hosts.externalIP=$(minikube ip) \
 --set global.appConfig.terraformState.enabled=true \
 --post-renderer ./hook.sh \
 --create-namespace \
 --namespace gitlab
```

### Update hosts file

```bash
echo $(printf $(minikube ip) && kubectl get ingress -n gitlab --no-headers=true  -o custom-columns=HOSTS:.spec.rules[].host | awk '{ printf " " $1}' )  | sudo tee -a  /etc/hosts
```

### Trust certificates root CA

Export the root CA and trust it on your system:

1. [Fedora](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/){:target="_blank"}
1. [Ubuntu](https://ubuntu.com/server/docs/install-a-root-ca-certificate-in-the-trust-store){:target="_blank"}

```bash
kubectl get secret gitlab-wildcard-tls-ca -ojsonpath='{.data.cfssl_ca}' | base64 --decode > gitlab.192.168.49.2.nip.io.ca.pem
```

### Access your instance

Your instance should now be accessible locally via **https://$(minikube ip).nip.io**.
