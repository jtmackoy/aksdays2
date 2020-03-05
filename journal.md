Ref: https://docs.microsoft.com/en-us/azure/dev-spaces/quickstart-netcore
Class Content: https://aka.ms/aksdayscontentpartii

TEALS Program - volunteer to give students access to computer science
  Technology Education and Literacy in Schools
  Need 88 tech volunteers to fill the need locally in DFW
  Apply online - www.tealsk12.org/volunteers/new
  Q: robert@tealsk12.org
  @TealsProgram
  Microsoft.com/teals
  Look into maching for volunteer hours

Dan McQuygen - Prizes

Jerry

## Setting up the AKS cluster
  Mentioned main concern with the lesson is the use of Kubenet CNI vs Azure CNI
## Primitives
### Horizontal Pod Autoscaler (HPA)
Will use metrics to determine whether the deployment needs to scale
- Will typically evaluate every 3-5 minutes
- Not to be confused with the Kubernetes Autoscaler
###   Stateful Sets
For stateful-ish workloads

Useful for Dev environments, not for Prod (still recommend statefull workloads be managed outside of the cluster)
### Daemon Sets
Log forwarding, APM enablement is the idea here

OMS Agent daemon set will run on AKS clusters to report data back to the Azure Portal
### Jobs
One-off operations

Facilitatates ordering of steps when deploying container
#### CronJobs
Schedule Jobs

Think for jobs like indexing periodically throughout the day
### Services
#### ClusterIP
      Creates internal IP address, pulling from overlay network
    NodePort
      Creates internal IP through overlay, but also an open node point on every node within the cluster
      Can then setup a load balancer to each node port
    LoadBalancer
      Usually used in a provider landscape
      Creates the node port *and* a load balancer in the cloud platform
  Ingress Controller
    Interface kubernets creates that allows 3rd parties to integrate
    Ingress
      nginx, haproxy, traffic, istio versions of these are all available
      Allows for things like URL (host) direction to backend services/service ports
  Configmaps
    Allows for multiple pods to use a similar configuration - like an environment variable
    Changing values in a configmap doesn't mean the pod will automatically have that data or that the pod will be restarted
  Secrets
    Similar to configmaps - can mount in environment variables, filesystem
    Is base64 encoded
    Will allow for things like SPN ID, Tennant ID, etc. to allow a SPN to authenticate against keyvault
    Allows RBAC on API and Namespace, Configmaps are just on the API
    Can inject binary files (because they are base64 encoded)
      Java trust store certificate, for example
  Volumes
    Persistent Volumes
      Facilitates the creation (and mounting) or just mounting of an existing disk in AZ
  Network Policy
    Firewall rules in the cluster
    Will leverage CNI you've implemented to create network isolation
Benefits of Containers
  Think framework like Java that enables lightweight and secure portability of application working environment
## Docker Directives
### ENV
Generally don't want to set environment variables here - it should be for system/container related stuff
### CMD vs. RUN
CMD = execution when the container has started

RUN = execution during build
### Entrypoint
Entrypoint could be a shell script with the first argument as the command you actually want to run

Setup hooks after a sigterm, as an example

### EXPOSE
Which ports will be open to the outside world/Internet

Helps Ops teams to understand, from the dockerfile, where connections can be made.

## Image Repositories
Default is DockerHub
- Offers a private repository

### Credentials
File that contains where to look for a system keyring to connect to a private registry

## Dependencies and Image Size
### Image Size
Latency from downloading the image can sometimes come into play.

Advise you reduce the size of your container to mitigate
- Alpine OS - minified version of linux, downside is that you're missing a lot of basic tools without installing them
- Nano Server - .NET Core applications
  - Majority of industry is migrating to this
- Windows Server Core

## Configurability and Security
### Configrability
Considerations for portability.

Do not build configuration into your image - horrible practice
### Security
Scanning options are available
- Twistlock, Aqua Security, Clair

Not building secrets into your containers is also advisable
## Tagging and Versioning
SEMVER is a viable strategy, but there are others
- SHA256 of git commit
- Tag with build ID

Avoid using the "latest" tag
- Configuring to pull every time will add latency
- Image changes won't automatically result in re-deployment
- Kube spec won't recognize changes to latest spec, so redeployment will never take place

## .NET Core React Demo Review

## Deployments Exercise
Ref: https://github.com/redapt/microsoft-aks-days/tree/master/day_2/03_deploy_to_aks



    kubectl 
Recommended that if you don't need the dashboard exposed, you don't.  It's a separate product from kubernetes that has its own sets of vulnerabilities.
    
    kubectl top pods --all-namespaces
Uses the Metrics Server

    kubectl top nodes --all-namespaces
RAM usage on all nodes

    john@Azure:~/clouddrive/AKS_Workshop$ code nginx-deployment.yml

