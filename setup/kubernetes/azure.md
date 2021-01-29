--- 
title: Set Up an Azure Kubernetes Service Cluster
description: Learn how to set up an AKS cluster for your Coder deployment.
---
This deployment guide shows you how to set up an Azure Kubernetes Service (AKS)
cluster on which Coder can deploy.

## Prerequisites

Please make sure that you have the [Azure
CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) and
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed on
your machine.

## Step 1: Create the Resource Group

Create a resource group (be sure to set the **$RESOURCE_GROUP** and
**$LOCATION** environment variables accordingly):

```bash
az group create \ 
  --resource-group "$RESOURCE_GROUP" \ 
  --location "$LOCATION"
```

## Step 2: Create the Azure Kubernetes Service Cluster

Create the Azure Kubernetes Service Cluster (be sure to replace the placeholder
and environment variables with the values that apply to you):

```bash
# You may have to run `az extension add --name aks-preview`
#
# You may also need to create a service principal manually using
# `az ad sp create-for-rbac --skip-assignment`, then setting the
# --service-principal and --client-secret flags

CLUSTER_NAME="MY_CLUSTER_NAME" \
LOCATION="##" SUBSCRIPTION="##" RESOURCE_GROUP="##" \
az aks create \
  --name "$CLUSTER_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --subscription "$SUBSCRIPTION" \
  --generate-ssh-keys \
  --enable-addons http_application_routing \
  --enable-cluster-autoscaler \
  --location "$LOCATION" \
  --max-count 10 \
  --min-count 2 \
  --node-vm-size Standard_B8ms \
  --network-plugin "kubenet" \
  --network-policy "calico"
```

## Step 3: Configure kubectl to Point to the Cluster

After deploying your AKS cluster, configure kubectl to point to your cluster:

```bash
az aks get-credentials --name "$CLUSTER_NAME" --resource-group $RESOURCE_GROUP"
```

## Step 4: Create the Coder Namespace (Optional)

We recommend running Coder in a separate namespace; to do so, run

```bash
kubectl create namespace coder
```

Next, change the kubectl context to point to your newly created namespace:

```bash
kubectl config set-context --current --namespace=coder
```
