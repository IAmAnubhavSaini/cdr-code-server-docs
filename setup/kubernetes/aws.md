---
title: Set Up an Amazon Elastic Kubernetes Cluster
description: Learn how to set up an Amazon EKS cluster for your Coder deployment.
---

This deployment guide shows you how to set up an Amazon Elastic Kubernetes
Engine cluster on which Coder can deploy.

## Prerequisites

Before proceeding, please make sure that you have the [eksctl command line
utility](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
installed on your machine.

## Step 1: Spin up a K8 Cluster

The following will spin up a Kubernetes cluster using the eksctl command;
replace the parameters and environment variables as needed to reflect those for
your environment.

```bash
CLUSTER_NAME="MY_CLUSTER_NAME" \
  SSH_KEY_PATH="~/.ssh/my-public-key.pub" REGION="us-east-2" \
  eksctl create cluster \
  --name "$CLUSTER_NAME" \
  --version 1.17 \
  --region "$REGION" \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 8 \
  --ssh-access \
  --ssh-public-key "$SSH_KEY_PATH" \
  --managed
```

## Step 2: Adjust the K8 Storage Class

Once you've created the cluster, adjust the default Kubernetes storage class to
support immediate volume binding.

First, delete the gp2 storage class: `kubectl delete sc gp2`

Next, recreate the gp2 storage class with the `volumeBindingMode` set to
`Immediate`:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
volumeBindingMode: Immediate
EOF
```

## Step 3: Create the Coder Namespace (Optional)

We recommend running Coder in a separate namespace; to do so, run

```bash
kubectl create namespace coder
```

Next, change the kubectl context to point to your newly created namespace:

```bash
kubectl config set-context --current --namespace=coder
```