This was pulled from https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/deployment.yaml

    john@Azure:~/clouddrive/AKS_Workshop$ ls
    nginx-deployment.yml  set_env.sh
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl create -f nginx-deployment.yml
    deployment.apps/nginx-deployment created
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl apply -f nginx-deployment.yml
    Warning: kubectl apply should be used on resource created by either kubectl create--save-config or kubectl apply
    deployment.apps/nginx-deployment configured
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get deployments
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2/2     2            2           31s
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get replicasets
    NAME                         DESIRED   CURRENT   READY   AGE
    nginx-deployment-6dd86d77d   2         2         2       38s
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    nginx-deployment-6dd86d77d-l94mq   1/1     Running   0          44s
    nginx-deployment-6dd86d77d-rkwjt   1/1     Running   0          44s

### kubectl describe
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get deployments
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2/2     2            2           7m12s
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl describe nginx-deployment
    error: the server doesn't have a resource type "nginx-deployment"
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl describe deployment nginx-deployment
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Thu, 05 Mar 2020 16:55:49 +0000
    Labels:                 <none>
    Annotations:            deployment.kubernetes.io/revision: 1
                            kubectl.kubernetes.io/last-applied-configuration:
                              {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},"spec":{"replica...
    Selector:               app=nginx
    Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
      nginx:
        Image:        nginx:1.7.9
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-6dd86d77d (2/2 replicas created)
    Events:
      Type    Reason             Age    From                   Message
      ----    ------             ----   ----                   -------
      Normal  ScalingReplicaSet  7m59s  deployment-controller  Scaled up replica set nginx-deployment-6dd86d77d to 2
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get replicaset
    NAME                         DESIRED   CURRENT   READY   AGE
    nginx-deployment-6dd86d77d   2         2         2       8m6s
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl describe replicaset nginx-deployment-6dd86d77d
    Name:           nginx-deployment-6dd86d77d
    Namespace:      default
    Selector:       app=nginx,pod-template-hash=6dd86d77d
    Labels:         app=nginx
                    pod-template-hash=6dd86d77d
    Annotations:    deployment.kubernetes.io/desired-replicas: 2
                    deployment.kubernetes.io/max-replicas: 3
                    deployment.kubernetes.io/revision: 1
    Controlled By:  Deployment/nginx-deployment
    Replicas:       2 current / 2 desired
    Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
    Pod Template:
      Labels:  app=nginx
              pod-template-hash=6dd86d77d
      Containers:
      nginx:
        Image:        nginx:1.7.9
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Events:
      Type    Reason            Age    From                   Message
      ----    ------            ----   ----                   -------
      Normal  SuccessfulCreate  8m24s  replicaset-controller  Created pod: nginx-deployment-6dd86d77d-rkwjt
      Normal  SuccessfulCreate  8m24s  replicaset-controller  Created pod: nginx-deployment-6dd86d77d-l94mq
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl get pods
    NAME                               READY   STATUS    RESTARTS   AGE
    nginx-deployment-6dd86d77d-l94mq   1/1     Running   0          8m32s
    nginx-deployment-6dd86d77d-rkwjt   1/1     Running   0          8m32s
    john@Azure:~/clouddrive/AKS_Workshop$ kubectl describe pods nginx-deployment-6dd86d77d-l94mq
    Name:           nginx-deployment-6dd86d77d-l94mq
    Namespace:      default
    Priority:       0
    Node:           aks-nodepool1-24466890-vmss000000/10.240.0.4
    Start Time:     Thu, 05 Mar 2020 16:55:49 +0000
    Labels:         app=nginx
                    pod-template-hash=6dd86d77d
    Annotations:    <none>
    Status:         Running
    IP:             10.244.2.2
    IPs:            <none>
    Controlled By:  ReplicaSet/nginx-deployment-6dd86d77d
    Containers:
      nginx:
        Container ID:   docker://08cbdcf96beacddd328f30a6651797aecae3ddb1c89e8c1f3606210e68142e36
        Image:          nginx:1.7.9
        Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Thu, 05 Mar 2020 16:55:58 +0000
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-gl27h (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      default-token-gl27h:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-gl27h
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                    node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type    Reason     Age    From                                        Message
      ----    ------     ----   ----                                        -------
      Normal  Scheduled  8m49s  default-scheduler                           Successfully assigned default/nginx-deployment-6dd86d77d-l94mq to aks-nodepool1-24466890-vmss000000
      Normal  Pulling    8m47s  kubelet, aks-nodepool1-24466890-vmss000000  Pulling image "nginx:1.7.9"
      Normal  Pulled     8m43s  kubelet, aks-nodepool1-24466890-vmss000000  Successfully pulled image "nginx:1.7.9"
      Normal  Created    8m40s  kubelet, aks-nodepool1-24466890-vmss000000  Created container nginx
      Normal  Started    8m40s  kubelet, aks-nodepool1-24466890-vmss000000  Started container nginx

.


    kubectl edit 
doesn't persist change to the yaml file.

Short Lady

## Azure Dev Spaces
