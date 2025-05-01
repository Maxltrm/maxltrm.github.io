---
layout: post
title: "Enable GitLab Container Registry on Minikube"
date: 2025-05-01 18:54:54 +0100
categories: notes how-to gitlab homelab kubernetes devops container docker
comments: true
#nav-post: true
#related-posts: true
---

## **Enable GitLab Container Registry on Minikube**

Begin by following the instructions [Gitlab on minikube]({{ site.baseurl }}{% post_url 2024-09-22-gitlab-on-minikube %}) for setting up minikube and GitLab.

Stop before running the `helm upgrade command`.
Remove any unnecessary `--set`, such as those related to Terraform state, and add the following `--set` flags to enable the container registry:

```bash
 --set registry.enabled=true \
 --set global.registry.enabled=true \
 --set global.registry.tls.enabled=true \
```

### Trust the Docker Registry Certificate

> **_NOTE:_** Make sure the root CA is installed also system-wide, as explained in [Gitlab on minikube]({{ site.baseurl }}{% post_url 2024-09-22-gitlab-on-minikube %}). 

Export the root CA and move it to the docker certs path:

```bash
kubectl get -n gitlab secret gitlab-wildcard-tls-ca -ojsonpath='{.data.cfssl_ca}' | base64 --decode > gitlab.$(minikube ip).nip.io.ca.crt
```

```bash
sudo mkdir -p /etc/docker/certs.d/registry.192.168.49.2.nip.io:443
sudo mv gitlab.$(minikube ip).nip.io.ca.crt /etc/docker/certs.d/registry.$(minikube ip).nip.io:443/
```

Refer to the official Docker [Documentation](https://docs.docker.com/engine/security/certificates/#understand-the-configuration) for detailed guidance on configuring certificate trust. 

### Restart docker
```bash
systemctl restart docker
```

### Retrieve inital root password

```bash
kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' -n gitlab | base64 -d ; echo
```

### Access your registry

Using the initial root password

```bash
docker login registry.192.168.49.2.nip.io
```
