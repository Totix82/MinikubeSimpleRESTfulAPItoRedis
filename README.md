# MinikubeSimpleRESTfulAPItoRedis

# DevOps task

## The Task

To create a new key/value store service that runs in Kubernetes.
That service will expose a simple RESTful API on a given port and use Redis to read/write.
At a minimum, this API should allow to GET and POST a value based on a key.
The details of the API are free for you to decide, but we need to be able to run
tests using the documentation/examples you provide.

For the actual storage of the key/value pairs, we want to use an existing 3rd
party data store. The choice is Redis and will run on another another container in the same cluster.
Will use a public image from Docker Hub.

Example dataset to initialize the data store
during the first deployment is in JSON

## Dataset

```
[
    {
        "Homer": "Simpson",
        "Jeffrey": "Lebowski",
        "Stan": "Smith"
    }
]
```

##  Task Objectives:

- To write a program that serves the REST API and read/write in the data store.
- To provide a Dockerfile that can build your service.
- To provide the necessary configuration to deploy this service and the data
  store in a Kubernetes environment using helm chart.

## Requirements

- A bash script that builds the project, installs all requirements and starts 
  everything on a locak cluster would be appreciated.
- Local cluster Minikube
- To run the minimum GET/POST requests
  mentioned earlier.
  
## Description and Development

Step 1: start minikube
minikube start

Step 2: create secrets
A password for our redis cluster.
First, use a base64 encoding tool to convert redis password for your redis cluster to a base64 representation
$MyPassRedisUnencoded = 'MyPass123'
$MyPassRedisEncoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($MyPassRedisUnencoded))
Write-Output $MyPassRedisEncoded
e.g. >>>TQB5AFAAYQBzAHMAMQAyADMA

Create the secrets in your kubernetes cluster by applying (Not create since we might edit it later on) related yaml inside the minikube folder
./minikube kubectl -- apply -f 'C:\minikube\app-secret.yaml'

Step 3: create a redis cluster
To create redis cluster, I used Helm. So I enabled it as an addon in my Minikube:
./minikube addons enable helm-tiller

Install Helm v3.x, run the following commands:

curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get-helm-3 > get_helm.sh
./get_helm.sh

Install Chocolatey with Powershell:
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

Then install Helm
choco install kubernetes-helm

Then I have run below command to add the bitnami chart to my helm repo.

helm repo add bitnami https://charts.bitnami.com/bitnami

The install of kubernetes-helm was successful.
Software installed to 'C:\ProgramData\chocolatey\lib\kubernetes-helm\tools'

Then I created a redis cluster with a single command using helm

helm install salvoredisdeployv01 bitnami/redis --values C:\minikube\app-secret.yaml
The above command creates a redis cluster named ‘salvoredisdeployv01’ with configuration parameters mentioned in the file C:\minikube\app-secret.yaml.

NAME: salvoredisdeployv01
LAST DEPLOYED: Sun Apr 10 16:30:51 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.8.5
APP VERSION: 6.2.6

** Please be patient while the chart is being deployed **

Redis&trade; can be accessed on the following DNS names from within your cluster:

    salvoredisdeployv01-master.default.svc.cluster.local for read/write operations (port 6379)
    salvoredisdeployv01-replicas.default.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default salvoredisdeployv01 -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis&trade; server:

1. Run a Redis&trade; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.6-debian-10-r179 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace default -- bash

2. Connect using the Redis&trade; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h salvoredisdeployv01-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h salvoredisdeployv01-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/salvoredisdeployv01-master : &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p

Run the below command to verify redis-cluster has started successfully:

./minikube kubectl -- get all
I got an output as below.

NAME                                 READY   STATUS    RESTARTS      AGE
pod/salvoredisdeployv01-master-0     1/1     Running   0             3m56s
pod/salvoredisdeployv01-replicas-0   1/1     Running   1 (60s ago)   3m56s
pod/salvoredisdeployv01-replicas-1   1/1     Running   1 (64s ago)   2m27s
pod/salvoredisdeployv01-replicas-2   1/1     Running   0             108s

NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                     ClusterIP   10.96.0.1       <none>        443/TCP    2d8h
service/salvoredisdeployv01-headless   ClusterIP   None            <none>        6379/TCP   3m58s
service/salvoredisdeployv01-master     ClusterIP   10.98.191.28    <none>        6379/TCP   3m57s
service/salvoredisdeployv01-replicas   ClusterIP   10.111.181.94   <none>        6379/TCP   3m57s

NAME                                            READY   AGE
statefulset.apps/salvoredisdeployv01-master     1/1     3m57s
statefulset.apps/salvoredisdeployv01-replicas   3/3     3m57s

Any pod in the kubernetes cluster can connect to the redis cluster we just created by mentioning the service ‘service/salvoredisdeployv01-master’

Step 4: create configmap
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume e.g. ConfigMaps as environment variables (among other things).

We need a configmap now to mention our redis host to the pods that are going to deploy the app.

So run

./minikube kubectl -- apply -f 'C:\minikube\app-configmap.yaml'

Step 5: create deployment of service with all liveness and readiness probes to publish a simple Rest API
We will create a simple API using Express.js.

Set up the project:

Download Nodejs from 
https://nodejs.org/en/ and install it.

If you installed under the default configuration, Node.js should now be added to your PATH. Run command prompt or powershell and input the following to test it out:

> node -v
The console should respond with a version string. Repeat the process for npm:

