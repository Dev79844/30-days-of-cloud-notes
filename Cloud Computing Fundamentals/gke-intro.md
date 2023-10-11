# GKE

## Create a GKE Cluster

    gcloud container clusters create --machine-type=e2-medium --zone=us-east1-c lab-cluster

## Get authentication credentials for the cluster

    gcloud container clusters get-credentials lab-cluster

## Deployment
- Deploy an application using `kubectl create deployment name-of-application`
- Now, create a k8 service to expose the application to external traffic. Use command `kubectl expose deployment name-of-application`
- To inspect the service, use `kubectl get service`
