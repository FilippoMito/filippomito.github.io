---
title: "My toolbox for Kubernetes development (so far) UPDATED"
excerpt: "Here’s a not-so-comprehensive list of all the tools that I use to work with K8s to further enhance my development work."
permalink: /kubernetes-toolbox/
header:
  overlay_image: /assets/images/k8s-tools.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  caption: Tools for new Kubernetes users
categories:
  - Programming
  - DevOps
tags:
  - kubernetes
  - tools
toc: true
toc_sticky: true
toc_icon: "tools"
author_profile: false
sidebar:
  title: "Type of tool"
  nav: kubernetes-toolbox
---

# The Toolbox

## kubebox

**GitHub Repo:** <https://github.com/astefanutti/kubebox>
{: .notice--info}
{: style="text-align: center;"}

> Terminal and Web console for Kubernetes.

Features

* Configuration from _kubeconfig_ files (`KUBECONFIG` environment variable or `$HOME/.kube`)
* Switch contexts interactively
* <<authentication,Authentication support>> (token, username / password, private key / cert, OpenID Connect, Amazon EKS, Google Kubernetes Engine, Digital Ocean)
* Namespace selection and pods list watching
* Container log scrolling / watching
* Container resources usage (memory, CPU, network charts) footnote:[Currently requires priviledged access / role.]
* Container remote exec terminal

## kubectx / kubens

**GitHub Repo:** <https://github.com/ahmetb/kubectx>
{: .notice--info}
{: style="text-align: center;"}

> Switch faster between clusters and namespaces in kubectl

`kubectx` is a utility to manage and switch between `kubectl` contexts. `kubectx` supports <kbd>Tab</kbd> completion on bash/zsh/fish shells to help with long context names. You don't have to remember full context names anymore.

`kubens` is a utility to switch between Kubernetes namespaces. `kubens` also supports <kbd>Tab</kbd> completion on bash/zsh/fish shells.

## kubefwd

**GitHub Repo:** <https://github.com/txn2/kubefwd>
{: .notice--info}
{: style="text-align: center;"}

> Kubernetes port forwarding for local development.

`kubefwd` is a command line utility built to port forward some or all pods within a Kubernetes namespace. `kubefwd` uses the same port exposed by the service and forwards it from a loopback IP address on your local workstation. `kubefwd` temporally adds domain entries to your /etc/hosts file with the service names it forwards.

## kubeval

**GitHub Repo:** <https://github.com/instrumenta/kubeval>
{: .notice--info}
{: style="text-align: center;"}

> Validate your Kubernetes configuration files.

`kubeval` is a tool for validating a Kubernetes YAML or JSON configuration file. It does so using schemas generated from the Kubernetes OpenAPI specification, and therefore can validate schemas for multiple versions of Kubernetes.

## octant

**GitHub Repo:** <https://github.com/vmware-tanzu/octant>
{: .notice--info}
{: style="text-align: center;"}

> A web-based, highly extensible platform for developers to better understand the complexity of Kubernetes clusters.

Octant is a tool for developers to understand how applications run on a Kubernetes cluster. It aims to be part of the developer's toolkit for gaining insight and approaching complexity found in Kubernetes. Octant offers a combination of introspective tooling, cluster navigation, and object management along with a plugin system to further extend its capabilities.

Features

* **Resource Viewer** Graphically visualizate relationships between objects in a Kubernetes cluster. The status of individual objects are represented by color to show workload performance.
* **Summary View** Consolidated status and configuration information in a single page aggregated from output typically found using multiple kubectl commands.
* **Port Forward** Forward a local port to a running pod with a single button for debugging applications and even port forward multiple pods across namespaces.
* **Log Stream** View log streams of pod and container activity for troubleshooting or monitoring without holding multiple terminals open.
* **Label Filter** Organize workloads with label filtering for inspecting clusters with a high volume of objects in a namespace.
* **Cluster Navigation** Easily change between namespaces or contexts across different clusters. Multiple kubeconfig files are also supported.
* **Plugin System** Highly extensible plugin system for users to provide additional functionality through gRPC. Plugin authors can add components on top of existing views.

## rakkess

**GitHub Repo:** <https://github.com/corneliusweig/rakkess>
{: .notice--info}
{: style="text-align: center;"}

> Review Access - kubectl plugin to show an access matrix for k8s server resources.

Have you ever wondered what access rights you have on a provided kubernetes cluster? For single resources you can use `kubectl auth can-i list` deployments, but maybe you are looking for a complete overview? This is what `rakkess` is for. It lists access rights for the current user and all server resources, similar to `kubectl auth can-i --list`. It is also useful to find out who may interact with some server resource.

## stern

**GitHub Repo:** <https://github.com/wercker/stern>
{: .notice--info}
{: style="text-align: center;"}

> Multi pod and container log tailing for Kubernetes.

Stern allows you to `tail` multiple pods on Kubernetes and multiple containers within the pod. Each result is color coded for quicker debugging.