> npm -v
If both commands work, your installation was a success, and you can start using Node.js!

New-Item -Path 'C:\minikube\my-backend-api' -ItemType Directory
New-Item -Path 'C:\minikube\my-backend-api\server.js' -ItemType File
cd 'C:\minikube\my-backend-api'
npm init
npm i express --save

Next, add a Dockerfile and .dockerignore:

// Dockerfile
FROM node:12

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . .

EXPOSE 80
CMD ["node", "server.js"]
// .dockerignore
node_modules

Then, build the image and push it to the Docker Hub registry:

If you want to skip this step, you can use the existing image here.
docker build -t <YOUR_DOCKER_ID>/my-backend-api .
docker push <YOUR_DOCKER_ID>/my-backend-api
3. Deploy
Now, we deploy the image to our local Kubernetes cluster. We use the default namespace.

Create a deployment:

kubectl create deploy my-backend-api --image=salvo/my-backend-api
Alternatively, create a deployment with a YAML file:
kubectl create -f deployment.yaml
		
Create now a service:

kubectl expose deploy my-backend-api --type=ClusterIP --port=80
Alternatively, create a service with a YAML file:
kubectl create -f service.yaml

Check that everything was created and the pod is running:

kubectl get deploy -A
kubectl get svc -A
kubectl get pods -A
Once the pod is running, the API is accessible within the cluster only. One quick way to verify the deployment from our localhost is by doing port forwarding:

Replace the pod name below with the one in your cluster
kubectl port-forward my-backend-api-84bb9d79fc-m9ddn 3000:80
Now, you can send a curl request from your machine
curl http://localhost:3000/user/123
To correctly manage external access to the services in a cluster, we need to use ingress. Close the port-forwarding and let's expose our API by creating an ingress resource.

An ingress controller is also required, but k3d by default deploys the cluster with a Traefik ingress controller (listening on port 80).
Recall that when we created our cluster, we set a port flag with the value "80:80@loadbalancer". If you missed this part, go back and create your cluster again.
Create an Ingress resource with the following YAML file:

kubectl create -f ingress.yaml
kubectl get ing -A

Now try it out!
curl http://localhost:80/user/123

--------------------------

	
PS C:\minikube> ./minikube kubectl -- cluster-info
Kubernetes control plane is running at https://192.168.59.100:8443
CoreDNS is running at https://192.168.59.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
PS C:\minikube> ./minikube kubectl -- get nodes
NAME       STATUS   ROLES                  AGE    VERSION
minikube   Ready    control-plane,master   2d7h   v1.23.3

PS C:\minikube> ./minikube kubectl -- describe node
Name:               minikube
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    minikube.k8s.io/commit=362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
                    minikube.k8s.io/name=minikube
                    minikube.k8s.io/primary=true
                    minikube.k8s.io/updated_at=2022_04_08T08_14_41_0700
                    minikube.k8s.io/version=v1.25.2
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 08 Apr 2022 08:14:13 +0200
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  minikube
  AcquireTime:     <unset>
  RenewTime:       Sun, 10 Apr 2022 15:52:21 +0200
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Sun, 10 Apr 2022 15:50:08 +0200   Fri, 08 Apr 2022 08:14:13 +0200   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sun, 10 Apr 2022 15:50:08 +0200   Fri, 08 Apr 2022 08:14:13 +0200   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sun, 10 Apr 2022 15:50:08 +0200   Fri, 08 Apr 2022 08:14:13 +0200   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Sun, 10 Apr 2022 15:50:08 +0200   Fri, 08 Apr 2022 08:15:01 +0200   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.59.100
  Hostname:    minikube
Capacity:
  cpu:                2
  ephemeral-storage:  17784752Ki
  hugepages-2Mi:      0
  memory:             2186464Ki
  pods:               110
Allocatable:
  cpu:                2
  ephemeral-storage:  17784752Ki
  hugepages-2Mi:      0
  memory:             2186464Ki
  pods:               110
System Info:
  Machine ID:                 ecb1a34fc74a4ecaa0b2ffd53e83c3e7
  System UUID:                576e4e5b-9882-6d49-b47b-492f0a7b8b84
  Boot ID:                    af8cf2c6-3c4a-469b-8db8-8480295c6c53
  Kernel Version:             4.19.202
  OS Image:                   Buildroot 2021.02.4
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://20.10.12
  Kubelet Version:            v1.23.3
  Kube-Proxy Version:         v1.23.3
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (8 in total)
  Namespace                   Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                ------------  ----------  ---------------  -------------  ---
  kube-system                 coredns-64897985d-7wdjj             100m (5%)     0 (0%)      70Mi (3%)        170Mi (7%)     2d7h
  kube-system                 etcd-minikube                       100m (5%)     0 (0%)      100Mi (4%)       0 (0%)         2d7h
  kube-system                 kube-apiserver-minikube             250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d7h
  kube-system                 kube-controller-manager-minikube    200m (10%)    0 (0%)      0 (0%)           0 (0%)         2d7h
  kube-system                 kube-proxy-gpmwt                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d7h
  kube-system                 kube-scheduler-minikube             100m (5%)     0 (0%)      0 (0%)           0 (0%)         2d7h
  kube-system                 storage-provisioner                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d7h
  kube-system                 tiller-deploy-6d67d5465d-9rftf      0 (0%)        0 (0%)      0 (0%)           0 (0%)         18m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                750m (37%)  0 (0%)
  memory             170Mi (7%)  170Mi (7%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>
PS C:\minikube>	



