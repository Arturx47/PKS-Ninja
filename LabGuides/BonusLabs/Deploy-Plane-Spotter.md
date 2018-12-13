# Deploy Plane Spotter App

## Overview of App
Now that we have a Kubernetes cluster, let's look at the magic of Kubernetes by deploying an application. For this exercise we will be using an app called 'planespotter' developed by the very talented @yfauser [here](https://github.com/yfauser/planespotter). Planespotter essentially lets you query Aircraft data from FAA. It has the following components

1. Front-end: User interface to take queries and showcase results
2. API App server: to retrieve data from DB
3. MySQL DB: Stores Aircraft registration data from FAA
4. Redis-server: Memory cache server to fetch data of Aircrafts currently airborne

## Explore the YAML files that will be used for the deployments. For example look at the front-end deployment YAML file to see how many pods and replicas the deployment YAML has specified.

The deployment YAML for planespotter-frontend has specified 2 replica sets, hence post deployment you should see two pods deployed for the frontend app. Similarly, take a note of the labels the frontend service has been allocated. These labels will be used to built the frontend loadbalancer when we expose the app. Kubernetes will understand which pods to route the incoming traffic for the front-end loadbalancer. The traffic is routed via pod labels, hence there is no need to mess with IP addresses, host names etc!

Go ahead, take a look at the planespotter deployment YAML here! https://github.com/Boskey/planespotter/blob/master/kubernetes/frontend-deployment_all_k8s.yaml

You will recall that we used git to clone the above repo locally and modified one of the manifests (.yaml file) to pull an image from our private Harbor registry. We will use the local files to deploy our app, some will pull images from Docker Hub while the manifest we edited earlier will pull from our Harbor registry.

## Deployment Procedure

With Kubernetes, each component needed for the app is defined in the deployment YAML. The deployment YAML identifies the base container image , the dependencies needed from the infrastructure etc. The yaml files also creates an API service that frontends the component-pod.

1. Create namespace "planespotter" and use that namespace as default

`kubectl create ns planespotter`

`kubectl config set-context my-cluster --namespace planespotter`

This will create a private namespace for us to deploy all the application components into and set the namespace as default for our current context.

Verify your namespace has been created

`kubectl get namespaces`

You should see the planespotter namespace created along with others like, vke-system, kube-system etc.

2. Create a persistent volume claim for mysql

`kubectl create -f ~/planespotter/kubernetes/storage_class.yaml`

`kubectl create -f ~/planespotter/kubernetes/mysql_claim.yaml`

The above commands will create a storage class and generate a persistent volume claim needed to store data for the MySQl server , this claim will generate a 2 GB volume. Explore the YAML files to see what will be claimed, the volume type, the amount of storage needed etc. 

Verify the persistent volume has been generated.
 
`kubectl get pv`

3. Deploy the MySQL Pod

`kubectl create -f ~/planespotter/kubernetes/mysql_pod.yaml`

This will deploy MySQL server which will utilize the Persistent Volume created in the above step. 
Verify the MySQL server app has been deployed

`kubectl get pods --namespace planespotter`

You should see the pod created with name 'mysql-0'. 

4. Deploy the App-Server Pod 

`kubectl create -f ~/planespotter/kubernetes/app-server-deployment_all_k8s.yaml`

5. Deploy the Frontend

`kubectl create -f ~/planespotter/kubernetes/frontend-deployment_all_k8s.yaml`

6. Deploy Redis and the ADSB Sync Service

`kubectl create -f ~/planespotter/kubernetes/redis_and_adsb_sync_all_k8s.yaml`

Verify all the pods needed for front-end, redis, DBC Sync services and App server (7 total) have been deployed and have entered Running state.

`kubectl get pods --namespace planespotter`

**Planespotter app is now deployed and running!**


When we were deploying the various app components ( micro-services) needed for planespotter, we were also building services for each component. The service creation is necessary so that the pods associated with each component can talk to each other. The service binds ports needed for the component to the cluster's internal Pod network. (Similar to SNAT/DNAT)

Check all the services that were created for each component. You can also explore the YAML file to see how the services were specified.

`kubectl get services --namespace planespotter`

You should see a service created for each component, like MySQL, frontend, app-server etc. Also note the Type of service, each one is a "Cluster IP" type with NO External-IP allocated to them.

## Next: Continue to Publishing the Planespotter app



