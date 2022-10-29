---
title: "Kubernetes Tips & Tricks"
date: 2022-10-29
classes: wide
categories:
  - Kubernetes
tags:
  - kubernetes
  - containers
---

In this blog post I am sharing some of the useful Kubernetes aliases that I use on daily basis to make my job a little bit easier.

## Tip 1

When working with a complex Kubernetes cluster where you are dealing with multiple namespaces, it could become quite cumbersome to switching between different namespaces. Following alias helps me to that quickly:

```bash
alias kn='kubectl config set-context --current --namespace'
```

### Usage examples

```bash
kn dev # Switch to dev namespace
kn prod # Switch to prod namespace
```

## Tip 2

As you develop Kubernetes YAML definition files, you may need a quick way to test if the configuration that you are building will yield desired results or not. For example, you create a network policy to ensure that only API pod can call the database pod. How would you test this configuration?  
This is where following alias comes in handy:

```bash
alias kt='kubectl run test-pod --image=busybox -it --rm --restart=Never --'
```

Running the `kt` command quickly spins up a pod with busybox image allowing you to quickly run any commands required for testing your configurations. And thanks to `--rm` parameter, this pod will automatically get deleted as soon as you are done with your testing.

### Usage examples

```bash
kt /bin/sh # Open a shell inside the test-pod
kt wget -O- http://pod1.dev.svc.cluster.local # Test if the pod is able to reach pod1 in the dev namespace
kt wget -O- www.google.com # Check if the pod is able to reach the internet
```

## Tip 3

Often times you need to generate YAML for various Kubernetes objects like deployments, Pods, Services etc. using the `--dry-run=client -o yaml` parameters. Instead of typing these parameters every time, you can use the following handy shortcut:

```bash
export dr='--dry-run=client -o yaml'
```

### Usage examples

```bash
kubectl run nginx-pod --image=nginx $dr # Generate YAML definition for a Pod
kubectl create deployment nginx-deploy --image=nginx --replicas=2 $dr # Generate YAML definition for a Deployment
```

## Tip 4

The following shortcut is particularly a time saver when you are testing different configurations and need to quickly delete pods:

```bash
export now='--force --grace-period 0'
```

### Usage examples

```bash
kubectl delete pod nginx $now # Force delete a Pod without any grace period
```

## Tip 5

Following are few other aliases that I use on daily basis:

```bash
alias k='kubectl'
alias kd='kubectl describe'
alias kg='kubectl get'
```
