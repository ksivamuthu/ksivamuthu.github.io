---
layout: post
title: Terraform - Tillerless Helm Deployment
description: This blog post explains why installing tiller on kubernetes cluster should not be done, how to install helm charts through locally running tiller server and terraform script to achieve tillerless helm deployment. 
tags: [terraform, helm, kubernetes, devops]
featured_image_thumbnail: assets/images/posts/2019/1.jpg
featured_image: assets/images/posts/2019/1.jpg
---

The helm package manager for Kubernetes helps you install and manage applications on your Kubernetes cluster. For more information, see the [Helm documentation](https://github.com/helm/helm) 

Helm has two major components - Helm client and Tiller server.

As in documentation:

The Helm Client is a command-line client for end users. The client is responsible for the following domains:

* Local chart development
* Managing repositories
* Interacting with the Tiller server
* Sending charts to be installed
* Asking for information about releases
* Requesting upgrading or uninstalling of existing releases

The Tiller Server is an in-cluster server that interacts with the Helm client, and interfaces with the Kubernetes API server. The server is responsible for the following:

* Listening for incoming requests from the Helm client
* Combining a chart and configuration to build a release
* Installing charts into Kubernetes, and then tracking the subsequent release
* Upgrading and uninstalling charts by interacting with Kubernetes

In a nutshell, the client is responsible for managing charts, and the server is responsible for managing releases.

When you run the tiller server on your cluster, tiller server gets its full Kubernetes administrator permissions. Tiller server becomes an attack vector.

When you run the tiller server on your local machine, users don't inherit the tiller server permissions on the cluster. Instead, tiller inherits the Kubernetes permissions of the end-user. Its the recommended approach on installing helm chart in remote kubernetes cluster without tiller server on cluster.

## Install tiller & helm locally

* Install helm locally using either homebrew on macOS or chocolatey on windows. 
```bash
  //macOS
  $ brew install kubernetes-helm

  //Windows
  $ choco install kubernetes-helm

  // Linux/Unix script
  $ curl -LO https://git.io/get_helm.sh
  $ chmod 700 get_helm.sh
  $ ./get_helm.sh
  ```

* Create a namespace called tiller with the following command.
```
kubectl create namespace tiller
```

* Set the TILLER_NAMESPACE environment variable to the tiller.
```
export TILLER_NAMESPACE=tiller
```

* Start the tiller server locally
```
tiller -listen=localhost:44134 -storage=secret
```

* Initialize Helm client with --client-only flag to make sure, the tiller is not installed in kubernetes cluster

```
export HELM_HOST=:44134
helm init --client-only
```

* Verify that helm is communicating with the tiller server properly.

```
helm repo update
```

At this point, you can run any helm commands in your helm client terminal window (such as helm install chart_name) to install, modify, delete, or query Helm charts in your cluster. 

## Helm Tiller Plugin

Rimas Mocevicius has developed a helm [plugin](https://github.com/rimusz/helm-tiller) for this purpose. See the [blog post](https://rimusz.net/tillerless-helm) for more details. 

```
$ helm plugin install https://github.com/rimusz/helm-tiller

```

It has commands such as `start`, `start-ci` & `stop`

`start` command starts the tiller server. `start-ci` command is used to start the tiller server and ci environments. `stop` command stops the tiller server.

See more commands of this helm tiller plugin in its [github documentation](https://github.com/rimusz/helm-tiller)

## Terraform script to do tillerless helm deployment

The setup below is to use tiller server running on localhost with the `terraform-helm-provider` 

The helm-tiller plugin `start-ci` command is used for starting tiller on localhost, is invoked via local-exec provisioner resource.

Helm provider is configured with tiller server host as localhost:44134 and set `install_tiller` to false to not install tiller server in kubernetes cluster.

Running the terraform script spins the tiller server and install helm chart in kubernetes cluster configured in helm provider.

```
resource "null_resource" "tiller" {
  provisioner "local-exec" {
    command = "export KUBECONFIG=${var.kubeconfig_path} && helm tiller start-ci &"
  }
}

provider "helm" {
  host            = "127.0.0.1:44134"
  install_tiller  = false

  kubernetes {
    host        = "${var.cluster_endpoint}"
    config_path = "${var.kubeconfig_path}"
  }
}

resource "helm_release" "redis" {
    name      = "redis"
    chart     = "stable/redis"
}

```

## Wrap Up

Helm v3 should be Tillerless. You can read the Helm v3 Design Proposal [here](https://github.com/helm/community/blob/master/helm-v3/000-helm-v3.md). The helm-tiller plugin and terraform script that uses helm-tiller commands to start the tiller server on localhost helps to do deployments securely on the Helm v2 release.

Happy coding and hope this post helps on setting up terraform script for tillerless helm deployment. 