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

### Install Gitlab

```bash
git clone https://gitlab.com/gitlab-org/charts/gitlab.git
cd gitlab
helm dependency update

tee runners.yml <<EOF
gitlab-runner:
  install: true
  certsSecretName: gitlab-wildcard-tls-chain
  rbac:
    create: true
  runners:
    locked: false
    secret: "nonempty"
    config: |
      [[runners]]
        [runners.kubernetes]
        image = "ubuntu:22.04"
  podSecurityContext:
    seccompProfile:
      type: "RuntimeDefault"
EOF

helm upgrade --install gitlab . -f https://gitlab.com/gitlab-org/charts/gitlab/raw/master/examples/values-minikube-minimum.yaml -f runners.yml \
 --timeout 600s \
 --set global.hosts.domain=$(minikube ip).nip.io \
 --set global.hosts.externalIP=$(minikube ip) \
 --set global.appConfig.terraformState.enabled=true \
 --create-namespace \
 --namespace gitlab
```

### Update hosts file

```bash
echo $(printf $(minikube ip) && kubectl get ingress -n gitlab --no-headers=true  -o custom-columns=HOSTS:.spec.rules[].host | awk '{ printf " " $1}' )  | sudo tee -a  /etc/hosts
```

### Trust certificates root CA

Export the root CA:

```bash
kubectl get -n gitlab secret gitlab-wildcard-tls-ca -ojsonpath='{.data.cfssl_ca}' | base64 --decode > gitlab.192.168.49.2.nip.io.ca.pem
```

Trust it on your system:

[Fedora](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/){:target="_blank"}

```bash
sudo mv gitlab.192.168.49.2.nip.io.ca.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust
```

[Ubuntu](https://ubuntu.com/server/docs/install-a-root-ca-certificate-in-the-trust-store){:target="_blank"}

### Retrieve inital root password

```bash
kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' -n gitlab | base64 -d ; echo
```

### Access your instance

Your instance should now be accessible locally via **https://$(minikube ip).nip.io**.

```bash
echo https://$(minikube ip).nip.io
https://192.168.49.2.nip.io
```
