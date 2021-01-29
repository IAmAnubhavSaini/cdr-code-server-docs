---
title: Set Up a Google Kubernetes Engine Cluster
description: Learn how to set up a GKE cluster for your Coder deployment.
---

This guide shows you how to set up a Google Kubernetes Engine (GKE) cluster to
which Coder can deploy.

## Step 1: Set Up the GKE Cluster

You can set up a GKE cluster:

1. Using the cloud console
2. Using the command line

### Option 1: Use the Cloud Console

1. Log in to Google Cloud Console and navigate to the Kubernetes Engine page
2. Click **CREATE CLUSTER**
3. Provide a **Name** for your cluster and indicate the number of nodes
4. Select the **Location type** you'd like, then choose the desired zone or
   region for your cluster
5. Choose a **Master** version; Coder works with versions
   greater than 1.14.6
6. In the left sidebar, select your **Node Pool** and click **Nodes**. Select a
   **Machine Configuration** that will work appropriately for your expected
   workloads; any option is fine as long as its resource allocation is greater
   than the [resource requirements specified](../requirements.md)
7. In the left sidebar, click **Networking**. Near the bottom, select **Enable
   HTTP load balancing** to disable the GCE ingress controller, as well as
   **Enable network policy** to enable the **networking.k8s.io/v1** Kubernetes
   extension
8. Click **Create** to proceed.

### Option 2: Use the Command Line

Before proceeding, make sure that the [gcloud
CLI](https://cloud.google.com/sdk/docs/quickstarts) is installed on your
machine.

The following will spin up a Kubernetes cluster using the `gcloud` command (be
sure to replace the parameters and environment variables as needed to reflect
the needs of your environment):

```bash
PROJECT_ID="MY_PROJECT_ID" CLUSTER_NAME="MY_CLUSTER_NAME" \
  gcloud beta container --project "$PROJECT_ID" \
  clusters create "$CLUSTER_NAME"
    --zone "us-central1-a" \ 
   --no-enable-basic-auth \ 
    --cluster-version "1.14.7-gke.14" \ 
    --machine-type "n1-standard-4" \ 
    --image-type "COS" \ 
    --disk-type "pd-standard" \ 
    --disk-size "100" \ 
    --metadata disable-legacy-endpoints=true \ 
    --scopes "https://www.googleapis.com/auth/cloud-platform" \ 
    --num-nodes "2" \ 
    --enable-stackdriver-kubernetes \ 
    --enable-ip-alias \ 
    --network "projects/${PROJECT_ID}/global/networks/default" \ 
    --subnetwork \
    "projects/${PROJECT_ID}/regions/us-central1/subnetworks/default" \
    --default-max-pods-per-node "110" \ 
    --addons HorizontalPodAutoscaling,HttpLoadBalancing \ 
    --enable-autoupgrade \ 
    --enable-autorepair \ 
    --enable-network-policy \ 
    --enable-autoscaling 
    --min-nodes "2" 
    --max-nodes "8" \
    --region "us-central1-c"
```

## Step 2: Configure kubectl to Point to the Cluster

After you deploy your cluster, you must configure kubectl to point to your
cluster:

1. Install the [gcloud CLI](https://cloud.google.com/sdk/docs/quickstarts) (if
   you haven't already)

2. Install kubectl: `gcloud components install kubectl`

3. Initialize kubectl with the cluster credentials: `gcloud container clusters
   get-credentials [CLUSTER_NAME]`

## Step 3: Create the Coder Namespace (Optional)

We recommend running Coder in a separate namespace; to do so, run

```bash
kubectl create namespace coder
```

Next, change the kubectl context to point to your newly created namespace:

```bash
kubectl config set-context --current --namespace=coder
```
