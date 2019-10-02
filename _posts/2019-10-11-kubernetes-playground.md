---
title: "Build you own Kubernetes Playground"
excerpt: "Have you ever wanted a destructible and fast Kubernetes playground? Does the Internet world provide only complicated solutions that work on Linux only? This VM based Kubernetes cluster is what you need."
permalink: /kubernetes-playground/
header:
  overlay_image: /assets/images/playground.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Promote constructive play!
categories:
  - Programming
tags:
  - kubernetes
  - vagrant
  - vm
  - linux
  - windows
  - macos
toc: true
---

# The need

Lately, I was in the need for a fast way to provision myself a free and simple Kubernetes cluster with multiple worker nodes and portable across multiple platforms (Windows si primary goal, because we use it at work and Unix-like are nice for personal developing).
I’m studying to achieve CKAD certification so I’ve talked to myself: is there a fast and complete solution?

[TL;DR: **YES**](https://github.com/filippobuletto/k8s-playground){: .btn .btn--success}
{: style="text-align: center;"}

## Solutions that don't fit

Running Kubernetes locally have a simple and straight forward name: [Minikube](https://minikube.sigs.k8s.io), which is a free and fast solution for day-to-day development workflows and learning purposes, but lacks a fundamental requirement: multiple worker nodes!

For the sake of completeness I'll leave here a brief and non exhaustive list of Minikube alternative:

For beginners:

- [Docker for Desktop](https://docs.docker.com/docker-for-mac/#kubernetes) macOS only
- [Microk8s](https://microk8s.dev/) Linux only

Intermediates:

- [KIND](https://kind.sigs.k8s.io/) requires Docker (WSL2 on Windows)
- [K3D](https://github.com/rancher/k3d) requires Docker (hard on Windows)

## Solution that fits: `kubeadm`

<kbd>kubeadm</kbd> helps you bootstrap a minimum viable Kubernetes cluster that conforms to best practices.
{: .notice}

Following the [Kubernetes Getting started guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) you can find a complete guide to bootstrap a test-conformant cluster with multiple worker nodes.

So, _I've done it for you_! And packaged in a simple **Vagrant format**!

I'm building a GitHub repository where you can find all the instructions:

**GitHub Repo:** <https://github.com/filippobuletto/k8s-playground>
{: .notice--info}
{: style="text-align: center;"}

Please try it and leave some feedback and/or [feature requests](https://github.com/filippobuletto/k8s-playground/issues/new)! It's still a work in progress, comments are welcome!