The query is a regular expression so the pod name can easily be filtered and you don't need to specify the exact id (for instance omitting the deployment id). If a pod is deleted it gets removed from tail and if a new pod is added it automatically gets tailed.

When a pod contains multiple containers Stern can tail all of them too without having to do this manually for each one. Simply specify the `container` flag to limit what containers to show. By default all containers are listened to.

## popeye

**GitHub Repo:** <https://github.com/derailed/popeye>
{: .notice--info}
{: style="text-align: center;"}

> A Kubernetes cluster resource sanitizer.

Popeye is a utility that scans live Kubernetes cluster and reports potential issues with deployed resources and configurations. It sanitizes your cluster based on what's deployed and not what's sitting on disk. By scanning your cluster, it detects misconfigurations and ensure best practices are in place thus preventing potential future headaches. It aims at reducing the cognitive overload one faces when operating a Kubernetes cluster in the wild. Furthermore, if your cluster employs a metric-server, it reports potential resources over/under allocations and attempts to warn you should your cluster run out of capacity. Popeye is a readonly tool.

## kube-ps1

**GitHub Repo:** <https://github.com/jonmosco/kube-ps1>
{: .notice--info}
{: style="text-align: center;"}

> Kubernetes prompt info for bash and zsh.

A script that lets you add the current Kubernetes context and namespace configured on `kubectl` to your Bash/Zsh prompt strings (i.e. the `$PS1`).

## krew

**GitHub Repo:** <https://github.com/kubernetes-sigs/krew>
{: .notice--info}
{: style="text-align: center;"}

> Package manager for kubectl plugins.

`krew` is a tool that makes it easy to use kubectl plugins. krew helps you discover plugins, install and manage them on your machine. It is similar to tools like apt, dnf or brew. Today, over [55 kubectl plugins](http://sigs.k8s.io/krew-index/plugins.md) are available on krew.

## kube-shell

**GitHub Repo:** <https://github.com/cloudnativelabs/kube-shell>
{: .notice--info}
{: style="text-align: center;"}

> Kubernetes shell: An integrated shell for working with the Kubernetes.

Under the hood kube-shell still calls kubectl. Kube-shell aims to provide ease-of-use of kubectl and increasing productivity.

## k9s

**GitHub Repo:** <https://github.com/derailed/k9s>
{: .notice--info}
{: style="text-align: center;"}

> Kubernetes CLI To Manage Your Clusters In Style!

K9s provides a curses based terminal UI to interact with your Kubernetes clusters. The aim of this project is to make it easier to navigate, observe and manage your applications in the wild. K9s continually watches Kubernetes for changes and offers subsequent commands to interact with observed Kubernetes resources.

## polaris

**GitHub Repo:** <https://github.com/FairwindsOps/polaris>
{: .notice--info}
{: style="text-align: center;"}

> Validation of best practices in your Kubernetes clusters.

Fairwinds' Polaris keeps your clusters sailing smoothly. It runs a variety of checks to ensure that Kubernetes pods and controllers are configured using best practices, helping you avoid problems in the future.

## kube-hunter

**GitHub Repo:** <https://github.com/aquasecurity/kube-hunter>
{: .notice--info}
{: style="text-align: center;"}

> Hunt for security weaknesses in Kubernetes clusters.

kube-hunter hunts for security weaknesses in Kubernetes clusters. The tool was developed to increase awareness and visibility for security issues in Kubernetes environments.

Run kube-hunter on any machine (including your laptop), select Remote scanning and give the IP address or domain name of your Kubernetes cluster. This will give you an attackers-eye-view of your Kubernetes setup.

You can run kube-hunter directly on a machine in the cluster, and select the option to probe all the local network interfaces.

You can also run kube-hunter in a pod within the cluster. This indicates how exposed your cluster would be if one of your application pods is compromised (through a software vulnerability, for example).

## kubespy

**GitHub Repo:** <https://github.com/pulumi/kubespy>
{: .notice--info}
{: style="text-align: center;"}

> Tools for observing Kubernetes resources in real time.

What happens when you boot up a `Pod`? What happens to a `Service` before it is allocated a public
IP address? How often is a `Deployment`'s status changing?

**`kubespy` is a small tool that makes it easy to observe how Kubernetes resources change in realtime,** derived from the work we did to make Kubernetes deployments predictable in [Pulumi's CLI](https://www.pulumi.com/kubernetes/). Run `kubespy` at any point in time, and it will watch and report information about a Kubernetes resource continuously until you kill it.

## kubectl-neat

**GitHub Repo:** <https://github.com/itaysk/kubectl-neat>
{: .notice--info}
{: style="text-align: center;"}

> Clean up Kuberntes yaml and json output to make it readable

When you create a Kubernetes resource, let's say a Pod, Kubernetes adds a whole bunch of internal system information to the yaml or json that you originally authored. This includes default values, tracking information, internal metadata, etc. `kubectl-neat` is a kubectl plugin that cleans up the yaml or json output of Kubernetes resources, and makes it readable again.
