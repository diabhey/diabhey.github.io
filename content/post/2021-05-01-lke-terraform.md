---
title: "Setup K8s cluster using LKE and Terraform"
date : '2021-05-01'
draft: false
toc : false
backtotop : no
disable_comments : true
categories: 
    - IaC
tags:
    - Terraform
    - Kubernetes
---

In this blog post, I'll demonstrate how I am using Linode Kubernetes Engine([LKE](https://www.linode.com/products/kubernetes/)) and [Terraform](https://www.terraform.io/) to setup the infrastructure for Hivenetes (a personal project that I am working on). I chose Linode as the cloud provider as it is extremely easy to setup, cost effective and many more. I dont think I will use any other cloud provider other than Linode at least for my personal projects. Now that I have established my love for Linode, let's get started shall we?

<u>*Before you begin*</u>

Make sure you have the following setup,

- Linode account (Note: Using LKE is not free)
- Terraform installed

## Setting up your workspace

- Generate your Linode API Token by following these [steps](https://www.linode.com/docs/guides/getting-started-with-the-linode-api/#get-an-access-token) and save them in a secure place.

- Clone the [hivenetes/apiculture](https://github.com/hivenetes/apiculture) repository

  - ```bash
    git clone https://github.com/hivenetes/apiculture.git && cd apiculture/lke_cluster
    ```

- Initial directory setup should be like this,

  - ```bash
       ├── hivenetes.tf            # Main Configuration File
        ── linode_types.json       # List of Linode Types - Pricing and Specs
       └── variables.tf            # Variables used in hivenetes.tf
    ```

- Create a *terraform.tfvars* file with the following content,

  - ```yaml
    # This just an example
    label = "hivenetes"
    k8s_version = "1.19"
    region = "eu-central"
    pools = [
      {
        # Type can be found from linode_types.json
        type : "g6-standard-2" 
        count : 3
      },
    ]
    ```

- Create a *secret.tfvars* file that holds the API Token

  - ```yaml
    token = "<your linode api token>"
    ```

[^Note]: the .tfvars file should not be checked in to as it may contain sensitive information

## Time to launch

Open a terminal and execute the following,

```bash
# Initialise terraform
terraform init
# View your terraform plan
terraform plan -var-file="secret.tfvars"
# Apply the plan
terraform apply -var-file="secret.tfvars"
# After a few minutes, you will see the outputs being displayed on the terminal
```

### Connecting to LKE cluster

Open a terminal and execute the following,

```bash
# Saving the cluster configuration to hivenetes-config.yaml
export KUBE_VAR=`terraform output kubeconfig` && sed -e 's/^"//' -e 's/"$//' <<<"$KUBE_VAR" | base64 -d > hivenetes-config.yaml
# Add the kubeconfig file to your $KUBECONFIG environment variable
export KUBECONFIG=hivenetes-config.yaml
```

You can now begin interacting with your cluster using kubectl

```bash
# To list all your LKE Nodes
kubectl get nodes
```

Note: I use [fubectl](https://github.com/kubermatic/fubectl) to interact with k8s as it makes my life way easier. I encourage you to check it out. If  you are using microk8s then you can use [microk8s fubectl](https://github.com/diabhey/fubectl), an microk8s adapted version that I made.

## Cleanup

Make sure to clean up and destroy your clusters after using as you will be charged based on your usage.

```bash
# Bye Bye Hivenetes
terraform destroy
```

Now that is how simple and easy it is to spin up a Kubernetes cluster on Linode.
